# Redis（13）集群

容量不够，redis如何扩容？

并发情况下怎么进行写操作？

另外，主从模式、薪火相传模式、主机宕机、导致ip地址发生变化，应用程序都需要修改对应的主机地址，端口等信息



## 1、什么是集群？

##### 无中心化集群：任何一台服务器都可以作为入口

Redis集群实现了对Redis的水平扩容，启动N个redis节点，将整个数据库分布存储在N个节点中，每个节点存储总数据的1/N。



##### 案例

搭建3主3从 无中心化集群

配置六个配置文件 6379 6380 6381 6389 6390 6391



##### redis cluster配置修改

```
cluster-enabled yes #打开集群模式
cluster-config-file nodes-6379.conf #设定节点配置文件名
cluster-node-timeout 15000 #设置节点失效时间，超过该时间（毫秒），集群自动进行主从切换
```



##### 启动6个redis服务

```
redis-server redis6379.conf
```



##### 将6个节点变成一个集群

确保服务都启动，node配置文件都正常生成

进入redis安装目录的src文件夹下：

```
redis-cli --cluster create --cluster-replicas 1 [ip:port...]
```

--cluster-replicas 1：采用最简单的方式配置集群，最后加上ip地址+端口号

这里ip地址需要写实际ip，写ip:port的顺序是一主一从

###### 分配原则：尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在同一个IP地址



##### 连接redis服务，集群方式

```
redis-cli -c -p 6379
```

用任何一个节点（端口）都是可以的



##### 查看集群节点信息

```
cluster nodes
```



##### 什么是slots

集群搭建成功后会有这样一句语句：

```
[OK] All 16384 slots covered.
```

一个Redis集群包含16384个插槽（hash slot），数据库中每一个键都属于这么多插槽中的一个

集群使用公式CRC16(key) % 16384 来计算key属于哪个槽

每个节点负责一部分插槽

通过key计算插槽的值：

```
cluster keyslot <key>
```

计算插槽里有几个键：

```
cluster countkeysinslot <插槽值> #只能看自己节点负责的插槽
```



## 2、故障恢复

##### 主机挂掉

这个主机的从机会变成主机，之前的主机重启之后会变成它的从机（15秒）



##### 主机从机都挂掉

1）cluster-require-full-coverage为yes，那么整个集群都挂

2）cluster-require-full-coverage为no，那么该段插槽的数据全部不能使用也不能存储



## 3、Jedis操作集群

```java
public class Cluster {
    public static void main(String[] args) {
        //redis服务要开启
        //创建对象
        HostAndPort hostAndPort = new HostAndPort("192.168.44.168", 6379);
        JedisCluster jedisCluster = new JedisCluster(hostAndPort);
        
        //进行操作
        jedisCluster.set("k1","v1");

        String value = jedisCluster.get("k1");
        System.out.println(value);
        jedisCluster.close();
    }
}
```



## 4、集群优缺点

##### 优点

实现扩容

分摊压力

无中心配置相对简单



##### 缺点

多键操作不被支持

多键的redis事务不支持

