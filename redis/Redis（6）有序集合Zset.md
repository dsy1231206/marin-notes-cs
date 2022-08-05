# Redis（6）有序集合Zset

## 1、概述

zset和普通集合set非常相似，是一个没有重复元素的字符串集合

不同之处是每个成员都关联了一个评分（score），评分被用来按照最低分到最高分来对元素进行排序



## 2、常用命令

zadd <key> <score1> <value1> <score2> <value2> ...将一个或多个元素按评分放入zset

zrange <key> <start> <stop> [WITHSCORES] 按下标取出元素

zrangebyscore key min max [WITHSCORES] [LIMIT OFFSET COUNT]:根据score的范围取出元素

zrevrangebyscore key min max：逆序取出元素



zincrby <key> <increment> <value>：为元素的score加上增量

zrem <key> <value>：删除指定值的元素

zcount <key> <min> <max>：统计指定范围的元素个数

zrank <key> <value>：返回集合中value元素的排名



## 3、数据结构

SortedSet(zset)是redis提供的一种数据结构，一方面等价于Map<String,Double>，可以对每一个元素赋值score，另一方面又类似于TreeSet

1）hash，hash的作用就是关联元素的value和权重score，保障元素的唯一性

2）跳表，跳表是为了给value排序，根据score的范围获取元素列表