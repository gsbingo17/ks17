# SQL Server

## 使用 交易式複寫 + 由備份初始化訂閱 的方式 搬遷SQLServer 至 CloudSQL SQLServer

CloudSQL SQLServer 有支援 Microsoft 原生的複寫(replication)功能\
不論當作 [訂閱者(Subscriber)](https://cloud.google.com/solutions/migrating-data-from-sql-server-to-cloud-sql-for-sql-server) 以及 [發行者(Publisher)](https://cloud.google.com/sql/docs/sqlserver/replication/configure-external-replica) 都沒有問題\
大家可參考上面Link中, 都有詳細的指令與步驟

以下特別針對當 來源SQLServer資料庫很大(e.g. TB) 又希望能持續性的將資料異動同步到CloudSQL SQLServer, 以減少搬遷cut-over date的downtime時間.  這樣的情境下, 就推薦使用 [交易式複寫 + 由備份初始化訂閱](https://docs.google.com/document/d/12CZB71dqRHWL8QoZF\_iUbp-8CmU6HaD6MjWD2-Cts-M/edit) 的方式來搬遷到CloudSQL.

![](<../../.gitbook/assets/image (5).png>)

以下說明每個步驟中的關鍵要點:

Step1: Create Publication Distribution

Step2: Make a DB full backup

Step3: Copy DB full backup to GCS

Step4: Import DB full backup bak to CloudSQL

Step5: Create subscription to CloudSQL



