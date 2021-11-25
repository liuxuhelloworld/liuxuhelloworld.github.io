# Redis keys
Redis keys are binary safe, this means that you can use any binary sequence as key, from a string like "foo" to the content of a JPEG file. 

The empty string is also a valid key.

The maximum allowed key size is 512 MB.

A few rules:
- very long keys are not a good idea.
- very short keys are not a good idea.
- try to stick with a schema.

# Redis Strings

# Redis Lists
Redis lists are implemented with Linked Lists because for a database system it is crucial to be able to add elements to a very long list in a very fast way.

When fast access to the middle of a large collection of elements is important, you should use sorted sets.

## common use cases
1. Remember the latest updates posted by users into a social network
2. Communication between processes, using a producer-consumer pattern where producer pushes items into a list, and a consumer comsumes those items and executed actions

# Redis Hashes

# Redis Sets
Redis sets are unordered collections of strings.

Sets are good for expressing relations between objects. For instance we can easily use sets in order to implement tags.
```redis
sadd news:1000:tags 1 2 5 77

sadd tag:1:news 1000
sadd tag:2:news 1000
sadd tag:5:news 1000
sadd tag:77:news 1000

sinter tag:1:news tag:2:news tag:10:news tag:27:news
```

Sets can also be used implement a web-based poker game.
```redis
sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3 H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6 S7 S8 S9 S10 SJ SQ SK

sunionstore game:1:deck deck

spop game:1:deck
spop game:1:deck
spop game:1:deck
spop game:1:deck
spop game:1:deck
```

# Redis Sorted Sets
Every element in a sorted set is associated with a floating point value, called the *score*.

A common use case of sorted sets is leader boards, which show the top N users, and the user rank in the leader board.

# Bitmaps
Bitmaps are not an actual data type, but a set of bit-oriented operations defined on the String type. Since strings are binary safe blobs and their maximum length is 512 MB, they are suitable to set up to 2^32 different bits.

One of the biggest advantages of bitmaps is that they often provide extreme space savings when storing information. For example in a system where different users are represented by incremental user IDs, it is possible to remember a single bit information of 4 billion of users using just 512 MB of memory.

Bitmaps are trivial to split into multiple keys, for example for the sake of sharding the data set abd because in general it is better to avoid working with huge keys. To split a bitmap across different keys instead of setting all the bits into a key, a trivial strategy is just to store M bits per key and obtain the key name with bit-number/M and the Nth bit to address inside the key with bit-number MOD M.

# HyperLogLogs
A HyperLogLog is a probabilistic data structure used in order to estimate the cardinality of a set. Usually counting unique items requires using an amount of memory proportional to the number of items you want to count, because you need to remember the elements you have already seen in the past in order to avoid counting them multiple times. However there is a set of algorithms that trade memory for precision: you end up with an estimated measure with a standard error, which in the case of the Redis implementation is less than 1%.