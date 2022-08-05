# MySQL（2）-MVCC

###### 很重要的一个点：

​		可重复读，是在每次开启事务的时候创建一个Read View

​		而读已提交，是在事务每次读取数据行的时候新建Read View，所以会出现两次读数据不一致的情况



###### 实际上，除了select是快照读，其他update delete 这样的都是当前读

​		当然，select ....  for update这样的语句，就可以变成当前读



###### 在可重复读的情况下，使用select... for update语句，就可能会出现两次查询结果不一致的情况，所以，InnoDB为了解决【可重复读】情况下使用当前读造成幻读问题，就出现了next-key锁，就是记录锁和间隙锁本身。



###### 记录锁，锁的是记录本身

###### 间隙锁，锁的是两个值之间的空隙

![image-20220630143827347](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220630143827347.png)