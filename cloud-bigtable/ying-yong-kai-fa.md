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
