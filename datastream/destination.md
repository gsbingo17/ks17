# Destination

## BigQuery

### 如何理解并调整 BigQuery 的 Staleness

Datastream 会持续监控 Source 数据库中的数据变化，并将这些变化通过 Write API 即刻写入到 BigQuery 中。 Staleness 限制（也就是 BigQuery 中的 `max_staleness`）是指 BigQuery CDC 机制将这些更改应用（合并）到现有数据的频率。

一旦在 BigQuery 将表创建成功，用户可以通过 **ALTER TABLE** 语句对 max\_staleness 进行修改。以下示例是将 ‘employees’ 表的 `max_staleness` 修改到 15 分钟。

```sql
ALTER TABLE employees
SET OPTIONS(
  max_staleness = INTERVAL 15 MINUTE
);
```

### 如果表中没有主键，如何手动 merge 数据

在 BigQuery 直接作为 destination 时，如果source端的表没有 primary key，新写入的数据会以 append-on 的形式存在：

| Row | id | name | datastream\_metadata.uuid            | datastream\_metadata.source\_timestamp | datastream\_metadata.is\_deleted |
| --- | -- | ---- | ------------------------------------ | -------------------------------------- | -------------------------------- |
| 1   | 1  | aaa  | ba516709-b9b6-476e-8d14-08bf00000000 | 1668495657000                          | false                            |
| 2   | 2  | bbb  | ba516709-b9b6-476e-8d14-08bf00000001 | 1668495657000                          | false                            |
| 3   | 3  | c    | ba516709-b9b6-476e-8d14-08bf00000010 | 1668495657000                          | false                            |
| 4   | 4  | cd   | ba516709-b9b6-476e-8d14-08bf00000011 | 1668495657000                          | false                            |
| 5   | 5  | cd   | ba516709-b9b6-476e-8d14-08bf00000100 | 1668495657000                          | false                            |
| 6   | 3  | jz   | 4285ed1c-1323-4416-90b9-032900000000 | 1669181821000                          | false                            |
| 7   | 5  | cd   | cabe34af-9c13-4dac-93bf-57ad00000000 | 1669185018000                          | true                             |
| 8   | 6  | jun  | 3fac2b8f-b582-48b8-b0b3-22d900000000 | 1669185121000                          | false                            |

如果需要得到 Merge 后的数据，可以手动执行以下语句：

```sql
WITH final AS
(SELECT DISTINCT id
               , name
               , DATETIME(TIMESTAMP_MILLIS(CAST(datastream_metadata.source_timestamp as INT64)),"Asia/Shanghai") as Updated_timestam
               , datastream_metadata.is_deleted
               , RANK() OVER (PARTITION BY id ORDER BY datastream_metadata.source_timestamp DESC) AS finish_rank
           FROM `rock-collector-342013.awsmysql.Mu_test1019`)
 
SELECT id
     , name
 FROM final
WHERE finish_rank=1
AND is_deleted=false
ORDER BY id DESC;sql
```

### BigQuery Destination 中的时间转换

在 BigQuery 中，datastream\_metadata.source\_timestamp 是由 unix 时间戳表示的 UTC 时间，可通过以下语句转换成可读时间以及进行 timezone 转换

```sql
select id
     , name
     , DATETIME(TIMESTAMP_MILLIS(CAST(datastream_metadata.source_timestamp as INT64)),"Asia/Shanghai") as Updated_timestamp
     , datastream_metadata.source_timestamp
     , datastream_metadata.is_deleted 
 from `rock-collector-xxxxxx.awsmysql.Mu_test1019`;
```
