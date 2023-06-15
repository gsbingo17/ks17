# 目标

## BigQuery

### 当需要将数据同步到 BigQuery 时，直接到 BigQuery 和通过 Dataflow 到 BigQuery 有什么区别？

#### 从源端通过 Datasteam 直接到 BigQuery

这种方式可以以最低延迟（Datastream 的延迟<2mins) 将数据从源断复制到 BigQuery。如果源表上有 Primary Key，数据将直接在 BigQuery 段根据设定的 Staleness 进行 merge，得到 1:1 的复制；如果没有 Primary Key，则以 append-on 的形式进行复制。在 BigQuery 端将只有一个与源表对应的表。

#### 从源端经过 Datastream 将数据先存储到 Cloud Storage，在经过 Dataflow 处理，最终存储到 BigQuery

在使用这种方案时，由于数据经过了更多的组件，总体的延迟要略高于直接到 BigQuery 的方案。但是这种方案给数据的使用提供了更高的可靠性以及灵活性。首先当数据保存到 Cloud Storage 上之后，一方面对数据进行一次备份，同时为下游的其他服务使用数据提供了可能。 而 Dataflow 可以挂在 java 或者是 SQL scripts，可以对数据进行更灵活的格式以及逻辑上的调整。最后进入到 BigQuery 的时候，会分为两个表，其中的 replica table 是与源表 1:1 的复制表，另外还有一个 staging 表，将 DML 的操作以 append-on 的形式记录，可以为用户对数据进行 validation 或者会看历史记录提供便利。

### 现在 Dataflow 提供哪些 Datastream 模板？如何查看相关 template 的更新？

现在 Dataflow 提供 3 种 templates，可以支持将数据以 CDC 的方式同步到更广泛的终端：

* Datastream to BigQuery
* Datastream to Cloud Spanner
* Datastream to SQL (Cloud SQL MySQL / PostgreSQL)

以上templates 可以通过 Console，gcloud 以及 API 的方式直接调用，关于 templates 的更多内容，请见：

* [Google Cloud Dataflow Template Pipelines](https://github.com/GoogleCloudPlatform/DataflowTemplates)
* GitHub [GoogleCloudPlatform/DataflowTemplates/Release Note](https://github.com/GoogleCloudPlatform/DataflowTemplates/releases)

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

<table><thead><tr><th width="100">Row</th><th width="100">id</th><th width="100">name</th><th>datastream_metadata.uuid</th><th>datastream_metadata.source_timestamp</th><th>datastream_metadata.is_deleted</th></tr></thead><tbody><tr><td>1</td><td>1</td><td>aaa</td><td>ba516709-b9b6-476e-8d14-08bf00000000</td><td>1668495657000</td><td>false</td></tr><tr><td>2</td><td>2</td><td>bbb</td><td>ba516709-b9b6-476e-8d14-08bf00000001</td><td>1668495657000</td><td>false</td></tr><tr><td>3</td><td>3</td><td>c</td><td>ba516709-b9b6-476e-8d14-08bf00000010</td><td>1668495657000</td><td>false</td></tr><tr><td>4</td><td>4</td><td>cd</td><td>ba516709-b9b6-476e-8d14-08bf00000011</td><td>1668495657000</td><td>false</td></tr><tr><td>5</td><td>5</td><td>cd</td><td>ba516709-b9b6-476e-8d14-08bf00000100</td><td>1668495657000</td><td>false</td></tr><tr><td>6</td><td>3</td><td>jz</td><td>4285ed1c-1323-4416-90b9-032900000000</td><td>1669181821000</td><td>false</td></tr><tr><td>7</td><td>5</td><td>cd</td><td>cabe34af-9c13-4dac-93bf-57ad00000000</td><td>1669185018000</td><td>true</td></tr><tr><td>8</td><td>6</td><td>jun</td><td>3fac2b8f-b582-48b8-b0b3-22d900000000</td><td>1669185121000</td><td>false</td></tr></tbody></table>

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

