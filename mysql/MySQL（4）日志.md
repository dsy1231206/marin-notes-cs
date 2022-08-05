# MySQL（4）日志

##### Mysql日志分为undo log、redo log、binlog

redo log和binlog都了解过了，而undo log是为了事务回滚用的，也就是mvcc那里



### 1、Buffer Pool

![image-20220701101505723](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220701101505723.png)

###### 	MySQL的数据都是存在磁盘中的，那么要更新一条记录的时候，要从磁盘中读取该记录，然后在内存中修改这个记录。修改完之后，这条记录是直接写回磁盘，还是放到缓存中呢？

###### 	肯定是缓存起来，这样下次查询语句命中这个记录，就可以直接读取了。

###### 	有了Buffer Pool后：

​	1）读取数据时，数据存在Buffer Pool中，就会直接读取，不需要再读磁盘

​	2）修改数据时，直接可以修改Buffer Pool中数据所在的页，然后将页设置为脏页（说明和磁盘上的页数据不一致），为了减少磁盘I/O，后续再将脏页写到磁盘中



##### 	Buffer Pool缓存什么？

​	InnoDB会把存储的数据划分为若干个【页】，以【页】为磁盘和内存交互的基本单位，一个页默认大小16KB。

![image-20220701102008705](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220701102008705.png)

​	例如undo页，事务开启时，如果是更新操作，就会生成一条undo log，会写入Buffer Pool的undo页里

###### 	查询一条记录时，InnoDB会把整个页的数据加载到Buffer Pool中，然后再通过页里面的【页目录】去定位到某条具体的记录



### 2、redo log

###### 在事务提交时，只需要先将redo log持久化到磁盘即可，可以不需要将缓存在Buffer Pool里的脏页数据持久化到硬盘。当系统崩溃时，虽然脏页没有持久化，但是redo log已经持久化了，可以根据redo log的内容将数据恢复到最新状态。



###### redo log和undo log的区别：

1）redo log记录的是此时事务【完成后】的数据状态，记录的是【更新后】的值

2）undo log记录的是此次事务【完成前】的数据状态，记录的是【更新前】的值

###### 事务提交之前发生了崩溃，利用undo log回滚事务；事务提交之后发生崩溃，利用redo log恢复事务

![image-20220701102634680](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220701102634680.png)



### 3、binlog

###### 如果数据库不小心删除了，能用redo log恢复吗？

​	其实是不可以的，因为redo log是循环写，写满之后会覆盖之前的内容

​	而binlog写满之后会新开一个文件继续写

​	binlog文件保存的是全量的日志，也就是保存了所有数据变更的情况。



###### MySQL的主从复制依赖于binlog，也就是记录MySQL上的所有变化并以二进制的形式保存在硬盘上



### 4、事务的两阶段提交

事务提交后，redo log和binlog都要持久化到磁盘，但这是两个独立的逻辑，如果出现半成功的情况，那么会造成两份日志之间的逻辑不一致。



##### 两阶段提交的过程？

为了保证两个日志的一致性，mysql使用了内部XA事务，当客户端执行commit语句或者在自动提交的情况下，MySQL内部开启一个XA事务，分两阶段来完成XA事务的提交。

![image-20220701111737886](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220701111737886.png)

###### 将redo log的写入分成了两阶段，prepare和commit，中间再穿插写入binlog：

1）prepare：将XID（XA事务的ID）写入到redo log，同时将redo log对应的事务设置成prepare，然后将redo log刷新到硬盘

2）commit：将XID写入到binlog，然后将binlog刷新到磁盘，接着将redo log的状态设置成commit



##### 异常重启会出现什么现象？

![image-20220701111940274](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220701111940274.png)

###### 不管是A时刻崩溃还是B时刻崩溃，redo log的状态都是prepare。

###### MySql重启后会扫描redo log文件，碰到处在prepare状态的redo log就拿着XID去binlog里面查找是否存在此XID：

​	1）如果binlog中没有当前事务XA的XID，说明redo log完成刷盘，但binlog没有，就回滚事务；

​	2）如果binlog中有当前事务XA的XID，说明binlog也完成了刷盘，则提交事务。

###### 可见，处于prepare状态的redo log，即可以提交事务也可以回滚事务，这取决于能否在binlog里找到相同的XID。



###### 事务没提交的时候，redo log会持久化到硬盘吗？

​	会的。

​	事务执行时redo log也是直接写在redo log buffer中，这些缓存会后台一起持久化到硬盘。