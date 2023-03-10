# 性能与监控

### &#x20;如何判断MemoryStore for Redis遇到了性能瓶颈？

通常来说，如果内存使用率没有满的话，Redis的性能瓶颈通常就是在CPU上。很直观就可以看到CPU使用率达到90%以上甚至100%。但是，由于Redis 6开始支持multi-threaded I/O，所以Redis可以用到了多个Core了，而不像以前的版本只能用到一个Core。不过，不是所有的Redis操作都能够用到multi-threaded I/O。举个例子，这里有Redis 6.x的实例使用到6个Core的计算资源，由于不是所有的操作都能用到multi-threaded I/O，所以Redis在这里是没办法用满这些计算资源的，或者来说，不太容易找到一个CPU百分比来监控CPU的资源使用情况，因为Redis自己没办法用满CPU。

所以，目前MemoryStore for Redis提供了CPU Seconds来监控CPU的资源使用情况。当观察到CPU Seconds接近于下表的I/O Threads时，可以认为CPU遇到性能瓶颈。

![](../.gitbook/assets/image.png)

### 如何理解MemoryStore for Redis的内存使用情况？

举个例子来解释，当你在Console创建一个16GB的Instance的时候，你创建的Redis的实例的maxmemory size就是16GB，而运行这个Redis的实例的计算节点的内存是21.52GB。可以发现，你创建一个16GB的实例，
