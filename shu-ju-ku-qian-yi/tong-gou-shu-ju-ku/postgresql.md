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

### 如何使用 pglogical 將 多個來源PostgreSQL 搬遷到同一個 CloudSQL PostgreSQL instance

Step1: pglogical replication setting on Source PostgreSQL

Step2: pglogical replication setting on Dest CloudSQL

Step3: pglogical.create\_subscription on Dest CloudSQL

Step4: Check status pglogical.show\_subscription\_status
