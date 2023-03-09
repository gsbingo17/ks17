# Redis 实例缩容

* Redis RAM 从20G 缩容到8G ，错误如下所示：

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption><p>重新调整容量错误提示</p></figcaption></figure>

* 解决方法：

默认Redis 实例是**启用RDB 快照**的，关闭改配置后再重新调整容量即可：

**注意**：调整后的容量至少要预留出当前数据更多容量，比如实际用量是5.5G，建议调整后的容量到6.5G；

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption><p>停用实例快照功能</p></figcaption></figure>
