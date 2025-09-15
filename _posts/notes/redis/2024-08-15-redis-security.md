By default Redis binds to all the interfaces and has no authentication at all.

1. Make sure the port Redis uses to listen for connections (by default 6379 and additionally 16379 if you run Redis in cluster mode, plus 26379 for Sentinel) is firewalled, so that it's not possible to contact Redis from the outside world.
2. Use a configuration file where the **bind** directive is set in order to guarantee that Redis listens to only the network interfaces you are using.
3. Use the **requirepass** option in order to add an additional layer of security so that clients will require to authenticate using the **AUTH** command.
4. Use **spiped** or another SSL tunneling software in order to encrypt traffic between Redis servers and Redis clients if your environment requires encryption.

Redis is designed to be accessed by trusted clients inside trusted environments. This means that usually it is not a good idea to expose the Redis instance directly to the internet or, in general, to an environment where untrusted clients can directly access the Redis TCP port or UNIX socket.

In general, untrusted access to Redis should always be mediated by a layer implementing ACLs, validating user input, and deciding what operations to perform against the Redis instance.

## protected mode
Since version 3.2.0, Redis enters a special mode called **protected mode** when it is executed with the default configuration (binding all the interfaces) and without any password in order to access it. In this mode, Redis only replies to queries from the loopback interfaces, and replies to clients connecting from other addresses with an error that explains the problem and how to configure Redis properly.

## authentication
Redis provides two ways to authenticate clients. The recommended authentication method, introduced in Redis 6, is via Access Control Lists, allowing named users to be created and assigned fine-grained permissions.

The legacy authentication method is enabled by editing the **redis.conf** file, providing a database password using the **requirepass** setting. When the **requirepass** setting is enabled, Redis will refuse any query by unauthenticated clients. A client can authenticate itself by sending the **AUTH** command followed by the password. The password is set by the system administrator in clear text inside the **redis.conf** file. It should be long enough to prevent brute force attacks.

Since the **AUTH** command, like every other Redis command, is sent unencrypted, it does not protect against an attacker that has enough access to the network to perform eavesdropping.

## attacks triggered by malicious inputs from external clients
There is a class of attacks that an attacker can trigger from the outside even without external access to the instance.

An attacker could supply, via a web form, a set of strings that are known to hash to the same bucket in a hash table in order to turn the O(1) expected time to the O(N) worst case. This can consume more CPU than expected and ultimately cause a Denial of Service. To prevent this specific attack, Redis uses a per-execution, pseudo-random seed to the hash function.

Redis implements the SORT command using the qsort algorithm. Currently, the algorithm is not randomized, so it is possible to trigger a quadratic worst-case behavior by carefully selecting the right set of inputs.

## ACL
ACL is the feature that allows certain connections to be limited in terms of the commands that can be executed and the keys that can be accessed.