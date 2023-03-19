# Columnar Engine

### 如何确认语句是否使用了 Columnar Engine？

可以在 query plan 中查看，是否有“Custom Scan (columnar scan)”字样

```
EXPLAIN (VERBOSE OFF, COSTS OFF, TIMING OFF, SUMMARY OFF)

select sum(l_quantity) from lineitem_t group by l_quantity::text, l_orderkey order by 1;
                             QUERY PLAN
----------------------------------------------------------------------
Sort
  Sort Key: (sum(l_quantity))
  ->  HashAggregate
        Group Key: ((l_quantity)::text), l_orderkey
        ->  Append
              ->  Custom Scan (columnar scan) on lineitem_t
                    Columnar cache search mode: columnar filter only
              ->  Seq Scan on lineitem_t

(8 rows)
```

### 如何查看 Columnar Engine 分配的内存？

通过 google\_columnar\_engine.memory\_size\_in\_mb 查看

```
ethereum=> CREATE EXTENSION google_columnar_engine;
CREATE EXTENSION

ethereum=> show google_columnar_engine.memory_size_in_mb;
 google_columnar_engine.memory_size_in_mb 
------------------------------------------
 19295
(1 row)
```

### 如果我的数据库大小比 Columnar Engine 配置的要大怎么办？

建议先通过推荐功能（recommendation) 来判断哪些表以及具体的哪些列应该被缓存到Columnar Engine 中，以达到最佳性能。如果推荐的表和列的大小超过了 Columnar Engine 的内存配置，我们将尽可能多的填充 block 以适应内存。查询将透明的使用 row store（行式存储）来查询剩余的部分。

### 如何手动将表加入到 Columnar Engine？

在上边的内存分配示例中，整个的 columnar engine 占用了 19295MB 的内存，在手动将 ‘transactions' 表 load 到 columnar engine 中后，这个表占用了 15155MB 的内存。如果占用内存为 19295，很有可能表中的数据没有全部 load 到引擎中。

```
ethereum=> SELECT google_columnar_engine_add('transactions');
-[ RECORD 1 ]--------------+------
google_columnar_engine_add | 15155
```

### 如何查看 Columnar Engine 中缓存了哪些列？

```
SELECT * FROM g_columnar_columns;
```

### 如何查看 Columnar Engine 针对最近执行过的语句所做的执行统计？

可以通过 `g_columnar_stat_statements` 进行查看

```
// Enable the pg_stat_statements extension:
CREATE EXTENSION pg_stat_statements;

SELECT * FROM g_columnar_stat_statements WHERE columnar_unit_read > 0;
```
