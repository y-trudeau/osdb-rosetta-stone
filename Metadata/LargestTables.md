# Largest tables

## MySQL

```
mysql>
SELECT concat(table_schema,'.',table_name) schema_table,concat(round(table_rows/1000000,2),'M') nrows,
  concat(round(data_length/(1024*1024*1024),2),'G') Data,round(data_length/table_rows,0) DataRow ,
  concat(round(index_length/(1024*1024*1024),2),'G') idx,round(index_length/table_rows,0) as IdxRow,
  concat(round((data_length+index_length)/(1024*1024*1024),2),'G') totSize,round(index_length/data_length,2) idxfrac
  FROM information_schema.TABLES where table_schema not in ('mysql','information_schema','performance_schema')
  ORDER BY data_length+index_length DESC LIMIT 10;
+-------------------+-------+-------+---------+-------+--------+---------+---------+
| schema_table      | nrows | Data  | DataRow | idx   | IdxRow | totSize | idxfrac |
+-------------------+-------+-------+---------+-------+--------+---------+---------+
| sysbench.sbtest3  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest10 | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest2  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest8  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest5  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest9  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest7  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest6  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest4  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
| sysbench.sbtest1  | 0.10M | 0.02G |     229 | 0.00G |     32 | 0.02G   |    0.14 |
+-------------------+-------+-------+---------+-------+--------+---------+---------+
10 rows in set (0,07 sec)
```

## PostgreSQL

```
sysbench=# SELECT CONCAT(n.nspname,'.', c.relname) AS table, pg_size_pretty(st.table_size) "data",
  pg_size_pretty(st.index_size) "index", pg_size_pretty(st.table_size+st.index_size) total_size,
  c.reltuples as rows
FROM pg_class c
  JOIN (SELECT x.indrelid, pg_relation_size(x.indrelid) AS table_size,
          sum(pg_relation_size(x.indexrelid)) AS index_size,
          pg_size_pretty(pg_total_relation_size(x.indrelid)) AS total_size
        FROM pg_class c
          JOIN pg_index x ON c.oid = x.indrelid
          JOIN pg_class i ON i.oid = x.indexrelid
        WHERE c.relkind = ANY (ARRAY['r', 't'])
          AND c.relnamespace NOT IN (99, 11, 12375)
        GROUP BY x.indrelid) st ON st.indrelid = c.oid
  JOIN pg_namespace n ON n.oid = c.relnamespace
ORDER BY total_size DESC
LIMIT 10;
      table      | data  | index | total_size |  rows  
-----------------+-------+-------+------------+--------
 public.sbtest1  | 25 MB | 18 MB | 43 MB      | 100000
 public.sbtest2  | 25 MB | 18 MB | 43 MB      | 100000
 public.sbtest3  | 25 MB | 18 MB | 43 MB      | 100000
 public.sbtest4  | 25 MB | 18 MB | 43 MB      | 100000
 public.sbtest5  | 25 MB | 18 MB | 43 MB      | 100000
 public.sbtest6  | 25 MB | 18 MB | 43 MB      | 100000
 public.sbtest7  | 25 MB | 18 MB | 43 MB      | 100000
 public.sbtest8  | 25 MB | 19 MB | 43 MB      | 100000
 public.sbtest9  | 25 MB | 18 MB | 43 MB      | 100000
 public.sbtest10 | 25 MB | 18 MB | 43 MB      | 100000


   oid  | relname | relnamespace | reltype | reloftype | relowner | relam | relfilenode | reltablespace | relpages | reltuples | relallvisible | reltoastrelid | relhasindex | relisshared | relpersistence | relkind | relnatts | relchecks | relhasrules | relhastriggers | relhassubclass | relrowsecurity | relforcerowsecurity | relispopulated | relreplident | relispartition | relrewrite | relfrozenxid | relminmxid | relacl | reloptions | relpartbound
-------+---------+--------------+---------+-----------+----------+-------+-------------+---------------+----------+-----------+---------------+---------------+-------------+-------------+----------------+---------+----------+-----------+-------------+----------------+----------------+----------------+---------------------+----------------+--------------+----------------+------------+--------------+------------+--------+------------+--------------
 16390 | sbtest1 |         2200 |   16392 |         0 |    16386 |     2 |       16390 |             0 |     3165 |    100000 |          3165 |             0 | t           | f           | p              | r       |        4 |         0 | f           | f              | f              | f              | f                   | t              | d            | f              |          0 |          492 |          1 |        |            |

  oid  | datname  | datdba | encoding | datcollate  |  datctype   | datistemplate | datallowconn | datconnlimit | datlastsysoid | datfrozenxid | datminmxid | dattablespace |                          datacl                          
-------+----------+--------+----------+-------------+-------------+---------------+--------------+--------------+---------------+--------------+------------+---------------+----------------------------------------------------------
 16387 | sysbench |     10 |        6 | fr_CA.UTF-8 | fr_CA.UTF-8 | f             | t            |           -1 |         13441 |          479 |          1 |          1663 | {=Tc/postgres,postgres=CTc/postgres,sbtest=CTc/postgres}


 sysbench=# select * from pg_tablespace where oid=1663;
 oid  |  spcname   | spcowner | spcacl | spcoptions
------+------------+----------+--------+------------
 1663 | pg_default |       10 |        |
(1 ligne)


```
