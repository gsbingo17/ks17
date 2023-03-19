---
description: >-
  採用 GCP 提供的 DMS 可相當方便的搬遷至CloudSQL PostgreSQL
  https://cloud.google.com/database-migration/docs/postgres/quickstart
  以下提供使用DMS過程常遇到的問題與解法 以及 特別的要點
---

# PostgreSQL

### 哪些[PostgreSQL数据库](https://cloud.google.com/database-migration/docs/postgres/configure-source-database)可以用DMS迁移？

* RDS 9.6.10+, 10.5+, 11.1+, 12, 13, 14
* 地端OP的PostgreSQL 9.4, 9.5, 9.6, 10, 11, 12, 13, 14
* Cloud SQL 9.6, 10, 11, 12, 13, 14
* Aurora 10.11+, 11.6+, 12.4+, 13.3+
* DMS利用PostgreSQL extension [pglogical](https://github.com/2ndQuadrant/pglogical) 实现的数据库迁移，所以主要云商的RDS PostgreSQL基本可以迁移

### 使用DMS to CloudSQL PostgreSQL需要注意的限制

* 表必須要有Primary Key
* Generated Sequence Number 不保證和源數據致
* User Auth Table是不會被遷移

### No pglogical extension installed on databases

![](<../../.gitbook/assets/image (53).png>)

在 Test the migration job階段 由於DMS會將所有來源PostgreSQL的使用者資料庫都搬遷到CloudSQL, 所以 **每個使用者資料庫** (包括**系統資料庫postgres**) 都需要啟用pglogical extension.  解法十分簡單, 連線到資料庫後執行以下SQL語句即可.

```
E.g.
postgres=> CREATE EXTENSION IF NOT EXISTS pglogical;
dvdrental=> CREATE EXTENSION IF NOT EXISTS pglogical;
```

### Replication user XXX doesn't have sufficient privileges: replication user doesn't have rolreplication role

![](<../../.gitbook/assets/image (49).png>)

在 Test the migration job階段, DMS用來連線到來源PostgreSQL的帳號,會需要 **replication** 的權限, 這樣代表缺乏此權限, 連線到資料庫後執行以下SQL語句即可.

```
E.g.
postgres=> ALTER USER postgres REPLICATION;
```

### 如何使用 pglogical 將 多個來源PostgreSQL 搬遷到同一個 CloudSQL PostgreSQL instance

由於DMS目前還不支援此功能, 我們可以直接使用 pglogical extension實現此需求.

![](<../../.gitbook/assets/image (40).png>)

![](<../../.gitbook/assets/image (26).png>)

**Step1: pglogical replication setting on Source PostgreSQL**

以下動作在所有要搬遷至CloudSQL的 Source PostgreSQL 都要做

[安裝 pglogical 以及配置 postgresql.conf](https://cloud.google.com/database-migration/docs/postgres/configure-source-database#on-premise-self-managed-postgresql) 的幾個重要參數

```
shared_preload_libraries = 'pglogical';
wal_level = 'logical';
wal_sender_timeout = 0;
max_replication_slots = #;
max_wal_senders = #;
max_worker_processes = #;
```

連線到想要進行同步的資料庫啟用EXTENSION pglogical

```
E.g.
# psql -h10.66.80.2 -ddvdrental -Uuser -W
dvdrental=> CREATE EXTENSION pglogical;

# psql -h10.66.80.51 -ddvdrental -Uuser -W
dvdrental=> CREATE EXTENSION pglogical;
```

建立 provider 的 Node 以及加入需要同步的Tables

```
E.g. In postgresql_1
dvdrental=> 
SELECT pglogical.create_node(
  node_name := 'provider_partial_1',
  dsn := 'host=10.66.80.2 dbname=dvdrental user,password');

SELECT pglogical.replication_set_add_all_tables('default', ARRAY['public']);

--List the tables joined for replication
SELECT * FROM pglogical.replication_set_table;

E.g. In postgresql_2
dvdrental=> 
SELECT pglogical.create_node(
  node_name := 'provider_partial_2',
  dsn := 'host=10.66.80.51 dbname=dvdrental user,password');

SELECT pglogical.replication_set_add_all_tables('default', ARRAY['public']);

--List the tables joined for replication
SELECT * FROM pglogical.replication_set_table;
```

**Step2: pglogical replication setting on Dest CloudSQL**

於Database flags啟用pglogical以及幾個重要參數\
cloudsql.enable\_pglogical (on) \
cloudsql.logical\_decoding (on)

```
shared_preload_libraries = 'pglogical';
wal_level = 'logical';
wal_sender_timeout = 0;
max_replication_slots = #;
max_wal_senders = #;
max_worker_processes = #;
```

建立目的端的資料庫並啟用EXTENSION pglogical

```
E.g.
# psql -h10.66.82.14 -ddvdrental -Uuser -W

postgres=# create database dvdrental_1;
dvdrental_1=> CREATE EXTENSION pglogical;

postgres=# create database dvdrental_2;
dvdrental_2=> CREATE EXTENSION pglogical;
```

**Step3: create\_subscription on Dest CloudSQL**

```
dvdrental_1=> 
--This user could fully access dvdrental_1
SELECT pglogical.create_node(
  node_name := 'subscriber',
  dsn := 'host=10.66.82.14 dbname=dvdrental_1, user, password')

--This user could connect to source postgreSQL
SELECT pglogical.create_subscription(
  subscription_name := 'dvdrental_partial_1',
  provider_dsn := 'host=10.66.80.2 dbname=dvdrental, user, password');


dvdrental_2=> 
--This user could fully access dvdrental_2
SELECT pglogical.create_node(
  node_name := 'subscriber',
  dsn := 'host=10.66.82.14 dbname=dvdrental_2, user, password')

--This user could connect to source postgreSQL
SELECT pglogical.create_subscription(
  subscription_name := 'dvdrental_partial_2',
  provider_dsn := 'host=10.66.80.51 dbname=dvdrental, user, password');
```

**Step4: Check replication status show\_subscription\_status**

```
SELECT pglogical.show_subscription_status();
```

可以觀看結果有 replicating 的字眼 就代表建立完成.
