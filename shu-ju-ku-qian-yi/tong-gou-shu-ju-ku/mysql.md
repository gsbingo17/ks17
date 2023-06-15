---
description: >-
  採用 GCP 提供的 DMS 可相當方便的搬遷至CloudSQL MySQL
  https://cloud.google.com/database-migration/docs/mysql/quickstart
  以下提供使用DMS過程常遇到的問題與解法 以及 特別的要點
---

# MySQL

### 哪些[MySQL数据库](https://cloud.google.com/database-migration/docs/mysql/configure-source-database#overview-mysql)可以用DMS迁移？

* RDS 5.6，5.7，8.0
* 地端OP的MySQL 5.6，5.7，8.0
* Cloud SQL 5.6，5.7，8.0
* Aurora 5.6，5.7，8.0
* DMS利用MySQL Replication实现的数据库迁移，所以主要云商的RDS MySQL基本可以迁移

### 源端数据库有什么基本要求吗？

* Row-based binlog 要打开，并且保留足够长的时间能覆盖到数据库迁移的时间，通常可以设置7日
* 数据库的权限要够
* 网络要能够打通

### **Timeout waiting for no write traffic on source**

DMS在数据库迁移的第一步使用 mysqldump --single-tranaction进行导出，并保证数据一致性；single-transation是要求数据库实例“停写”很短的时间，然后开始这个导出操作。有些数据库实例非常繁忙，不能满足这个“停写”很短的时间，所以，通常的建议，是在源库建立一个启用binlog的副本，然后从这个副本作为源数据库配置DMS数据库迁移任务；在启动任务时，将源数据库主从复制停止一段时间；任务启动完成后，恢复源数据库主从复制。

### **Failure connecting to the source database**

若是在 Test migration Job 過程, 發現Error: **Failure connecting to the source database** \
![](<../../.gitbook/assets/image (45).png>)

這樣代表無法從 CloudSQL端 連線到 來源資料庫主機.\
**PrivateIP環境:** 進入跟 CloudSQL 做Peering的VPC後, 點選 Private Service Connection --> Allocated Ranges For Services\
找到 CloudSQL 的 IP Range之後 將其加入 來源資料庫主機 網路環境中的 Firewall allow ingress 即可

![](<../../.gitbook/assets/image (32).png>)\
\
**PubliceIP環境:** 為了取得 CloudSQL 真正用來連線到 來源資料庫主機 的 PublicIP, 可到CloudSQL instance的頁面, 如下方截圖的位置取得PublicIP, 將其加入 來源資料庫主機 網路環境 的 Firewall allow ingress 即可\
![](<../../.gitbook/assets/image (13).png>)

### Definer is not supported

若是在 Test migration Job 過程, 發現開頭為 Definer is not supported... 的Error\
\
![](<../../.gitbook/assets/image (11).png>)\
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

![](<../../.gitbook/assets/image (3).png>)

### Definer user MySQLXXX@% does not exist

若出現的訊息為Definer is not supported，Definer user MySQLXXX@% does not exist,代表這些View,Trigger等 已經改成特定MySQLUser 的Definer,但是這個MySQLUser 尚未在CloudSQL MySQL存在。

解法為 將這個MySQLUser 建立到這個DMS新建立的CloudSQL MySQL之後, 再啟動DMS即可\
![](<../../.gitbook/assets/image (20).png>)

### Lost Connection to MySQL during Query

若是在migration已經開始,但卻在跑一段時間後出現Error: \
Lost Connection to MySQL during Query \
![](<../../.gitbook/assets/image (28).png>)

這樣代表來源MySQL資料庫有些size較大的table, 持續做Fulldump的過程較久,導致被MySQL判定太久沒有回應為超時timeout

解法為在來源MySQL資料庫 my.cnf 增加/調整 底下參數\
\[mysqld] \
net\_read\_timeout=3600\
net\_write\_timeout=3600\
max\_allowed\_packet=1G

### 通过互联网DMS的速度大概在多少

经过实测从北弗吉尼亚的AWS Aurora到美中区域，1.2TB的数据大概36小时实现同步，传输速度大概在34GiB/小时。

### dump阶段出现报错如何处理

对于写操作比较繁忙的数据库，dump阶段可能会出现如下报错：

![](https://lh6.googleusercontent.com/N4Deo82PkPgGGdWMDT\_a0-5aNTRSFVnco8mYPTse3NxG3Gz9OyzOCw6Slqbc-9mxdE6q-SLDOyXvFUihTIWoWarMROpJ8CIDQnxIYxMyvwlnfKx4Hsc1HHek1GU7yHrIIwaBRCXr6CDmNcL4HOFWVDxDkclyt-yOpaadBVOWq6MtZOekaBv4m\_xeE7PdlaJaJK4HOoYCF4uaJryoYN1CNAUc1Qt0wv6grvaVbw)

如果show process list出现如下报错：

![](https://lh4.googleusercontent.com/PxX5KjQsxvzn3UhdUKjEVelZ7cnKmGlkwpSJsTmLEgMfsKqk67QZhxhIDmYhJHCQUdVVA4y3IT2AyQltXzOh4MaeUAfN9kFN4OvqrvMp6C3zKGhrv-0LGrKJgXs7L63Oe30phjyb37HX1Nzf\_X5hCkTjBQgIL3-YaBOTqxq2mUGDtaLAAmNYOOCcMzUhpm2E7uKPX-mlvuJdh\_w6n1Ps9gl5bQCxHZ8wfC8zBQ)

出现这种错误的原因是当DMS处在dump阶段时，需要在很短的时间内用读锁刷新表来创建快照。 但是如果有高吞吐量的写操作，会因为没有readonly时间窗口而导致dump失败。

解决的方法是：暂停写入操作高的应用程序，等dump的操作完成，快照创建以后，恢复这个应用程序的写入。

### DMS过程中可以有DDL的操作吗？

DMS分为两个阶段，dump同步阶段和CDC阶段。dump阶段时不允许DDL操作修改schema的，但CDC阶段可以允许DDL操作，并且会同步到Destination DB。

![](https://lh5.googleusercontent.com/0txow54JXM4FkZKa2NaWvUmRN31VoAySPf\_AL1NepApyg--PTcMFzszr6-wCFmZArR\_ml3vPHZ35ZQcgt39AtXAKZNhwDkbSZ50UFX0L039N4tP1l4pBwOCkGy0lMdyuOWcVgq2DeMbYb1Uu5tipws\_xHdO52Kf7TWnNdvlbCt6qMtf6Nn9-myif0VaE\_xEmgDeCiWUE5si3JcS4ZY8VunuOVvdjYSs8MQ-8hg)

方案：在dump阶段，暂停DDL操作；从DMS的monitorig里看到进入到CDC阶段时，放开DDL操作。\
