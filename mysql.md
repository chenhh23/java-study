#### 存储引擎
尽管存储引擎可以管理自己的锁，Mysql本身还是会使用各种各样的表锁来实现不同的目的。

##### innoDB
特点：事务管理、行级锁、灾难恢复

##### Myisam
插入查询比较多

##### Memory
不能持久化，数据保存在内存中

##### Merge 
多个Myisam表的合并