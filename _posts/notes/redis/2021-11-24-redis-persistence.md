Persistence refers to the writing of data to durable storage, such as a solid-state disk (SSD).

# RDB (Redis Database)
RDB persistence performs point-in-time snapshots of your dataset at specified intervals.

RDB is a very compact single-file point-in-time representation of your Redis data. RDB files are perfect for backups. For instance you may want to archive your RDB files every hour for the latest 24 hours, and to save an RDB snapshot every day for 30 days. This allows you to easily restore different versions of the dataset in case of disasters.

RDB is very good for disaster revovery, being a single compact file that can be transferred to far data centers.

RDB maximums Redis performance since the only work the Redis parent process needs to do in order to persist is forking a child that will do all the rest. The parent instance will never perform disk I/O or alike.

RDB allows faster restarts with big datasets compared to AOF.

On replicas, RDB supports partial resynchronizations after restarts and failovers.

RDB is not good if you need to minimize the chance of data loss in case Redis stops working. You will usually create an RDB snapshot every five minites or more, so in case of Redis stopping working without a correct shutdown for any reason you should be prepared to lose the latest minutes of data.

RDB needs to fork() often in order to persist on disk using a child process. fork() can be time consuming if the dataset is big, and may result in Redis stopping serving clients for some milliseconds or even for one second if the dataset is very big and the CPU performance is not great. AOF also needs to fork() (during log rewriting) but less frequently and you can tune how often you want to rewrite your logs without any trade-off on durability.

By default Redis saves snapshots of the dataset on disk, in a binary file called **dump.rdb**. You can configure Redis to have it save the dataset every N seconds if there are at least M changes in the dataset, or you can manually call the **SAVE** or **BGSAVE** commands.

Whenever Redis needs to dump the dataset to disk, this is what happens:
- Redis forks, we now have a child and a parent process
- the child starts to write the dataset to a temporary RDB file
- when the child is done writing the new RDB file, it replaces the old one

Background saving fails with a fork() error on Linux? The Redis background saving schema relies on the copy-on-write semantic of the **fork** system call in modern operating systems: Redis forks (creates a child process) that is an exact copy of the parent. The child process dumps the DB on disk and finally exits. In theory the child should use as much memory as the parent being a copy, but actually thanks to the copy-on-write semantic implemented by modern operating systems the parent and child process will share the common memory pages. A page will be duplicated only when it changes in the child or in the parent. Since in theory all the pages may change while the child process is saving, Linux can't tell in advance how much memory the child will take, so if the **overcommit_memory** setting is set to zero the fork will fail unless there is as much free RAM as required to really duplicate all the parent memory pages. If you have a Redis dataset of 3 GB and just 2 GB of free memory it will fail. Setting **overcommit_memory** to 1 tells Linux to relax and perform the fork in a more optimistic allocation fashion, and this is indeed what you want for Redis.

# AOF (Append Only File)
AOF persistence logs every write operation received by the server. These operations can then be replayed again at server startup, reconstructing the original dataset.

Using AOF Redis is much more durable: you can have different *fsync* policies: no fsync at all, fsync every second, fsync at every query. With the default policy of fsync every second, write performance is still great.

The AOF log is an append-only log, so there are no seeks, nor corruption problems if there is a power outage. Even if the log ends with an half-written command for some reason, the redis-check-aof tool is able to fix it easily.

Resid is able to automatically rewrite the AOF in background when it gets too big. The rewrite is completely safe as while Redis continues appending to the old file, a completely new one is produced with the minimal set of operations need to create the current dataset, and once this second file is ready Redis switches the two and starts appending to the new one.

AOF contains a log of all the instructions one after the other in an easy to understand and parse format. 

AOF files are usually bigger than the equivalent RDB files for the same dataset.

It is possible the server crashed while writing the AOF file, or the volume where the AOF file is stored was full at the time of writing. When this happens the AOF still contains consistent data representing a given point-in-time version of the dataset (that may be old up to one second with the default AOF fsync policy), but the last command in the AOF could be truncated. The latest major versions of Redis will be able to load the AOF anyway, just discarding the last non well formed command in the file.

If the AOF file is not just truncated, but corrupted with invalid byte sequences in the middle, things are more complex. The best thing to do is to run the **redis-check-aof** utility, initially without the **--fix** option, then understand the problem, jump to the given offset in the file, and see if it is possible to manually repair the file.

# RDB vs AOF
The general indication is that you should use both persistence methods if you want a good degree of data safety.

If you care a lot of your data, but still can live with a few minutes of data loss in case of disasters, you can simply use RDB alone.

It is not recommended to use AOF alone, because having a RDB snapshot from time to time is a great idea for doing database backups, for faster restarts, and in the event of bugs in the AOF engine.

Redis >= 2.4 makes sure to avoid triggering an AOF rewrite when an RDB snapshotting operation is already in progress, or allowing a BGSAVE while the AOF rewrite is in progress. This prevents two Redis background processes from doing heavily disk I/O at the same time.

In the case both AOF and RDB persistence are enabled and Redis restarts the AOF file will be used to reconstruct the original dataset since it is guaranteed to be the most complete.

# Backing Up Redis Data
Make sure to backup your database.
- create a cron job in your server creating hourly snapshots of the RDB file in one directory and daily snapshots in a different directory
- every time the cron script runs, make sure to call the **find** command to make sure too old snapshots are deleted: for instance you can take hourly snapshots for the last 48 hours, and daily snapshots for one or two months
- at least one time every day make sure to transfer an RDB snapshot outside your data center or at least outside the physical machine running your Redis instance.

# Backing Up AOF Persistence
If you run a Redis instance with only AOF persistence enabled, you can still perform backups.
- trun off automatic rewrites with **CONFIG SET auto-aof-rewrite-percentage 0**
- check there is no current rewrite in progress using **INFO persistence** and verifying **aof_rewrite_in_progress** is 0
- now you can safely copy the files in the appenddirname directory
- re-enable rewrites when done, **CONFIG SET auto-aof-rewrite-perecentage \<prev-value\>