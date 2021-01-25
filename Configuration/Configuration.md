# Configuration

## Current variable values

### MySQL

Some variable values are quite large (ex: optimizer_switch) so I restricted to the variables starting with an "a".
```
mysql> show global variables like 'a%';
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| activate_all_roles_on_login | OFF   |
| admin_address               |       |
| admin_port                  | 33062 |
| auto_generate_certs         | ON    |
| auto_increment_increment    | 1     |
| auto_increment_offset       | 1     |
| autocommit                  | ON    |
| automatic_sp_privileges     | ON    |
| avoid_temporal_upgrade      | OFF   |
+-----------------------------+-------+
9 rows in set (0,01 sec)
```

There are 625 variables in Percona Server 8.0.20-11.  The variables are normally defined in the file `/etc/mysql/my.cnf`.

### PostgreSQL

```
sysbench=# select name, setting from pg_settings;
                  name                  |                 setting                 
----------------------------------------+-----------------------------------------
 allow_system_table_mods                | off
 application_name                       | psql
 archive_cleanup_command                |
 archive_command                        | (disabled)
 archive_mode                           | off
 archive_timeout                        | 0
 array_nulls                            | on
 authentication_timeout                 | 60
 autovacuum                             | on
...
  wal_sender_timeout                     | 60000
 wal_sync_method                        | fdatasync
 wal_writer_delay                       | 200
 wal_writer_flush_after                 | 128
 work_mem                               | 4096
 xmlbinary                              | base64
 xmloption                              | content
 zero_damaged_pages                     | off
(314 rows)
```

The variables are normally defined in the file `/etc/postgresql/12/main/postgresql.conf` where "12" is the version number and "main" the cluster name.

## Main memory buffers

### MySQL

Here's we'll focus on InnoDB, the dominant MySQL storage engine.

#### innodb_buffer_pool_size

The InnoDB buffer pool is the main data cache used by MySQL as MySQL/InnoDB doesn't rely on the OS file cache. On a dedicated large server, the buffer pool can use in excess of 80% of all the available memory, as long as there is no swap activity.  For example, a server with 128GB of RAM could have a buffer pool of 100GB or even higher.  When the configuration stresses the available memory of a Linux server, make sure you configure these sysctl variables:

```
vm.swappiness = 1  # Linux pressure for the file cache
vm.dirty_bytes = 268435456  # Max amount of dirty data from async writes, 256MB ok for a database
vm.dirty_background_bytes = 134217728  # low water mark to start flushing dirty async writes, 128MB
```

#### table_open_cache

MySQL/InnoDB stores data in multiple files which are tablespaces and queries needs file handles to every of these objects.  For performance reasons, a caching of table file handles exists and its size is the defined by table_open_cache.

#### thread_cache_size

MySQL uses a distinct thread for each connections.  When the connections are short lived, this may cause the creation of a large number of threads.  To reduce the number of thread been created, MySQL keeps a number of threads from terminated connections into the thread cache and reuse these threads for new connections.  This variable control the size of the thread cache.

### PostgreSQL

#### shared_buffers

The shared buffer is the PostgreSQL main data cache although PostgreSQL also relies on the OS file cache. The available memory must be balanced between the shared buffer size and the OS file cache.  The PostgreSQL documentation recommends between 25 and 40% of the available memory. Since the amount of memory used by PostgreSQL is smaller, there is no swapping issue on Linux caused by the default vm.swappiness value.

#### effective_cache_size

#### work_mem

When processing queries, PostgreSQL needs memory for operations like sorts and hash joins. This buffer is per session and it defaults to 4MB.  If it is too small, disk will be used but if too large, it can induce swap activity.

#### maintenance_work_mem

While work_mem is a a buffer used for the user operations, the maintenance_work_mem is used by the background operations like vacuum. Normally, it should be at most a few percents of the server memory.

## Redo/WAL logging/

### MySQL

#### innodb_log_buffer_size

