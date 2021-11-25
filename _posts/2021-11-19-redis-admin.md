redis-server: the Redis Server itself
redis-sentinel: Redis Sentinel executable (monitoring and failover)
redis-cli: command line interface utility to talk with Redis
redis-benchmark: used to check Redis performances

# Starting Redis
```bash
redis-server

redis-server /xxx/redis.conf
```

# Check if Redis is working
External programs talk to Redis using a TCP socket and a Redis specific protocol. This protocol is implemented in the Redis client libraries for the different programming languages. 

**redis-cli** is a command line utility that can be used to send commands to Redis.

```bash
redis-cli
```

# Securing Redis
By default Redis binds to all the interfaces and has no authentication at all.

1. Make sure the port Redis uses to listen for connections (by default 6379 and additionally 16379 if you run Redis in cluster mode, plus 26379 for Sentinel) is firewalled, so that it's not possible to contact Redis from the outside world.
2. Use a configuration file where the **bind** directive is set in order to guarantee that Redis listens to only the network interfaces you are using.
3. Use the **requirepass** option in order to add an additional layer of security so that clients will require to authenticate using the **AUTH** command.
4. Use **spiped** or another SSL tunneling software in order to encrypt traffic between Redis servers and Redis clients if your environment requires encryption.
