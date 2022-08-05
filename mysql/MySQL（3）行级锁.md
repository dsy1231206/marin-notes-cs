# MySQL（3）锁

#### 行级锁

###### 对记录加锁时，加锁的基本单位是next-key lock，它是由记录锁和间隙锁组合而成的，next-key lock是前开后闭区间，而间隙锁是前开后开区间





##### 案例

![image-20220630144210036](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220630144210036.png)

以这张表为例，id是主键索引，b是普通索引，a是普通列

当使用唯一索引进行等值查询的时候，查询的记录存不存在，加锁的规则也不同：

###### 	当查询的记录存在，用唯一索引进行等值查询时，next-key lock会退化成【记录锁】

###### 	当查询的记录不存在，next-key lock会退化成间隙锁



而使用非唯一索引等值查询的时候，加锁的规则是：

###### 	当查询的记录存在时，除了会加next-key lock外，还会加间隙锁

###### 	当查询的记录不存在，加next-key lock，然后会退化成间隙锁



### 1、全局锁

使用全局锁后，整个数据库都变成只读的，通常用在备份数据库的时候。

这个时候其他命令比如insert update delete都会被阻塞



有没有其他方式可以避免并发度太低呢？

​	使用MVCC，在备份数据库事务开始时，创建一个Read View，这样备份的时候就使用这个Read View



### 2、表级锁

MySQL里面的表级锁有这几种：表锁、元数据锁、意向锁、AUTO-INC锁



### 3、行级锁

###### 主要是行级锁，行级锁有三类：

###### 	1）Record Lock，记录锁，仅仅把一条记录锁上

###### 	2）Gap Lock，间隙锁，锁定一个范围，但不包含记录本身

###### 	3）Next-Key Lock，记录锁和间隙锁的组合，锁定一个范围加记录本身



#### 行级锁是怎么加锁的？

对记录加锁时，加锁的基本单位是next-key lock，next-key lock是前开后闭区间，间隙锁是前开后开区间

![image-20220630152129985](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220630152129985.png)

##### 唯一索引等值查询

###### 	1）查询的记录存在，next-key lock会退化成记录锁

###### 	2）查询的记录不存在，会退化成间隙锁

![image-20220630151940978](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220630151940978.png)

##### 

##### 唯一索引范围查询

![image-20220630152424026](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220630152424026.png)



##### 非唯一索引等值查询

###### 1）当查询的记录存在，除了会加next-key lock之外，还会加间隙锁

###### 2）当查询的记录不存在，只会加next-key lock，然后会退化成间隙锁



##### 但普通索引的范围查询，next-key lock不会退化为间隙锁和记录锁





###### 因此，例如在使用update语句时，如果where条件没有使用索引，就会全表扫描，于是会对所有记录都加上next-key lock锁，相当于把整个表都锁住了，接下来的操作都会被阻塞直到事务结束。