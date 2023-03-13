# 应用开发

### 我有个用MySQL开发的应用，现在想试试Spanner，有什么办法可以快速开始吗?

Cloud Spanner的社区提供了工具软件[Harbourbridge](https://github.com/cloudspannerecosystem/harbourbridge)，使用这个工具能够快速实现库表结构的转换和测试数据的迁移。具体操作参考如下：

{% code overflow="wrap" %}
```
harbourbridge schema-and-data -source=mysql -source-profile="host=localhost,user=gsbingo,password=hello,db_name=mydb" -target-profile="instance=testing,dbname=mydb1215"
```
{% endcode %}

目前这个工具支持的源数据库包括了MySQL、PostgreSQL和DynamoDB。

### Spanner提供了哪些读和写的客户端的接口，方便开发者使用？

Spanner提供了两大类的读和写的客户端接口：

1. SQL的[Query](https://cloud.google.com/spanner/docs/getting-started/java?hl=zh-cn#query\_data\_using\_sql)和[DML](https://cloud.google.com/spanner/docs/getting-started/java?hl=zh-cn#write-data)的接口，使用的方法和使用其他关系型数据库是一样；同时，也提供了Java JDBC的支持，正因为如此，也兼容常用的Java的ORM开源框架。
2. Read和Mutation的接口，[Read](https://cloud.google.com/spanner/docs/getting-started/java?hl=zh-cn#read\_data\_using\_the\_read\_api)接口是用于读取数据，[Mutation](https://cloud.google.com/spanner/docs/getting-started/java?hl=zh-cn#write-data-with-mutations)接口是用于写入数据。Read和Mutation的接口是可以更好满足读写低延时的需求。使用这两个接口读写时，就类似于把Spanner当作一个NoSQL在使用。这也是Spanner融合了关系型数据库和NoSQL数据库在具体使用中的表现。作为分布式数据库，出于性能的考虑是尽量控制交易的大小在一个合理的范围，所以每笔交易最大的Mutations是4万个，通常情况是够用的；如果了解更多信息，请查看这个[文档](https://cloud.google.com/blog/products/databases/cloud-spanner-doubles-the-number-of-updates-per-transaction)。

### Spanner的事务机制有什么特点，写代码的时候有什么需要注意的吗？

大家写代码都用过事务，各种开发语言的客户端都提供了事务相关的SDK，在基于Spanner的写代码时，主要都是一样的，这里主要聊聊不同的地方：

* Spanner的事务是外部一致性，说起来高级，可以通俗解释为，Spanner的事务是基于TrueTime发生，事务发生的时间和先后顺序是和外部真实世界的时间和先后顺序是完全一样。Spanner事务的时间和你的手表/手机的时间的时间是一样的，甚至更准确。TrueTime是基于原子钟/GPS授时实现的高精度的时间。
* Spanner主要提供了2种事务：读写事务和只读事务。
* 读写事务使用悲观锁和Wound Wait（老事务优先，而且直接停止新事务），所以，增加了Retry机制，让事务如果被“Wound Wait”之后，能够继续执行。
* 只读事务是没有锁的，已经使用的MVCC。所以，如果是读操作，请使用只读事务，减少锁冲突，提供性能。
* Retry机制是Spanner的应用开发比较特别，可以自定义超时和重试等机制。详细内容可以看看这个[文档](https://cloud.google.com/spanner/docs/custom-timeout-and-retry?hl=zh-cn)。
* Spanner的表支持提交时间戳，配合客户端驱动可以自动记录事务相关的时间，例如某条记录最后的修改时间等。例如创建表的时候，加上allow\_commit\_timestamp=true)

```
CREATE TABLE Performances (
    ...
    LastUpdateTime  TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
    ...
) PRIMARY KEY (...);
```

### Spanner提供了哪些ORM的支持？

1. Java: Hibernate, Spring Data; 同时，MyBatis,MyBatis-plus也测试过是可以支持的
2. Golang: gorm
3. Python: Django, SQLAlchemy

更多的信息可以访问官方文档的[相关部分](https://cloud.google.com/spanner/docs/use-hibernate?hl=zh-cn)。

### Spanner支持自增ID或序列吗？

不支持；Spanner是一个分布式数据库，如果使用自增ID或序列，通常在大规模数据库负载的时候会带来写入的热点，所以，Spanner暂时没有支持自增ID或序列。按照用户的经验来看，通常使用uuid等作为替代，也有使用Snowflake ID的情形。

### Spanner中是如何管理数据库连接池的？

Spanner的客户端驱动缺省就提供了数据库连接池，通常使用缺省的配置就足够了。缺省的配置在大部分的客户端驱动上为400个会话。当然，你也可以调整数据库连接池，这里有一个node.js的客户端驱动的[参考做法](https://github.com/cloudymoma/node-spanner-sessions-sample)，可以了解如何调整和查看会话数目。

### Spanner支持存储过程吗？

Spanner支持关系型数据库的绝大部分特性，但目前没有支持存储过程、自增ID/序列等。一般通过一些外部的脚本语言实现存储过程的功能。

### Spanner的强一致性，对于开发者来说，具体意味着什么？

通俗说，就是代码在读取数据的时候，可以保证读到最新的数据。说起来容易，但举一些例子可能更好理解，例如使用MySQL的数据库，你可能担心代码读取的数据是从Replica读到，数据不是最新的；使用NoSQL的时候，你读取并修改一条记录，你可能担心是不是有其他的会话也在同时读取并修改同一条记录。而这些情形在Spanner中，已经在数据库层面解决，数据库可以保证这种强一致性。

### 我的代码怎么连接上Spanner呀？需要创建用户吗？

Spanner是云原生数据库，用户鉴权使用是Cloud IAM，所以不需要创建数据库级别的用户的。

你在笔记本电脑上做代码，代码是通过互联网访问Spanner的。这个时候你可能发现SQL运行好慢，这个慢通常是网络延时造成的，不要担忧Spanner的性能。

### 我用Java写应用代码，使用JDBC访问Spanner数据库，请问JDBC的连接串写法是什么？

先看个例子哈

{% code overflow="wrap" %}
```
jdbc:cloudspanner:/projects/bin-learning-centre/instances/bin-testing/databases/demodb?minSessions=400;maxSessions=800;numChannels=8
```
{% endcode %}

详细参数配置，可以看看这个[文档](https://github.com/googleapis/java-spanner-jdbc)。

### 我想有一个图形化的工具可以访问Cloud Spanner，查看数据库的表结构、数据等，有推荐吗？

&#x20;[DBeaver](https://dbeaver.io/)可以试试。

### 我想找一些spanner周边的开发、测试、demo等工具类的软件，在哪里有呀？

Spanner有很多open source的软件，目前在[Github](https://github.com/orgs/cloudspannerecosystem/repositories)上。
