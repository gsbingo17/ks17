# 创建实例

### AlloyDB 对 PostgresSQL 的兼容性如何？

AlloyDB 100% 兼容 PostgreSQL， 用户可以在不更改任何 code 的情况下使用 AlloyDB。

* [支持的数据库 extensions](https://cloud.google.com/alloydb/docs/reference/extensions)
* [支持的数据库 flags](https://cloud.google.com/alloydb/docs/reference/database-flags)
* [AlloyDB 独有的 extensions 和 flags](https://cloud.google.com/alloydb/docs/reference/alloydb-flags)

### AlloyDB 支持那些 PostgreSQL 版本？是否可以选择具体的某个小版本？

AlloyDB 支持 PostgreSQL 14。暂时不支持对特定小版本进行选择。

### AlloyDB 是否需要使用特殊的 ODBC/JDBC?

不需要，AlloyDB 完全适用现有的 ODBC 和 JDBC connectors。

### 在哪些 region 可以使用 AlloyDB？支持那些机型？是否可以进行自动扩展？

* 现在可以在多达 [22 个 region](https://cloud.google.com/alloydb/docs/locations) 中使用 AlloyDB
* 支持从 2 vCPU, 16 GB 到 64 vCPU, 512 GB 配比的机型
* AlloyDB 的扩展分为三个方面
  * Primary 实例可以做纵向上的机型扩展
  * Read Pool 出了可以进行纵向上的机型扩展，还可以通过增加 read pool 节点（最多 20 个）进行横向
  * 存储根据用量进行自动伸缩。不同于 Cloud SQL，在 AlloyDB，不光可以根据存储内容的增多自动扩展存储，在数据删除后，删除数据所占用的空间也会自动释放，总存储量降低。不再根据配置存储容量收费，而是根据实际用量，实现成本优化。
