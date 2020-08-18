# Configuration

## Current variables

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
