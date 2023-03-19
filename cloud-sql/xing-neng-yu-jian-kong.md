# 性能与监控

### 如何找到慢的SQL？

找到慢的SQL有不少方法。以MySQL为例，常见的方法是使用MySQL提供的慢查询日志功能。使用下面的方法打开慢查询

1. 配置Database flags：slow\_query\_log, log\_output, long\_query\_time; 可以使用下面的命令来完成：

{% code overflow="wrap" %}
```
gcloud sql instances patch bin-test-mysql-db --database-flags=log_output='FILE',slow_query_log=on,long_query_time=2
```
{% endcode %}

2. 然后，前往Cloud Logging查看mysql-slow.log
3. 有时候，你希望以原始的格式查看slow log，可以使用下面的命令：

{% code overflow="wrap" %}
```
gcloud logging read projects/yourproject/logs/cloudsql.googleapis.com%2Fmysql-slow.log --format json| jq -rnc --stream 'fromstream(1|truncate_stream(inputs)) | .textPayload' > slow-log-timestamp.log
```
{% endcode %}

现在，有更好的方法找到慢的SQL了，因为Cloud SQL提供了[Query Insights](https://cloud.google.com/sql/docs/postgres/using-query-insights?hl=zh-cn)。使用它可以在图形界面快速找到慢的SQL。

### 内存使用率高是问题吗？

你可能发现Cloud SQL的内存使用率一直在攀升，使用率越来越接近100%，担心出现内存溢出。如下图所示：\
![](https://lh3.googleusercontent.com/Q\_mowtavd36kRxA\_bQaz64pbfLR48X9FoC-mzuiDrmAYVvdCinUuS0\_Vxnd48LbifvkJFzwfBI9PQF4p03rCaoOQTw6akCPAXY-y1kyc0T45SqWOwIbibfhOnWPwbP7VYClAPJsPTJ49At\_HJj89HmPmsw1Ugawzs\_DFtI-TznJ4DYxjBGyDLb3bZPLeFR-NmEMXKhh4ivic\_9yXlg6tMQQJy890loFTAiXFoQ)

其实内存的原理就是尽可能的分配出去，所以不用过于担忧；只要数据库负载没有急剧增长，就不会出现内存溢出的情况。

当然，如果数据库负载主要以写入为主，在MySQL数据库中，可以适当降低innodb\_buffer\_pool\_size来回收不需要的读的缓存，降低数据库内存使用率。关于MySQL内存使用，可以看看这篇[文章](https://cloud.google.com/mysql/memory-usage?hl=zh-cn)。

### Query Insights是干嘛的？

Query Insights可帮助检测、诊断和避免 Cloud SQL 数据库的SQL性能问题。它支持直观监控并提供诊断信息，帮助在检测范围之外确定性能问题的根本原因。你可以理解，Query Insights是主要优化SQL的工具，它不会产生额外费用，可以访问过去一周的数据。
