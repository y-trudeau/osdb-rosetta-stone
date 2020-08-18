# Index sizes

## MySQL

The statement below only works for InnoDB:

```
select p.schema_table, index_name, round(p.pages*ps.page_size/1024/1024,1) IndexMB
from (select concat(database_name,'.',table_name) schema_table, index_name, stat_value pages
      from mysql.innodb_index_stats
      where database_name not in ('mysql','sys')
        and stat_name = 'size'
        and index_name <> 'PRIMARY') p
  inner join (select replace(NAME,'/','.') schema_table,
                if(ROW_FORMAT='Compressed',ZIP_PAGE_SIZE,@@innodb_page_size) page_size
              from INNODB_TABLES) ps on p.schema_table = ps.schema_table
order by IndexMB desc limit 10;
+-------------------+------------+---------+
| schema_table      | index_name | indexMB |
+-------------------+------------+---------+
| sysbench.sbtest1  | k_1        |     3.0 |
| sysbench.sbtest10 | k_10       |     3.0 |
| sysbench.sbtest2  | k_2        |     3.0 |
| sysbench.sbtest3  | k_3        |     3.0 |
| sysbench.sbtest4  | k_4        |     3.0 |
| sysbench.sbtest5  | k_5        |     3.0 |
| sysbench.sbtest6  | k_6        |     3.0 |
| sysbench.sbtest7  | k_7        |     3.0 |
| sysbench.sbtest8  | k_8        |     3.0 |
| sysbench.sbtest9  | k_9        |     3.0 |
+-------------------+------------+---------+
10 rows in set (0,00 sec)
```

The primary key indexes are not listed here because InnoDB is a clustered storage engine, the primary key b-tree is where the rows are stored.

## PostgreSQL

```
sysbench=# SELECT CONCAT(n.nspname,'.', c.relname) AS table,     i.relname AS index_name,   pg_size_pretty(pg_relation_size(x.indexrelid)) AS index_size FROM pg_class c      JOIN pg_index x ON c.oid = x.indrelid     JOIN pg_class i ON i.oid = x.indexrelid LEFT JOIN pg_namespace n ON n.oid = c.relnamespace     WHERE c.relkind = ANY (ARRAY['r', 't']) AND n.oid NOT IN (99, 11, 12375) ;
      table      |  index_name   | index_size
-----------------+---------------+------------
 public.sbtest1  | sbtest1_pkey  | 2208 kB
 public.sbtest1  | k_1           | 2224 kB
 public.sbtest2  | sbtest2_pkey  | 2208 kB
 public.sbtest2  | k_2           | 2224 kB
 public.sbtest3  | sbtest3_pkey  | 2208 kB
 public.sbtest3  | k_3           | 2224 kB
 public.sbtest4  | sbtest4_pkey  | 2208 kB
 public.sbtest4  | k_4           | 2224 kB
 public.sbtest5  | sbtest5_pkey  | 2208 kB
 public.sbtest5  | k_5           | 2224 kB
 public.sbtest6  | sbtest6_pkey  | 2208 kB
 public.sbtest6  | k_6           | 2224 kB
 public.sbtest7  | sbtest7_pkey  | 2208 kB
 public.sbtest7  | k_7           | 2224 kB
 public.sbtest8  | sbtest8_pkey  | 2208 kB
 public.sbtest8  | k_8           | 2224 kB
 public.sbtest9  | sbtest9_pkey  | 2208 kB
 public.sbtest9  | k_9           | 2224 kB
 public.sbtest10 | sbtest10_pkey | 2208 kB
 public.sbtest10 | k_10          | 2224 kB
 ```
