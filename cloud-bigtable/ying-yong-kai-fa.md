# 应用开发

### Java 访问Cloud Bigtable和访问HBase有啥区别吗?

Bigtable提供了主要开发语言的客户端驱动，当然也提供了Java的原生客户端驱动，尤其是为了兼容Java HBase的客户端驱动的接口，Bigtable提供了[Java bigtable hbase](https://github.com/googleapis/java-bigtable-hbase)这样子的wrapper，兼容了Java HBase的接口，使得可以像使用Java HBase一样的访问Bigtable。

### CAS(Compare-and-set) 在Cloud Bigtable支持吗？

这些都是基本功，没问题的。举个例子，例如你用Golang 访问HBase，代码大概写法是：

{% code overflow="wrap" %}
```
rs, _ = client.CheckAndMutate(defaultCtx, tableInbytes, []byte("row1"), []byte("f"), []byte("q1"), op, []byte("value1"),
           &hbase.TRowMutations{Row: []byte("row1"), Mutations: []*hbase.TMutation{&hbase.TMutation{Put: &hbase.TPut{Row: []byte("row1"), ColumnValues: []*hbase.TColumnValue{&hbase.TColumnValue{
               Family:    []byte("f"),
               Qualifier: []byte("q1"),
               Value:     []byte("value11")}}}}}})
               
rs, err = client.CheckAndPut(defaultCtx, tableInbytes, []byte("row1"), []byte("f"), []byte("q2"), nil, &hbase.TPut{Row: []byte("row1"), ColumnValues: []*hbase.TColumnValue{&hbase.TColumnValue{
       Family:    []byte("f"),
       Qualifier: []byte("q2"),
       Value:     []byte("value1")}}})

```
{% endcode %}

那在Golang访问Bigtable，使用Bigtable的Golang的客户端驱动，代码大概写法是：

```
conditionalMutation := bigtable.NewCondMutation(filter, nil, mut)
   matched := true
   applyOption := bigtable.GetCondMutationResult(&matched)
    rowKey := "row1"
    rs := tbl.Apply(ctx, rowKey, conditionalMutation, applyOption)
     if matched {
       fmt.Printf("Updated!")
   } else {
       fmt.Printf("No Updated!")
   }
```

### Bigtable支持二级索引吗？

不支持的。有一些Workaround，例如，通过写入两张表，两张表的主键不一样，满足不同的查询需求。

### 大批量删除数据(Bulk)的 code example (Golang)

```
func scanAndDelete(tableName string, rowKeyPrefix string, batchSize int) {

  // read set of row keys, then delete batch
  tbl := client.Open(tableName)

  // loop through range in increments of batchSize
  numDeletes := 0
  for true {
    s := make([]string, 0)

    // identify row keys from range for deletion
    err = tbl.ReadRows(ctx, bigtable.PrefixRange(rowKeyPrefix), func(row bigtable.Row) bool {
      s = append(s, row.Key())
      return true
    }, bigtable.RowFilter(bigtable.ChainFilters(bigtable.CellsPerRowLimitFilter(1), bigtable.StripValueFilter())), bigtable.LimitRows(int64(batchSize)))

    // no rows remaining to delete
    if len(s) == 0 {
      break
    }

    // create delete row mutations
    muts := make([]*bigtable.Mutation, len(s))

    for i := range s {
      muts[i] = bigtable.NewMutation()
      muts[i].DeleteRow()
      numDeletes++
    }

    // commit deletes in batch
    d_rowErrs, d_err := tbl.ApplyBulk(ctx, s, muts)

    if d_err != nil {
      log.Fatalf("Could not apply bulk row mutation: %v", d_err)
    }

    if d_rowErrs != nil {
      for _, rowErr := range d_rowErrs {
        log.Printf("Error deleting row: %v", rowErr)
      }
      log.Fatalf("Could not delete some rows")
    }

    // add a sleep (if desired)
    // time.Sleep(0 * time.Second)
  }

  fmt.Printf("Completed deletion for Table: %s, prefix id: %s, num rows deleted: %d\n", tableName, rowKeyPrefix, numDeletes)
}
```

### Bigtable支持Thrift server吗？

Thrift server是HBase中的一种服务，主要用于对多语言API的支持。基于[Apache Thrift](https://thrift.apache.org/)（多语言支持的通信框架）开发，目前有两种版本[thrift](https://github.com/apache/hbase/blob/master/hbase-thrift/src/main/resources/org/apache/hadoop/hbase/thrift/Hbase.thrift)和[thrift2](https://github.com/apache/hbase/blob/master/hbase-thrift/src/main/resources/org/apache/hadoop/hbase/thrift2/hbase.thrift)。

Bigtable默认不支持Thrift server。如果用户之前代码是通过Thrift server访问HBase的，那么可以参考这个[Github Repo](https://github.com/hellof20/bigtable-thrift)不修改代码的情况下访问Bigtable，也可以实现用原生的HBase shell访问Bigtable.
