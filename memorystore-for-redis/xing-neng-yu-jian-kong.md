# 性能与监控

### &#x20;如何判断MemoryStore for Redis遇到了性能瓶颈？

通常来说，如果内存使用率没有满的话，Redis的性能瓶颈通常就是在CPU上。很直观就可以看到CPU使用率达到90%以上甚至100%。但是，由于Redis 6开始支持multi-threaded I/O，所以Redis可以用到了多个Core了，而不像以前的版本只能用到一个Core。不过，不是所有的Redis操作都能够用到multi-threaded I/O。举个例子，这里有Redis 6.x的实例使用到6个Core的计算资源，由于不是所有的操作都能用到multi-threaded I/O，所以Redis在这里是没办法用满这些计算资源的，或者来说，不太容易找到一个CPU百分比来监控CPU的资源使用情况，因为Redis自己没办法用满CPU。

所以，目前MemoryStore for Redis提供了CPU Seconds来监控CPU的资源使用情况。当观察到CPU Seconds接近于下表的I/O Threads时，可以认为CPU遇到性能瓶颈。

![](../.gitbook/assets/image.png)

### 如何理解MemoryStore for Redis的内存使用情况？

举个例子来解释，当你在Console创建一个16GB的Instance的时候，你创建的Redis的实例的maxmemory size就是16GB，而运行这个Redis的实例的计算节点的内存是21.52GB。可以发现，你创建一个16GB的实例，MemoryStore提供了一个有21.52GB内存的计算节点，其中的75%，也就是16GB是Redis最大的使用内存。

在这个16GB的最大使用内存中，如果有主从复制，其中的10%，也就是1.6GB是用于主从复制的日志的缓存的。另外，如果是使用了RDB，也需要考虑做RDB的时候，如果有较多的修改操作，需要额外的内存用于copy-on-write。这些是内存使用情况的基本考虑。

Redis内存也有出现碎片的情形，造成看上去内存使用率不高，但Redis的操作异常，这个时候需要打开参数[activedefrag](https://cloud.google.com/memorystore/docs/redis/memory-management-best-practices#memory\_fragmentation)。简单的做法，就是在MemoryStore Console上打开这个参数。

### 我在监控MemoryStore for Redis，有什么具体的监控阀值吗？

还是以上面的例子来聊，16GB的Instance的系统内存有21.52GB，正常情况下，内存使用达到21.52GB的80%，请及时关注；在做Mainenance、导出操作、升配置等，尽量选择业务低峰期，内存使用控制在21.52GB的50%左右，这里主要是由于会涉及到RDB的数据备份，需要预留足够的内存完成RDB操作。\
