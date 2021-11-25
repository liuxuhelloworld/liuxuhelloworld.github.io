# RDB (Redis Database)
The RDB persistence performs point-in-time snapshots of your dataset at specified intervals.

RDB is a very compact single-file point-in-time representation of your Redis data. RDB files are perfect for backups. For instance you may want to archive your RDB files every hour for the latest 24 hours, and to save an RDB snapshot every day for 30 days. This allows you to easily restore different versions of the dataset in case of disasters.

RDB maximums Redis performance since the only work the Redis parent process needs to do in order to persist is forking a child that will do all the rest. The parent instance will never perform disk I/O or alike.

RDB is not good if you need to minimize the chance of data loss in case Redis stops working. You will usually create an RDB snapshot every five minites or more, so in case of Redis stopping working without a correct shutdown for any reason you should be prepared to lose the latest minutes of data.

RDB needs to fork() often in order to persist on disk using a child process. Fork() can be time consuming if the dataset is big, and may result in Redis to stop serving clients for some millisecond or even for one second if the dataset is very big and the CPU performance not great. AOF also needs to fork() but you can tune how often you want to rewrite your logs without any trade-off on durability.

# AOF (Append Only File)
The AOF persistence logs every write operation received by the server, that will be played again at server startup, reconstructing the original dataset.

AOF contains a log of all the instructions one after the other in an easy to understand and parse format. AOF files are usually bigger than the equivalent RDB files for the same dataset.

Using AOF Redis is much more durable: you can have different fsync policies: no fsync at all, fsync every second, fsync at every query. With the default policy of fsync every second write performances are still great.

The AOF log is an append only log, so there are no seeks, nor corruption problems if there is a power outage. Even if the log ends with an half-written command for some reason, the redis-check-aof tool is able to fix it easily.

Resid is able to automatically rewrite the AOF in background when it gets too big. The rewrite is completely safe as while Redis continues appending to the old file, a completely new one is produced with the minimal set of operations need to create the current dataset, and once this second file is ready Redis switches the two and starts appending to the new one.

AOF works by incrementally updating an existing state, like MySQL or MongoDB does, while the RDB snapshotting creates everything from scratch again and again, that is conceptually more robust.

# Which to Use?
The general indication is that you should use both persistence methods if you want a good degree of data safety.

If you care a lot of your data, but still can live with a few minutes of data loss in case of disasters, you can simply use RDB alone.

It is not recommended to use AOF alone, because having a RDB snapshot from time to time is a great idea for doing database backups, for faster restarts, and in the event of bugs in the AOF engine.

Redis >= 2.4 makes sure to avoid triggering an AOF rewrite when an RDB snapshotting operation is already in progress, or allowing a BGSAVE while the AOF rewrite is in progress. This prevents two Redis background processes from doing heavily disk I/O at the same time.

In the case both AOF and RDB persistence are enabled and Redis restarts the AOF file will be used to reconstruct the original dataset since it is guaranteed to be the most complete.

# Backup Your Data
Make sure to backup your database.
- create a cron job in your server creating hourly snapshots of the RDB file in one directory and daily snapshots in a different directory.
- every time the cron script runs, make sure to call the **find** command to make sure too old snapshots are deleted: for instance you can take hourly snapshots for the last 48 hours, and daily snapshots for one or two months.
- at least one time every day make sure to transfer an RDB snapshot outside your data center or at least outside the physical machine running your Redis instance.
