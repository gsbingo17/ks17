---
description: GCP的DMS已經開始支援 Oracle to CloudSQL PostgreSQL public preview
---

# Oracle到PostgreSQL的迁移

### 哪些[Oracle数据库](https://cloud.google.com/database-migration/docs/oracle-to-postgresql/migration-src-and-dest)可以用DMS迁移？

* Oracle 11g, Version 11.2.0.4&#x20;
* Oracle 12c, Version 12.1.0.2&#x20;
* Oracle 12c, Version 12.2.0.1&#x20;
* Oracle 18c&#x20;
* Oracle 19c

### 首先我們來看GCP提供的 Oracle to PostgreSQL migration journey ![](<../../.gitbook/assets/image (38).png>)

### Assessment&#x20;

Oracle DB assessment, GCP使用的是 partner tool **migVisor**\
從申請到操作的步驟都可參考此 [**文件**](https://docs.google.com/document/d/1x7n1YHituoLfTxwf2Dhcm5oUNr2kcgfgxZHndo5Zq8k/edit) , migVisor 有許多特點如下:

* 僅需相當小且Read-only的Oracle user權限就可以收集到來源Oracle的meta-data, 協助判讀migration的複雜度
* 操作十分簡單, 僅需在可以連線到來源Oracle的任何主機, 安裝一個migVisor\_MetadataCollector, 收集完成之後直接上傳到 Web portal 再開帳號給 Googler or Partner檢閱即可.  十分安全且保密.
* 從 Web portal 觀看 assessment 的結果之後, 即可評估哪些Oracle schema搬遷到PostgreSQL的複雜度, 並得知是哪些Oracle feature, DB object 影響了搬遷的難易度
* migVisor 目前由Google提供給客戶免費使用

### Schema conversion

這個階段會有兩個重要的tool, ora2pg & Ispirer

* **ora2pg** 是相當知名的免費 open source tool, 專注於Oracle to PostgreSQL的 Table schema, View, Trigger, Stored Procedure等等的轉換. 在使用 DMS 的過程就需要客戶使用 ora2pg 來手動進行 Table schema的轉換, 並將轉換後的Table schema 先建立在CloudSQL PostgreSQL.  接著再將 ora2pg.conf 上傳到 DMS 協助判別哪些Table需要搬遷. ora2pg的安裝可參考此 [**文件**](https://docs.google.com/document/d/140YEJkUtwvOT6fFCEdmhgdqA9MX97UURJMUAn\_OHBQE/edit)****
* **Ispirer** 是 partner tool 在 View, Trigger, Stored Procedure的PL/SQL to PL/pgSQL 轉換上特別出色, 若客戶發現ora2pg的轉換效果不好, 便可使用此tool, 此工具是需要付費的.

### Data Migration

參考 [GCP文件](https://cloud.google.com/database-migration/docs/oracle-to-postgresql/quickstart) 進行 Oracle to CloudSQL PostgreSQL migration

### Application Validation

當Oracle to PostgreSQL搬遷完成後, 由客戶的開發團隊改寫程式後連線到PostgreSQL確認功能面的驗證\
