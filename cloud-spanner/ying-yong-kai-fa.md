# 应用开发

### 我有一个用MySQL开发的应用，现在想试试Spanner，有什么办法可以快速开始吗?

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
2. Read和Mutation的接口，[Read](https://cloud.google.com/spanner/docs/getting-started/java?hl=zh-cn#read\_data\_using\_the\_read\_api)接口是用于读取数据，[Mutation](https://cloud.google.com/spanner/docs/getting-started/java?hl=zh-cn#write-data-with-mutations)接口是用于写入数据。Read和Mutation的接口是可以更好满足读写低延时的需求。使用这两个接口读写时，就类似于把Spanner当作一个NoSQL在使用。这也是Spanner融合了关系型数据库和NoSQL数据库在具体使用中的表现。

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

通俗说，就是代码在读取数据的时候，可以保证读到最新的数据。说起来容易，但举一些例子可能更好理解，例如使用MySQL的数据库，你可能担心代码读取的数据是从Replica读到，数据不是最新的；使用NoSQL的时候，你读取并修改一条记录，你可能担心是不是有其他的会话也在同时读取并修改同一条记录。而这些情形在Spanner中，已经在数据库层面解决，数据库可以保证这种强一致性，
