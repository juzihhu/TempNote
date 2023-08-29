# Note
## MySQL
### 事务隔离级别是怎么实现的？
1. 事务的特性 ACID
A: Atomicity 一个事务中的所有操作，要么全部完成，要么全部不完成(通过回滚日志保证)
C:Consistency 事务操作前后满足数据的完整性约束 （通过其他三者保证）
I:Isolation 多个事务并发执行 不会相互干扰 （MVCC或者锁机制保证）
D:Durability 事务执行结束，对事务的修改是永久的（通过重做日志保证）

2. 并行事务会引发什么问题？
在同时处理多个事务的时候，就可能出现脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题。
1.脏读（dirty read）：一个事务「读到」了另一个「未提交事务修改过的数据」
2.不可重复读（non-repeatable read）：
3.幻读（phantom read）
