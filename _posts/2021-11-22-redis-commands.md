- bitcount
```
bitcount key
```

- blpop
```
blpop mylist 5
```

- brpop
```
brpop mylist 5
```

- config get
```
config get *
```

- config set

- del
```
del mykey
```

- exists
```
exists mykey
```

- expire
```
expire mykey 5
```

- get

- getbit
```
getbit key 10
```

- getset: the **getset** command sets a key to a new value, returning the old value as the result.

- hget
```
hget user:1000 username
```

- hgetall
```
hgetall user:1000
```

- hincrby
```
hincrby user:1000 birthyear 10
```

- hmget
```
hmget user:1000 username birthyear
```

- hmset
```
hmset user:1000 username liuxu birthyear 1989 verified 1
```

- hset

- incr: the **incr** command parses the string value as an integer, increments it by one, and finally sets the obtained value as the new value. The **incr** command is atomic, multiple clients issuing **incr** against the same key will never enter into a race condition.
```
incr counter
```

- llen
```
llen mylist
```

- lpop
```
lpop mylist
```

- lpush
```
lpush mylist A B C
```

- lrange
```
lrange mylist 0 -1
```

- ltrim: Redis allows us to use lists as a capped collection, only remembering the latest N items and discarding all the oldest items using the **ltrim** command.
```
ltrim mylist 0 2
```

- mget
```
mget a b c
```

- mset
```
mset a 10 b 20 c 30
```

- persist
```
persist mykey
```

- pfadd
```
pfadd hll 1 2 3 4
```

- pfcount
```
pfcount hll
```

- rpop
```
rpop mylist
```

- rpush
```
rpush mylist A B C
```

- sadd
```
sadd myset 1 2 3
```

- scard
```
scard myset
```

- set
```
set mykey somevalue

set mykey newval nx

set mykey newval xx

set mykey somevalue ex 10
```

- setbit
```
setbit key 10 1
```

- sinter
```
sinter tag:1:news tag:2:news tag:10:news tag:27:news
```

- sismember
```
sismember myset 1
```

- smembers
```
smembers myset
```

- spop
```
spop game:1:deck
```

- sunionstore
```
sunionstore game:1:deck deck
```

- ttl
```
ttl mykey
```

- type
```
type mykey
```

- zadd
```
zadd hackers 1912 "Alan Turing"
```

- zrange
```
zrange hackers 0 -1

zrange hackers 0 -1 withscores
```

- zrangebylex
```
zrangebylex hackers [B [P
```

- zrangebyscore
```
zrangebyscore hackers -inf 1950
```

- zrank
```
zrank hackers "Linux Torvalds"
```

- zremrangebyscore
```
zremrangebyscore hackers 1940 1960
```

- zrevrange
```
zrevrange hackers 0 -1
```