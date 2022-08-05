# Redis（1）概述

## 1、概述

###### 更有扩展性的数据库，直接采用key-value形式存储

1）开源的key-value存储系统

2）支持的value类型很多，包括string、list、set、zset（sorted set有序集合）、hash

3）对数据的操作是原子性的

4）数据都是缓存在内存中，可以持久化到硬盘

5）redis会周期性地将数据写入磁盘

6）实现了master-slave主从同步



## 2、Redis相关知识

###### 1、默认16个数据库，下标从0开始，默认使用0号库

使用命令来切换数据库，例如select 8

```
select <dbid>
```

dbsize：查看当前数据库key的数量

flushdb：清空当前库

flushall：清空所有库



###### 2、redis是单线程+多路IO复用

