# Oracle到BigQuery的迁移

使用Datastream可以完成从Oracle到BigQuery的迁移。举个例子，可以使用BigQuery作为新的数据仓库。详细的信息可以参考这个[文档](https://cloud.google.com/blog/products/data-analytics/unlock-real-time-insights-from-your-oracle-data-in-bigquery)。

### 哪些[Oracle数据库](https://cloud.google.com/datastream/docs/sources-oracle#versionsfororaclesourcedb)可以用Datastream迁移？

* Oracle 11g, Version 11.2.0.4&#x20;
* Oracle 12c, Version 12.1.0.2&#x20;
* Oracle 12c, Version 12.2.0.1&#x20;
* Oracle 18c&#x20;
* Oracle 19c

### 源端数据库有什么基本要求吗？

* 啟用 [Oracle archive redo log](https://cloud.google.com/datastream/docs/configure-your-source-oracle-database#selfhostedoracle) 一般來說客戶端Production Oracle database都已經啟用, 若尚未啟用, 此動作會需要 重啟Oracle database.\
  ![](<../../.gitbook/assets/image (57).png>)
* 設定 Archive redo log retention day 保留一定天數的log檔案\
  GCP推薦是 4\~7 天\
  ![](<../../.gitbook/assets/image (2).png>)
* 啟用 Supplemental log, 才能讓redo log中紀錄完整的where filer幫助Datastream正確的將資料異動同步到目的地BigQuery\
  ![](<../../.gitbook/assets/image (27).png>)
* 設定合適的 **Archive frequency of Redo log**\
  這個動作是在Oracle才需特別注意的設定, 十分重要!\
  Oracle redo log --> Archived redo log --> Datastream --> BQ\
  **Datastream抓取的是 Oracle archived redo log 而不是直接抓取redo log的異動**, 若客戶十分關注資料異動同步到BigQuery的即時性, 我們有兩點可以建議給客戶\
  **1.** 設定 ARCHIVE\_LAG\_TARGET = 秒數, 來設定redo log變成Archived redo log的頻率 e.g. 600sec = 10min\
  **2.** 一般來說Oracle redo log都會由客戶DBA設定 數量 與 大小 持續 rotate, 被rotate出去的 redo log就會變成 Archived redo log\
  可以建議客戶找到最合適的Archived redo log產生頻率