Ongoing transactional data is acculated in the a buffer of this size.  When doing many large transactions, a large log buffer helps to reduce the number of writes to the log files.  The default value is 16MB and is correct for most cases.

#### innodb_log_file_size

InnoDB uses a ring buffer type approach for its redo log, the files are pre-allocated to this size.  The default value is 50MB which is really small. Larger values lower the overall write load by delaying flushing. When data pages stay dirty for a longer time in the buffer they can receive additional write operations and that deflates the write load. The drawback of large log files is longer stop if there are many dirty pages and longer recovery time after a crash.  A typical starting value for a server with 64GB of RAM would be 4GB.  

#### innodb_log_files_in_group

This parameter sets the number of InnoDB log files in the InnoDB redo log ring buffer.  The size of the InnoDB redo log ring buffer is thus innodb_log_files_in_group * innodb_log_file_size. What really matters is the total size of the ring buffer which can be as large as 512GB.  The default value for innodb_log_files_in_group is 2. Personally, I never modify this value, I prefer adjusting innodb_log_file_size.

### PostgreSQL


#### max_wal_size  

This parameter is a soft limit on the amount of redo log PostgreSQL keeps. The default value for max_wal_size 1024MB. Keep in mind PostgreSQL uses its redo logs for more than just crash recovery. In addition to the transaction recovery data, PostgreSQL uses the WAL for page double writes, like the InnoDB double write buffer in MySQL, and for replication. These usages cause a large amount of data be written to the WAL and thus, for any non-trivial write load, the default value of 1024MB is pretty small.

#### wal_compression

This parameter controls how the pages are written to the WAL during flushing (double writes). By default, the parameter is OFF and pages are not compressed.  If ON, an algorithm similar to lz4 is used when writing the pages.  Compression significantly reduces the amount of data written but increases the CPU usage.

#### wal_level

PostgreSQL uses its WAL for many purposes.  Of course, it always stores the crash recovery data, only that is stored when the parameter is set to minimal.  It can also contains data required for streaming replication.  Simplifying a bit, there are two levels for replication, replica and logical. When set to replica, the WAL contains the additional information required to update a hot standby server and when pushed to logical, it also adds the information needed for the logical replication.

#### wal_buffers

Data needing to be written to the WAL is accumulated in this buffer until either the buffer is full or a flush is requested. It serves a similar purpose than the innodb_log_buffer_size parameter in MySQL. The default size of this buffer is 512 WAL blocks which means, if all default values are used means 4MB.

#### wal_keep_size

Normally, there is no reason to keep old WAL files once the latest checkpoint points to a more recent file. That's why the default value is 0. Prior to version 13, the adjustment was through wal_keep_segments. Since PostgreSQL uses its WAL for replication, an amount of WAL data needs to be kept in order to allow replica to disconnect, reconnect and resynchronize. This parameter serves a purpose as expire_log_days or binlog_expire_logs_seconds in MySQL


## Flushing

Flushing is the process of writing dirty pages in memory back to disk.  It is an essential process in most database engines expect maybe the LSM oriented ones like RocksDB. Flushing is essential for data persistence.  Databases keep track of pages that have been saved by periodically performing checkpoints.

### MySQL

MySQL uses a fuzzy checkpointing algorithm, a checkpoint is essentially just a marker in the InnoDB redo log.  That means flushing is the only process writing dirty pages.  Its only goal is to write pages to disk fast enough to avoid the InnoDB redo log ring buffer to fill up.  If the redo log is full, transaction has to be paused until space is released.  InnoDB uses an adaptive algorithm that increases the flushing speed as the ring buffer fills.  

#### innodb_adaptive_flushing_lwm

This parameters sets the low water mark level at which the InnoDB adaptive flushing algorithm kicks in. It is expressed as a percentage of the amount of space used in the redo log ring buffer, also called the checkpoint age.  The default value is 10 and is very conservative.  If you have a steady write load with no spikes, you could benefit from having having a large low water mark value.

#### innodb_flush_neighbors

