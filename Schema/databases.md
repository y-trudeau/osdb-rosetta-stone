# Databases and schemas

The meaning of what is a database differ between MySQL and PostgreSQL.  While with MySQL, a database and a schema are the same thing, in PostgreSQL a "database" is a top level container which can contain many schemas (also called namespaces).  The PostgreSQL schemas are about the same a the MySQL databases/schemas but MySQL has nothing like a the PostgreSQL notion of database. A whole MySQL instance essentially forms a single PostgreSQL database.  Said otherwise, a single PostgreSQL instance with multiple databases is somewhat equivalent to multiple instances of MySQL, one per database.

Unless foreign data wrappers are used, a query in PostgreSQL is limited to one database but like with MySQL, it can span multiple schemas.  When a database is created in PostgreSQL, it comes with a default "public" schema.

## Database operations

This only applies to PostgreSQL.

### PostgreSQL

List databases:

```
sysbench=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | fr_CA.UTF-8 | fr_CA.UTF-8 |
 sysbench  | postgres | UTF8     | fr_CA.UTF-8 | fr_CA.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres+
           |          |          |             |             | sbtest=CTc/postgres
 template0 | postgres | UTF8     | fr_CA.UTF-8 | fr_CA.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | fr_CA.UTF-8 | fr_CA.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 test      | postgres | UTF8     | fr_CA.UTF-8 | fr_CA.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres+
           |          |          |             |             | dbmaster=CTc/postgres
(5 rows)

```

Connect to a database:

```
sysbench-# /c sysbench
sysbench-#
```

Create a database using the postgres user:
```
postgres@LabPG11_1:~$ psql
psql (12.3 (Ubuntu 12.3-1.pgdg18.04+1))
Saisissez « help » pour l'aide.

postgres=# create database rosetta;
CREATE DATABASE
```


## Schema operations



### MySQL

Keep in mind for MySQL a schema is a database.

List schemas:
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| sysbench           |
+--------------------+
8 rows in set (0,00 sec)
```

Creation of a schema:
```
mysql> create database rosetta;
Query OK, 1 row affected (0,01 sec)
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| rosetta            |
| sys                |
| sysbench           |
+--------------------+
6 rows in set (0,00 sec)
```

Use a schema:
```
mysql> use rosetta
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

### PostgreSQL

In PostgreSQL, a schema is a essentially a namespace.

List the schemas:
```
sysbench=> select * from pg_catalog.pg_namespace;
  oid  |      nspname       | nspowner |               nspacl                
-------+--------------------+----------+-------------------------------------
    99 | pg_toast           |       10 |
 12314 | pg_temp_1          |       10 |
 12315 | pg_toast_temp_1    |       10 |
    11 | pg_catalog         |       10 | {postgres=UC/postgres,=U/postgres}
  2200 | public             |       10 | {postgres=UC/postgres,=UC/postgres}
 13142 | information_schema |       10 | {postgres=UC/postgres,=U/postgres}
(6 rows)
```

Creation of a schema:
```
sysbench=> create schema test;
CREATE SCHEMA
sysbench=> select * from pg_catalog.pg_namespace;
  oid  |      nspname       | nspowner |               nspacl                
-------+--------------------+----------+-------------------------------------
    99 | pg_toast           |       10 |
 12314 | pg_temp_1          |       10 |
 12315 | pg_toast_temp_1    |       10 |
    11 | pg_catalog         |       10 | {postgres=UC/postgres,=U/postgres}
  2200 | public             |       10 | {postgres=UC/postgres,=UC/postgres}
 13142 | information_schema |       10 | {postgres=UC/postgres,=U/postgres}
 24804 | test               |    16386 |
(7 rows)
```

Use a schema:
```
sysbench=> set search_path to test;
SET
```
