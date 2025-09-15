Redis Sentinel provides high availability for Redis when not using Redis Cluster. It provides the following capabilities:
- monitoring, Sentinel constantly checks if your master and replica instances are working as expected
- notification, Sentinel can notify the system administrator, or other computer programs, via an API, that something is wrong with one of the monitored Redis instances
- automatic failover, if a master is not working as expected, Sentinel can start a failover process where a replica is promoted to master, the other additional replicas are reconfigured to use the new master, and the application using the Redis server are informed about the new address to use when connecting
- configuration provider, Sentinel acts as a source of authority for clients service discovery, clients connect to Sentinels in order to ask for the address of the current Redis master responsible for a given service. If a failover occurs, Sentinels will report the new address

## Redis Sentinel is a distributed system
Sentinel itself is designed to run in a configuration where there are multiple Sentinel processes cooperating together. 
- failure detection is performed when multiple Sentinels agree about the fact a given master is no longer avaiable. This lower the probability of false positives
- Sentinel works even if not all the Sentinel processes are working, making the system robust against failures. There is no fun in having a failover system which is itself a single point of failure

You need at least three Sentinel instances for a robust deployment. The three Sentinel instances should be placed into computers or virtual machines that are believed to fail in an independent way.

Sentinel + Redis distributed system does not guarantee that acknowledged writes are retaining during failures, since Redis uses asynchronous replication. However there are ways to deploy Sentinel that make the window to lose writes limited to certain moments.

## quorum
What does the *quorum* parameter mean in the sentinel configuration? The quorum is the number of Sentinels that need to agree about the fact the master is not reachable, in order to really mark the master as failing, and eventually start a failover procedure if possible. However the quorum is only used to detect the failure. In order to actually perform a failover, one of the Sentinels need to be elected leader for the failover and be authorized to proceed. This only happens with the vote of the majority of the Sentinel processes. 

For example, if you have 5 Sentinel processes, and the quorum for a given master set to the value of 2, this is what happens:
- if 2 Sentinels agree at the same time about the master being unreachable, one of the two will try to start a failover
- if there are at least a total of 3 Sentinels reachable, the failover will be authorized and will actually start

Quotum is the number of Sentinel processes that need to detect an error condition in order for a master to be flagged as **ODOWN**. The failover is triggered by the **ODOWN** state. Once the failover is triggered, the Sentinel trying to failover is required to ask for authorization to a majority of Sentinels (or more than the majority if the quorum is set to a number greater than the majority).

For example, if you have 5 Sentinel processes, and the quorum is configured to 5, all the Sentinels must agree about the master error condition, and the authorization from all Sentinels is required in order to failover.

This means that the quorum can be used to tune Sentinel in two ways:
- if a quorum is set to a value smaller than the majority of Sentinels we deploy, we are basically making Sentinel more sensitive to master failures, triggering a failover as soon as even just a minority of Sentinels is no longer able to talk with master
- if a quorum is set to a value greater than the majority os Sentinels, we are making Sentinel able to failover only when there are a very large number (larger than majority) of well connected Sentinels which agree about the master being down

## Sentinel API
Sentinel provides an API in order to inspect its state, check the health of monitored masters and replicas, subscribe in order to receive specific notifications, and change the Sentinel configuration at run time.

By default Sentinel runs using TCP port 26379. Sentinels accept commands using the Redis protocol, so you can use **redis-cli** or any other unmodified Redis client in order to talk with Sentinel.

It is possible to directly query a Sentinel to check what is the state of the monitored Redis instances from its point of view, to see what other Sentinels it knows, and so forth. Alternatively, using Pub/Sub, it is possible to receive push style notifications from Sentinels, every time some event happens, like a failover, or an instance entering an error condition, and so forth.

Sentinel acts as a configuration provider for clients that want to connect to a set of master and replicas. Because of possible failovers or reconfigurations, clients have no idea about who is the currently active master for a given set of intances, so Sentinel exports an API to ask this question.

## adding or removing Sentinels
Adding a new Sentinel to your deployment is a simple process because of the auto-discover mechanism implemented by Sentinel. All you need to do is to start the new Sentinel configured to monitor the currently active master. Within 10 seconds the Sentinel will acquire the list of other Sentinels and the set of replicas attached to the master.

If you need to add multiple Sentinels at once, it is suggested to add it one after the other, waiting for all the other Sentinels to already know about the first one before adding the next. This can be easily achieved by adding every new Sentinel with a 30 seconds delay, and during absence of network partitions. At the end of the process it is possible to use the command **SENTINEL MASTER mastername** in order to check if all the Sentinels agree about the total number of Sentinels monitoring the master.

