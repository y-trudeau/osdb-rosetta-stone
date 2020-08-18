# Databases

Here's how to list the databases, show their creation details and use them.

## MySQL

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

mysql> show create database sysbench\G
*************************** 1. row ***************************
       Database: sysbench
Create Database: CREATE DATABASE `sysbench` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */
1 row in set (0,00 sec)

mysql> use sysbench
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

```

## PostgreSQL

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

sysbench-# /c sysbench
sysbench-# 
```
