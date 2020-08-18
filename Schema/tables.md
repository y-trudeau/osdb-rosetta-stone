# Table definitions

## MySQL

```
mysql> show create database sysbench\G
*************************** 1. row ***************************
       Database: sysbench
Create Database: CREATE DATABASE `sysbench` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */
1 row in set (0,00 sec)

mysql> use sysbench
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+--------------------+
| Tables_in_sysbench |
+--------------------+
| sbtest1            |
| sbtest10           |
| sbtest2            |
| sbtest3            |
| sbtest4            |
| sbtest5            |
| sbtest6            |
| sbtest7            |
| sbtest8            |
| sbtest9            |
+--------------------+
10 rows in set (0,01 sec)

mysql> show create table sysbench.sbtest1\G
*************************** 1. row ***************************
       Table: sbtest1
Create Table: CREATE TABLE `sbtest1` (
  `id` int NOT NULL AUTO_INCREMENT,
  `k` int NOT NULL DEFAULT '0',
  `c` char(120) NOT NULL DEFAULT '',
  `pad` char(60) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  KEY `k_1` (`k`)
) ENGINE=InnoDB AUTO_INCREMENT=100001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8
1 row in set (0,00 sec)

mysql> desc sbtest1;
+-------+-----------+------+-----+---------+----------------+
| Field | Type      | Null | Key | Default | Extra          |
+-------+-----------+------+-----+---------+----------------+
| id    | int       | NO   | PRI | NULL    | auto_increment |
| k     | int       | NO   | MUL | 0       |                |
| c     | char(120) | NO   |     |         |                |
| pad   | char(60)  | NO   |     |         |                |
+-------+-----------+------+-----+---------+----------------+
4 rows in set (0,00 sec)
```

## PostgreSQL

```
sysbench=# \dt
         List of relations
 Schema |   Name   | Type  | Owner  
--------+----------+-------+--------
 public | sbtest1  | table | sbtest
 public | sbtest10 | table | sbtest
 public | sbtest2  | table | sbtest
 public | sbtest3  | table | sbtest
 public | sbtest4  | table | sbtest
 public | sbtest5  | table | sbtest
 public | sbtest6  | table | sbtest
 public | sbtest7  | table | sbtest
 public | sbtest8  | table | sbtest
 public | sbtest9  | table | sbtest
(10 rows)
```

There is apparently no way of getting the DDL creating a table like the "SHOW CREATE TABLE" statement of MySQL.  The desired output can be obtained from the `pg_dump` tool:

```
postgres@LabPG11_1:~$ pg_dump -s -d sysbench -t sbtest1
--
-- PostgreSQL database dump
--

-- Dumped from database version 12.3 (Ubuntu 12.3-1.pgdg18.04+1)
-- Dumped by pg_dump version 12.3 (Ubuntu 12.3-1.pgdg18.04+1)

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

SET default_tablespace = '';

SET default_table_access_method = heap;

--
-- Name: sbtest1; Type: TABLE; Schema: public; Owner: sbtest
--

CREATE TABLE public.sbtest1 (
    id integer NOT NULL,
    k integer DEFAULT 0 NOT NULL,
    c character(120) DEFAULT ''::bpchar NOT NULL,
    pad character(60) DEFAULT ''::bpchar NOT NULL
);


ALTER TABLE public.sbtest1 OWNER TO sbtest;

--
-- Name: sbtest1_id_seq; Type: SEQUENCE; Schema: public; Owner: sbtest
--

CREATE SEQUENCE public.sbtest1_id_seq
    AS integer
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


ALTER TABLE public.sbtest1_id_seq OWNER TO sbtest;

--
-- Name: sbtest1_id_seq; Type: SEQUENCE OWNED BY; Schema: public; Owner: sbtest
--

ALTER SEQUENCE public.sbtest1_id_seq OWNED BY public.sbtest1.id;


--
-- Name: sbtest1 id; Type: DEFAULT; Schema: public; Owner: sbtest
--

ALTER TABLE ONLY public.sbtest1 ALTER COLUMN id SET DEFAULT nextval('public.sbtest1_id_seq'::regclass);


--
-- Name: sbtest1 sbtest1_pkey; Type: CONSTRAINT; Schema: public; Owner: sbtest
--

ALTER TABLE ONLY public.sbtest1
    ADD CONSTRAINT sbtest1_pkey PRIMARY KEY (id);


--
-- Name: k_1; Type: INDEX; Schema: public; Owner: sbtest
--

CREATE INDEX k_1 ON public.sbtest1 USING btree (k);


--
-- PostgreSQL database dump complete
--

```
