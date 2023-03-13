---
description: >-
  採用 GCP 提供的 DMS 可相當方便的搬遷至CloudSQL MySQL
  https://cloud.google.com/database-migration/docs/mysql/quickstart
  以下提供使用DMS過程常遇到的問題與解法 以及 特別的要點
---

# MySQL

### **Failure connecting to the source database**

若是在 Test migration Job 過程, 發現Error: **Failure connecting to the source database** \
![](<../../.gitbook/assets/image (1) (1).png>)

這樣代表無法從 CloudSQL端 連線到 來源資料庫主機.\
**PrivateIP環境:** 進入跟 CloudSQL 做Peering的VPC後, 點選 Private Service Connection --> Allocated Ranges For Services\
找到 CloudSQL 的 IP Range之後 將其加入 來源資料庫主機 網路環境中的 Firewall allow ingress 即可

![](<../../.gitbook/assets/image (36).png>)\
\
**PubliceIP環境:** 為了取得 CloudSQL 真正用來連線到 來源資料庫主機 的 PublicIP, 可到CloudSQL instance的頁面, 如下方截圖的位置取得PublicIP, 將其加入 來源資料庫主機 網路環境 的 Firewall allow ingress 即可\
![](<../../.gitbook/assets/image (11).png>)\


## Definer is not supported

若是在 Test migration Job 過程, 發現開頭為 Definer is not supported... 的Error\
\
![](<../../.gitbook/assets/image (12).png>)\
**Definer is not supported. Definer user root@localhost not supported. Please update host to '%'.**\
代表來源MySQL資料庫有View,Trigger等是由root@localhost建立的,所以在create script會有DEFINER=\`...\`的字眼\
但CloudSQL MySQL (以及一般DBaaS) 都無開放root@localhost權限,故無法成功以此權限建立

解法為使用 [文件](https://cloud.google.com/database-migration/docs/mysql/mysql-definer) 中的指令 找出有使用哪些DEFINER

{% code overflow="wrap" %}
```
SELECT DISTINCT DEFINER FROM INFORMATION_SCHEMA.EVENTS WHERE EVENT_SCHEMA NOT IN ('mysql', 'sys'); 

SELECT DISTINCT DEFINER FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA NOT IN ('mysql', 'sys'); 

SELECT DISTINCT DEFINER FROM INFORMATION_SCHEMA.TRIGGERS WHERE TRIGGER_SCHEMA NOT IN ('mysql', 'sys'); 

SELECT DISTINCT DEFINER FROM INFORMATION_SCHEMA.VIEWS WHERE TABLE_SCHEMA NOT IN ('mysql', 'sys');
```
{% endcode %}

並把這些View,Trigger等 改建立為 非root帳號 的DEFINER即可



![](<../../.gitbook/assets/image (16) (2).png>)\
**Definer is not supported. Definer user MySQLXXX@% does not exist. Please create the user on the replica.**\
****若出現的訊息為Definer user MySQLXXX@% does not exist,代表這些View,Trigger等 已經改成特定MySQLUser 的Definer,但是這個MySQLUser 尚未在CloudSQL MySQL存在

解法為 將這個MySQLUser 建立到這個DMS新建立的CloudSQL MySQL之後, 再啟動DMS即可\
![](<../../.gitbook/assets/image (3).png>)\
\


## Lost Connection to MySQL during Query

若是在migration已經開始,但卻在跑一段時間後出現Error: \
Lost Connection to MySQL during Query \
![](<../../.gitbook/assets/image (2) (3).png>)

這樣代表來源MySQL資料庫有些size較大的table, 持續做Fulldump的過程較久,導致被MySQL判定太久沒有回應為超時timeout

解法為在來源MySQL資料庫 my.cnf 增加/調整 底下參數\
\[mysqld] \
net\_read\_timeout=3600\
net\_write\_timeout=3600\
max\_allowed\_packet=1G

