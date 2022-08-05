# MySQL（1）基础架构

### 1、基础结构

![image-20220629101901637](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220629101901637.png)

​	Mysql的基础结构如图所示，分为Server层和存储引擎层。Server层负责连接、SQL语句解析、优化、执行SQL语句等功能，而存储引擎层就负责存储数据，比如InnoDB、MyISAM这样的存储引擎。

​	值得注意的是，在MYSQL8.0版本以后，查询缓存这个功能就删除了，因为实用性不高。如果有更新数据库的语句出现的话，就会把缓存清空。

### 2、binlog和redo log

​	首先redo log，redo log是InnoDB引擎所特有的一种日志，用来记录操作记录。

​	比如新来一条语句update...，redo log就会把这记录写进日志。在某些特定的时间将记录更新到磁盘里。

​	![image-20220629102431990](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220629102431990.png)

​	redo log大小是固定的，write pos是当前写记录的指针，checkpoint是当前更新到磁盘里的指针。

​	他们都是循环使用的，当write pos追上checkpoint，就需要同步到磁盘了

​	有了redo log，InnoDB就有了crash-safe的能力



​	

​	其次是binlog，binlog是server层持有的日志，所有引擎都可以使用。binlog是没有crash-safe的能力的，它只用来归档。

###### 	binlog和redo log有以下不同：

​		1、redo log是InnoDB持有的，binlog是server层持有的，所有引擎都能用

​		2、redo log是物理日志，记录的是“在某个数据页做了什么修改”，binlog是逻辑日志，记录的是这个语句的原始逻辑

​		3、redo log是循环写的，空间固定会用完，而binlog可以追加写，不会覆盖以前的日志

![image-20220629103211631](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220629103211631.png)

###### 			浅色框是InnoDB执行的，深色的是执行器执行的。

###### 			这里需要用到两阶段提交，写入redo log进入prepare状态，执行器把操作写入binlog之后，再把redo log改为commit状态。这样才能保证两个日志的逻辑一致。