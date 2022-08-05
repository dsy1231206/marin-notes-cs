# Redis（2）String

## 1、命令

keys *：查看当前库的所有key

exists key：判断某个key是否存在

type key：查看key是什么类型

del key：删除key

unlink key：根据key选择非阻塞删除（异步删除）

expire key 10：设置key的过期时间，以秒为单位

ttl key：查看还有多少秒过期  -1表示永不过期  -2表示已过期



## 2、String

###### String是二进制安全的，意味着可以存任何数据，比如Jpg图片或者序列化的对象

###### 一个字符串的value最多可以是512M

常用命令：

set <key> <value>：存入key-value

get <key> ：查询对应键值

append <key> <value>：将给定的<value>追加到原值末尾

strlen <key> ：获得值的长度

setnx <key> <value>：只有当key不存在时，设置key的值

incr <key>：给key对应的数字+1，只能给数字字符串使用（原子操作）

decr <key>：给值减1

incrby/decrby <key> <步长>：加减特定的值

###### 原子操作：不会被线程调度机制打断的操作，因为redis是单线程的



同时对多个键值对操作：

mset <key1><value1><key2><value2>...

mget<key1><key2>...

msetnx<key1><value1><key2><value2>...

###### 原子性：有一个失败则都失败



getrange <key> <开始位置> <结束位置>：获得值的范围，substring，前闭后闭

setrange <key> <开始位置> <value>：用value覆写指定位置的字符串的值



setex <key> <过期时间> <value>：设置有过期时间的键值对

getset <key> <value> ：写新值的同时获取旧值



## 3、数据结构

![image-20220704141020294](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220704141020294.png)

###### String的数据结构是简单的动态字符串，为当前字符分配的空间capacity一般高于实际字符串的长度length，当字符串大小小于1M时，扩容都是加倍现有空间，如果超过1M，扩容时一次多扩1M。最大长度不能超过512M

