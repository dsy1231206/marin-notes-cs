# Redis（5）哈希表

## 1、概述

hash是一个键值对集合，类似于Map<String,Object>结构

![image-20220704164042259](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220704164042259.png)

## 2、常用命令

hset <key><field><value>：给key的hash表里面添加field-value对

 hget <key> <field>

hmset<key> <field1><value1> <field2> <value2>

hexists <key> <field>

hkeys <key> 查看hash集合中所有的field

hvals <key> 查看所有的value

hincrby <key> <field> <increment>：给某个field的值加个增量

hsetnx <key> <field> <value>



## 3、数据结构

Hash类型对应的数据结构有两种，ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。