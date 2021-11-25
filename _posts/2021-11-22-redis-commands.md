- bitcount
```redis
bitcount key
```

- blpop
```redis
blpop mylist 5
```

- brpop
```redis
brpop mylist 5
```

- del
```redis
del mykey
```

- exists
```redis
exists mykey
```

- expire
```redis
expire mykey 5
```

- get

- getbit
```redis
getbit key 10
```

- getset

The **getset** command sets a key to a new value, returning the old value as the result.

- hget
```redis
hget user:1000 username
```

- hgetall
```redis
hgetall user:1000
```

- hincrby
```redis
hincrby user:1000 birthyear 10
```

- hmget
```redis
hmget user:1000 username birthyear
```

- hmset
```redis
hmset user:1000 username liuxu birthyear 1989 verified 1
```

- hset

- incr
```redis
incr counter
```

The **incr** command parses the string value as an integer, increments it by one, and finally sets the obtained value as the new value.

The **incr** command is atomic, multiple clients issuing **incr** against the same key will never enter into a race condition.

- llen
```redis
llen mylist
```

- lpop
```redis
lpop mylist
```

- lpush
```redis
lpush mylist A B C
```

- lrange
```redis
lrange mylist 0 -1
```

- ltrim
```redis
ltrim mylist 0 2
```

Redis allows us to use lists as a capped collection, only remembering the latest N items and discarding all the oldest items using the **ltrim** command.

- mget
```redis
mget a b c
```

- mset
```redis
mset a 10 b 20 c 30
```

- persist
```redis
persist mykey
```

- pfadd
```redis
pfadd hll 1 2 3 4
```

- pfcount
```redis
pfcount hll
```

- rpop
```redis
rpop mylist
```

- rpush
```redis
rpush mylist A B C
```

- sadd
```redis
sadd myset 1 2 3
```

- scard
```redis
scard myset
```

- set
```redis
set mykey somevalue

set mykey newval nx

set mykey newval xx

set mykey somevalue ex 10
```

- setbit
```redis
setbit key 10 1
```

- sinter
```redis
sinter tag:1:news tag:2:news tag:10:news tag:27:news
```

- sismember
```redis
sismember myset 1
```

- smembers
```redis
smembers myset
```

- spop
```redis
spop game:1:deck
```

- sunionstore
```redis
sunionstore game:1:deck deck
```

- ttl
```redis
ttl mykey
```

- type
```redis
type mykey
```

- zadd
```redis
zadd hackers 1912 "Alan Turing"
```

- zrange
```redis
zrange hackers 0 -1

zrange hackers 0 -1 withscores
```

- zrangebylex
```redis
zrangebylex hackers [B [P
```

- zrangebyscore
```redis
zrangebyscore hackers -inf 1950
```

- zrank
```redis
zrank hackers "Linux Torvalds"
```

- zremrangebyscore
```redis
zremrangebyscore hackers 1940 1960
```

- zrevrange
```redis
zrevrange hackers 0 -1
```
