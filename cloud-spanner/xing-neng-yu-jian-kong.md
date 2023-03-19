# 性能与监控

### 怎么理解Spanner的性能？

Spanner解决了数据库的横向扩展能力，可以随着工作负载的增加，提供性能扩展，并且保证稳定读写延时。举一个通俗的例子，在每秒几百笔的请求下，Cloud Spanner和其他的数据库的性能水平是一样的，但是当每秒请求到了几万笔或者几十万笔的时候，Cloud Spanner还能够提供稳定的性能，可能其他的数据库已经由于无法扩展，而支撑不了这个性能了。

### 如何利用Spanner的性能？

Spanner自动实现数据库层面的分片功能，通过数据库分片达到性能的横向扩展。在具体使用中，需要对表配置主键（Primary Key），表是按照主键进行自动分片。这里的关键的考虑的地方在于：

* 选择合适的Primary Key，可以参考这篇[文档](https://cloud.google.com/spanner/docs/schema-and-data-model#choosing\_a\_primary\_key)。另外，在Schema Design上，这里有一篇博客《[关于 Cloud Spanner，DBA（数据库管理员）需要了解哪些内容？第 1 部分：键和索引](https://www.infoq.cn/article/eiyek5ga1evqr20ttuha)》，值得一读。
* 需要足够多的数据或者负载。分片会基于存储容量和工作负载产生新分片和分片在多个Spanner Node上的均衡。举个通俗的例子，当表之有1个GB的时候，就很难产生新分片；当负载仅仅每秒几百个请求的时候，也很难触发产生新分片，所以当数据量小，负载低的时候，是不能有限产生更多的分片的。举个例子，由于分片少，尽管你有10个node，你也无法用到这个10个node的计算资源的。所以，通常实践是，在大数据量的情况下，例如几百GB，通过提前预热Pre-warming，达到产生足够的分片，并均衡到所有的node上的效果。这在一些基于Spanner开发的大型应用上线的时候广泛使用，可以参考这篇[文档](https://cloud.google.com/blog/products/databases/cloud-spanner-makes-application-launches-easier-with-warmup-and-benchmarking-tool)。

### Spanner的Interleaved Table是干嘛的呀？

当多个表需要频繁Join查询的时候，由于Spanner是一个分布式数据，为了避免跨分片的Join的情况，这种情况性能开销会非常大，我们通常推荐是使用[交错表](https://cloud.google.com/spanner/docs/schema-and-data-model?hl=zh-cn#parent-child)Interleaved table，使得这些需要频繁Join的表能够保存在同一个分片，不会出现跨分片Join。交错表是Spanner比较独特的表，创建的语法可以参考如下：

```
CREATE TABLE Singers (
 SingerId   INT64 NOT NULL,
 FirstName  STRING(1024),
 LastName   STRING(1024),
 SingerInfo BYTES(MAX),
 ) PRIMARY KEY (SingerId);

CREATE TABLE Albums (
 SingerId     INT64 NOT NULL,
 AlbumId      INT64 NOT NULL,
 AlbumTitle   STRING(MAX),
 ) PRIMARY KEY (SingerId, AlbumId),
INTERLEAVE IN PARENT Singers ON DELETE CASCADE;
```

交错表最多可以有7层，你看了上面的例子，它是用的Pre-join的办法，子表的主键包含了父表的主键的。

需要补充的是，如果是不需要频繁Join查询的表，Spanner也是支持传统的关系型数据库所使用的[外键](https://cloud.google.com/spanner/docs/foreign-keys/overview?hl=zh-cn)关联的这种更加通用的做法，创建的语法可以参考如下：

```
CREATE TABLE OrderItems (
  OrderID INT64 NOT NULL,
  ProductID INT64 NOT NULL,
  Quantity INT64 NOT NULL,
  FOREIGN KEY (ProductID) REFERENCES Products (ProductID)
) PRIMARY KEY (OrderID, ProductID),
  INTERLEAVE IN PARENT Orders ON DELETE CASCADE;
```

### 能介绍一下Spanner的索引吗

Spanner的每一张表都有主键，主键就是索引；当需要支撑个多查询的时候，需要创建二级索引。

需要注意的地方是，二级索引的字段的选择和表的主键选择一样，也要避免出现热点，也要尽量均衡。

创建二级索引通常也是耗费时间的，但Spanner对创建二级索引做了类似限流的设计，尽量减少了对数据库整体性能的影响。

创建二级索引的语法参考：

```
//基本的创建
CREATE INDEX SingersByFirstLastName ON Singers(FirstName, LastName)
//二级索引可以带入更多的数据，这样子可以一次返回所需的信息，不用去Base table取得数据，例如这个例子中
//需要查询MarketingBudget
CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget);
```

查看创建索引的进度，这个也是很多人关心的，到底还要多久可以创建完；有些做法是，为了加速创建索引，可以增加多个节点；创建完之后，再减少增加的节点。Spanner的通过增加和减少节点实现扩容和缩容很简单。详细的信息请查看这个[文档](https://cloud.google.com/spanner/docs/secondary-indexes?hl=zh-cn#index-progress)。

```
gcloud spanner operations list --instance=INSTANCE --database=DATABASE

gcloud spanner operations describe _auto_op_123456 \
    --instance=INSTANCE \
    --database=DATABASE
```

### Spanner提供了哪些具体的监控信息吗？

* System insights
* Query insights
* Transaction insights
* Lock insights

你都可以在Console直接使用上面这些直观易用的监控图形化工具。

### Spanner提供执行计划吗？

当然有呀！最简单的方法就是在Console上执行查询语句并查看执行计划。可以看看下图就明白了。

![](https://lh3.googleusercontent.com/sMuYLccMrXx9McymsTxKJJM2pXi0D6wwmAb-RJMcYw7TpQ-Jd2ehL2zBJ5sUTZ4572Vd1T7GuBC5d7XqHU5Zs5YTGKaRdx4exe5QFm5A2U7fhkIvBWcFXA1pa12u\_Wx2sLqmu3bWimzQHKS5HOrDWZ\_XoX-ne-MUrjhWh6PfSUEuOdGqK\_eV8lAld5iHkMnx6PCCGRdmhqHiNFLPoU6b-d-IATBrLVCqIrkF8g)

### Spanner有maintenance dowtime吗？

没有！