When InnoDB flush pages, it can check if nearby pages from data file perspective are also in the buffer pool and dirty. If so, this parameter tells InnoDB to perform a larger  IO operation and flush more than just the page it is considering.  It is performance tweak designed for spinning disks and is enabled by default. If your storage is flash based, there will be no performance gains and it will increase the write load and lower the life span of the storage devices. Another miss use of this tweak is with cloud block storage like AWS EBS.  These counts IOPS by IO operations of up to 16KB so flushing pages early hits the total available IOPS.

#### innodb_flush_sync

This parameter contols how InnoDB will flush pages if it almost exhausts the space in its redo log ring buffer. When the variable is set on ON, the default, InnoDB will flush pages as fast as the hardware can support it to allow the transaction processing to continue.  This may impact the read load.

#### innodb_flushing_avg_loops

The InnoDB adaptive flushing algorithm averages its flushing rate over over a number of flushing cycles determined by this variable. The default value is 30 and since a flushing takes about one second, it means an average over 30s. This means it can take InnoDB up to 30s to react to a sudden change in workload.  

#### innodb_io_capacity

This variables controls how many IOPS InnoDB uses for operations like:
- Idle flushing when the checkpoint age is below innodb_adaptive_flushing_lwm
- Change buffer operations
- Purge operations

The default value is 200 and it must be lower than innodb_io_capacity_max.  __Do not__ align to your hardware actual write IO capacity.

#### innodb_io_capacity_max

This parameter serves as a multiplier in the InnoDB adaptive flushing algorithm and also caps the maximum number of pages flushed by second.  Unless you have a very high write load and don't mind burning your flash devices, __do not__ align to your hardware actual write IO capacity.  The default value is 2000 and unless you really have a huge write load, this should be sufficient.  If set too high, the page cleaners will throw error message that they can't flush the required number of pages per second.

#### innodb_max_dirty_pages_pct  

This parameter controls the legacy flushing algorithm of InnoDB which was based only on the percentage of dirty pages in the buffer pool. There are very few reasons to change the default value of 90.

#### innodb_max_dirty_pages_pct_lwm

This parameter controls at which level does the legacy algorithm kicks in. The defaults value is 10, it represents a percentage of the buffer pool pages that are dirty. There are very few reasons to change the default value.

### PostgreSQL

PostgreSQL has a different approach to flushing, as it requires all dirty pages to be flushed at a given checkpoint. The flushing is thus a process that starts ahead of a checkpoint to help it completes in time.  If the flushing is too slow, the checkpoint process may have to flush a lot pages to complete the checkpoint and this will impact transaction processing.  It is thus important to make the background flushing is able to keep up with the write load.

#### bgwriter_delay

This parameter specifies a delay between the flushing runs. The goal is to avoid the saturation of the storage.  The default is 200 milliseconds. With

#### bgwriter_flush_after

This parameter determines how many flushed pages does PostgreSQL fsyncs the datafiles.  The default value is 64 pages which means 512KB.  Fsyncs are expensive calls so it makes sense to group many pages together but if pushed too far, that can stalls the storage and cause performance hiccups.

#### bgwriter_lru_maxpages

This variable sets the maximum number of pages that can be flushed by flush cycle.  The default is 100. This means, if bgwriter_delay is 200ms, there cannot be more than 500 pages flushed per second, not counting of course the pages that could be flushed by the checkpointing process.

#### bgwriter_lru_multiplier

This parameter controls how many pages will be flushed in a cycle, capped at bgwriter_lru_maxpages. It is a multipler of the amount of pages/buffers that were read recently, over the last 16 cycles. The default value is 2.

## Durability

### MySQL

| innodb_flush_log_at_timeout    | 1        |
| innodb_flush_log_at_trx_commit | 2        |

### PostgreSQL

synchronous_commit=off
 wal_writer_delay                       | 200
 wal_writer_flush_after                 | 128
  wal_sync_method                        | fdatasync

## Background threads

### MySQL

### PostgreSQL
