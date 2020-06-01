innodb引擎

###### Read commited和Repeatable Read的区别

1. Repeatable Read为默认隔离级别
2. 均使用了**非锁定的一致性读**

> 一致性的非锁定读（consistent nonlocking read）是指InnoDB存储引擎通过行多版本控制（multi versioning）的方式来读取当前执行时间数据库中行的数据。如果读取的行正在执行DELETE或UPDATE操作，这时读取操作不会因此去等待行上锁的释放。相反地，InnoDB存储引擎会去读取行的一个快照数据。

3. Read Commited读最新的历史版本，而Repeatable Read读事务开始时的历史版本。如下图，二者结果不同：

   (普通select均时非锁定读，但MVCC使用的快照版本不同)
   ![avatar](pic/read_comp.PNG)

   Read Commited读的结果为1,1,0；Repeatable Read读的结果为1,1,1。

4. Repeatable Read使用Next-Key Lock（Record Lock + gap Lock）算法来解决了**幻读**的问题，因此，已经满足了ACID中隔离性要求，不需要SERIALIZABLE级别。

   参考链接https://www.jianshu.com/p/04a2a6580bff