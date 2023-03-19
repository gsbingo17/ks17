# MySQL主从复制优化

MySQL的主从复制的延时是一个常见的挑战，如何优化主从复制，减少复制延时呢？通常的做法如下：

* 如果是MySQL 5.7之后，通过配置Database flags：slave\_parallel\_type = LOGICAL\_CLOCK，slave\_parallel\_workers = number， 不要大于MySQL实例中的数据库个数，启用并发复制。大部分情况下，效果都非常好。
* 检查主要的表，都应该有主键或者唯一索引
* 尽量减少大的事务，例如批量删除大量的数据，很容易造成大的主从复制延时
* 从库可以考虑配置sync\_binlog、innodb\_flush\_log\_at\_trx\_commit为非1，来减少I/O开销，提升复制速度
