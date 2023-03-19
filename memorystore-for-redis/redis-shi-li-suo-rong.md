# 实例缩容

举个例子，将MemoryStore Redis的内存大小从20G 缩容到8G ，可能遇到下面的错误提示：

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption><p>重新调整容量错误提示</p></figcaption></figure>

解决方法：

默认Redis 实例是**启用RDB 快照**的，RDB需要额外的内存资源；所以，可以先关闭RDB快照，然后再重新调整容量。

**注意**：调整后的容量至少要预留出当前数据更多容量，比如实际用量是5.5G，建议调整后的容量到6.5G；

<figure><img src="../.gitbook/assets/image (54).png" alt=""><figcaption><p>停用实例快照功能</p></figcaption></figure>
