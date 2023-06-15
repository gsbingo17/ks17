---
description: >-
  GCP的DMS支援SQLServer為DataSource 計畫在2023-H2推出, 敬請期待. 
  在目前階段可採用交易式複寫以及BAK完整備份還原的方式進行搬遷
---

# SQL Server

### 哪些SQLServer数据库可以用交易式複寫遷移？

* 只要來源的SQLServer有支援能當作 發行者(Publisher) 都沒問題
* 微軟官方提到 [差距兩個版本內](https://learn.microsoft.com/en-us/sql/relational-databases/replication/replication-backward-compatibility?view=sql-server-ver16#replication-matrix) 都可以使用交易式複寫搬遷\
  支援 SQLServer 2014 2016 2017 搬遷到 CloudSQL 2017\
  支援 SQLServer 2016 2017 2019 搬遷到 CloudSQL 2019

### 使用 交易式複寫 + 由備份初始化訂閱 的方式 搬遷SQLServer 至 CloudSQL SQLServer

CloudSQL SQLServer 有支援 Microsoft 原生的複寫(replication)功能\
不論當作 [訂閱者(Subscriber)](https://cloud.google.com/solutions/migrating-data-from-sql-server-to-cloud-sql-for-sql-server) 以及 [發行者(Publisher)](https://cloud.google.com/sql/docs/sqlserver/replication/configure-external-replica) 都沒有問題\
大家可參考上面Link中, 都有詳細的指令與步驟

以下特別針對當 來源SQLServer資料庫很大(e.g. TB) 又希望能持續性的將資料異動同步到CloudSQL SQLServer, 以減少搬遷cut-over date的downtime時間.  這樣的情境下, 就推薦使用 交易式複寫 + 由備份初始化訂閱 的方式來搬遷到CloudSQL. 詳細步驟可參考此 [文件](https://docs.google.com/document/d/12CZB71dqRHWL8QoZF\_iUbp-8CmU6HaD6MjWD2-Cts-M/edit).

![](<../../.gitbook/assets/image (21).png>)

以下說明每個步驟中的關鍵要點:

**Step1: Create Publication Distribution**\
在這個階段建立 散發集(Distribution) 以及 發行集(Publication) 選擇什麼資料庫以及什麼Table需要搬遷並加入發行集即可\
**重點1:** 這個發行集一定要 勾選/啟用 以備份初始化訂閱 (@allow\_initialize\_from\_backup) \
**重點2:** Table需要具備Primary Key才可以加入發行集進行交易式複寫.

**Step2: Make a DB full backup**\
進行MSSQL native的BAK完整備份,這個過程無須downtime, 切記這個備份一定要在這個資料庫設定成發行集(Publication) **Step1** **之後** 做, 才可以用來正常初始化訂閱!

**Step3: Copy DB full backup to GCS**\
使用 [gsutil cp](https://cloud.google.com/storage/docs/gsutil/commands/cp) 即可成功上傳到GCS

**Step4: Import DB full backup bak to CloudSQL**\
因為進行Import動作的是CloudSQL service account 而不是使用者的IAM account.  若發生權限不足的狀況, 確認 CloudSQL SQLServer 的serviceAccountEmailAddress, 至少需要具備存取GCS的Storage Legacy Object Reader權限

**Step5: Create subscription to CloudSQL**\
這個動作是在 發行集(Publication) 的 SQLServer執行 Push Subscription\
**重點1:** 須將Step2的BAK檔案放在 發行集(Publication) 的 SQLServer可以存取到的路徑, 因為SQLServer需要讀取這個BAK紀錄的LSN來得知要從哪個資料點開始接著做Replication. \
**重點2:** 必須用SQL指令執行, 用SSMS介面只能做用snapshot初始化訂閱,當資料庫很大的時候相當費時. &#x20;

以上就是完整步驟說明.
