# Redis（4）Set

## 1、概述

set对外提供的功能与list类似是一个列表的功能，但set是可以自动排重的

是String类型的无序集合，底层是一个value为null的hash表，所以添加删除查找的复杂度都是O(1)



## 2、常用命令

sadd <key> <value1> <value2>...：将一个或多个元素加入到集合key中，已经存在的元素将被忽略

smembers <key>：取出该集合的所有值

sismember <key> <value>：判断key集合是否含有对应的value

scard <key>：返回key集合中元素的个数

srem <key> <value1> <value2>：删除集合中的指定元素

spop <key>：随机从集合中吐出一个值

srandmember <key> <n>：随机取出n个值，但不会删掉

smove <source> <destination> value：把集合中的一个值移动到另一个集合中

sinter<key1> <key2>：返回两个集合中的交集

sunion <key1> <key2>：返回两个集合的并集

sdiff <key1> <key2> ：返回两个集合的差集，key1里的，不包含key2的



## 3、数据结构

set的数据结构是dict字典，字典是用哈希表实现的