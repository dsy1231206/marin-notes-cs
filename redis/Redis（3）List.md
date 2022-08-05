# Redis（3）List

## 1、简介

 Redis列表是简单的字符串列表，按照插入顺序排序，可以添加数据到头部或尾部

底层实际上就是一个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

![image-20220704141410411](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220704141410411.png)



## 2、常用命令

lpush/rpush <key> <value1> <value2> <value3>：从左端或右端插入值（头插法）

lpop/rpop <key> 从左端/右端取出一个值，值取光之后key就不存在了

rpoplpush <key1> <key2>：从key1的右端取出一个值放到key2左端

lrange <key> <start> <stop>：根据索引下标取值 (0 -1 代表取所有值)

lindex <key> <index> ：根据指定下标取元素

llen <key>：获取列表长度



## 3、数据结构

list的数据结构为快速链表 quickList

###### 列表元素较少的时候使用连续的数据结构存储，叫zipList

###### 当数据量比较多的时候，使用多个zipList来存储，变成quickList

###### 普通链表需要的附加指针空间太大，比较浪费空间，quickList就会比较省空间。