Removing a Sentinel is a bit more complex: Sentinels never forget already seen Sentinels, even if they are not reachable for a long time, since we don't want to dynamically change the majority needed to authorize a failover and creation of a new configuration number. So in order to remove a Sentinel the following steps should be performed in absence of network partitions:
- stop the Sentinel process of the Sentinel you want to remove
- send a **SENTINEL RESET mastername** command to all the other Sentinel instances. One after the other, waiting for at least 30 seconds between instances
- check that all the Sentinels agree about the number of Sentinels currently active, by inspecting the output of **SENTINEL MASTER mastername** of every Sentinel

## removing the old master or unreachable replicas
Sentinels never forget about replicas of a given master, even when they are unreachable for a long time. This is useful, because Sentinels should be able to correctly reconfigure a returning replica after a network partition or a failure event. 

However sometimes you want to remove a replica (that may be the old master) forever from the list of replicas monitored by Sentinels. In order to do this, you need to send a **SENTINEL RESET mastername** command to all the Sentinels. They will refresh the list of replicas within the next 10 seconds, only adding the ones listed as correctly replicating from the current master **INFO** output.

## Pub/Sub messages
A client can use a Sentinel as a Redis-compatible Pub/Sub server in order to **SUBSCRIBE** or **PSUBSCRIBE** to channels and get notified about specific events. The channel name is the same as the name of the event.

## SDOWN and ODOWN failure state
Redis Sentinel has two different concepts of being down, one is called a *Subjectively Down* condition (SDOWN) and is a down condition that is local to a given Sentinel instance. Another is called *Objectively Down* condition (ODOWN) and is reached when enough Sentinels have an SDOWN condition.

From the point of view of a Sentinel an SDOWN condition is reached when it does not receive a valid reply to **PING** requests for the number of seconds specified in the configuration as **is-master-down-after-milliseconds** parameter.

SDOWN is not enough to trigger a failover, it only means a single Sentinel believes a Redis instance is not available. To trigger a failover, the ODOWN state must be reached. To switch from SDOWN to ODOWN no strong consensus algorithm is used, but just a form of gossip. If a given Sentinel gets reports that a master is not working from enough Sentinels in a given time range, the SDOWN is promoted to ODOWN. A more strict authorization that uses an actual majority is required in order to really start the failover, but no failover can be triggered without reaching the ODOWN state.

The ODOWN condition only applies to masters. For other kind of instances Sentinel doesn't require to act, so the ODOWN state is never reached for replicas and other sentinels, but only SDOWN is.

## replica selection and priority
The replica selection process evaluates the following information about replicas:
1. disconnection time from the master
2. replica priority
3. replication offset processed
4. run ID

In most cases, **replica-priority** does not need to be set explicitly so all instances will use the same default value. If there is a particular fail-over preference, **replica-priority** must be set on all instances, including masters, as a master may become a replica at some future point in time, and it will then need the proper **replica-priority** settings.

## configuration epoch and configuration propagation
When a Sentinel is authorized, it gets a unique configuration epoch for the master it is failing over. This is a number that will be used to version the new configuration after the failover is completed. Because a majority agreed that a given version was assigned to a given Sentinel, no other Sentinel will be able to use it. This means that every configuration of every failover is versioned with a unique version.

Once a Sentinel is able to failover a master successfully, it will start to broadcast the new configuration so that other Sentinels will update their information about a given master. At this point, even if the reconfiguration of the replicas is in progress, the failover is considered to be successful, and all the Sentinels are required to start reporting the new configuration. 

Every Sentinel continuously broadcast its version of the configuration of a master using Redis Pub/Sub messages, both in the master and all the replicas. At the same time all the Sentinels wait for messages to see what is the configuration advertised by the other Sentinels. Because every configuration has a different version number, the greater version always wins over smaller versions. A set of Sentinels that are able to communicate will all converge to the same configuration with the higher version number.

## consistency under partitions
In general Redis + Sentinel as a whole are an **eventually consistent system** where the merge function is **last failover wins**, and the data from old masters are discarded to replicate the data of the current master, so there is always a window for losing acknowledged writes. This is due to Redis asynchronous replication and the discarding nature of the virtual merge function of the system.

There are only two ways to avoid losing acknowledged write:
- use synchronous replication (and a proper consensus algorithm to run a replicated state machine)
- use an eventually consistent system where different versions of the same object can be merged

Redis currently is not able to use any of the above systems, and is currently outside the development goals.

## TILT mode
The TILT mode is a special protection mode that a Sentinel can enter when something odd is detected that can lower the reliability of the system. What a Sentinel does is to register the previous time the timer interrupt was called, and compare it with the current call: if the time difference is negative or unexpectedly big, the TILT mode is entered.

When in TILT mode the Sentinel will continue to monitor everything, but:
- it stops acting at all
- it starts to reply negatively to **SENTINEL is-master-down-by-addr** requests as the ability to detect a failure is no longer trusted

If everything appears to be normal for 30 seconds, the TILT mode is exited.