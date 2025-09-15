Compared to collections, streams provide a view of data that lets you specify computations at a higher conceptual level. With a stream, you specify what you want to have done, not how to do it. You leave the scheduling of operations to the implementation.

There is a typical workflow when you work with streams. You set up a pipeline of operations in three stages:
1. create a stream
2. specify intermediate operations for transforming the initial stream into others, possibly in multiple steps
3. apply a terminal operation to produce a result. This operation forces the execution of the lazy operations that precede it. Afterwards, the stream can no longer be used

A stream seems superficially similar to a collection, allowing you to transform and retrieve data. But there are significant differences:
- a stream does not store its elements. They may be stored in an underlying collection or generated on demand
- stream operation doesn't mutate their source. For example, the **filter** method does not remove elements from a stream but yields a new stream in which they are not present
- stream operations are lazy when possible. This means they are not executed until their result is needed