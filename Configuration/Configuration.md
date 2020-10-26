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

The shared buffer is the PostgreSQL main data cache although it also relies on the OS file cache. The available memory must be balanced between the shared buffer size and the OS file cache.  The PostgreSQL documentation recommends between 25 and 40% of the available memory.

#### work_mem

#### maintenance_work_mem


## Redo/WAL logging/

### MySQL

### PostgreSQL

## Flushing

### MySQL

### PostgreSQL
bgwriter_delay
bgwriter_lru_maxpages
bgwriter_lru_multiplier
bgwriter_flush_after

## Durability

### MySQL

### PostgreSQL

## Background threads

### MySQL

### PostgreSQL
