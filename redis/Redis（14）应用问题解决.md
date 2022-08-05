# Redis（14）应用问题解决



## 1、缓存穿透

应用服务器压力变大了，先查缓存，但缓存中大部分数据都不存在（Redis命中率降低），所有数据都去查数据库，造成数据库压力增加而崩溃。

##### 原因

1）redis查询不到数据

2）出现很多非正常url访问

###### 很多缓存穿透情况都是受到服务器攻击产生的。



##### 解决办法

###### 1）对空值缓存

​		如果一个查询返回的数据为空（数据库中不存在这个数据），仍然把这个查询结果进行缓存，设置空结果的过期时间很短。

###### 2）设置可访问的名单（白名单）

​		使用bitmaps类型定义一个可以访问的名单，名单id作为bitmaps的偏移量

###### 3）采用布隆过滤器

###### 4）进行实时监控

​		当redis命中率开始急剧降低，需要排查访问对象和访问的数据



## 2、缓存击穿

数据库访问压力瞬时增加，redis里面并没有出现大量的key过期，redis还是正常运行状态。



##### 原因

redis某个key过期了，而这个key正好有大量的访问



##### 解决方案

###### 1）预先设置热门数据

​		在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大key的时长

###### 2）实施调整key时长

###### 3）使用锁

​		查询某个数据不存在的适合，先上锁，到数据库中去同步缓存，再释放锁



## 3、缓存雪崩

大量redis中的key过期，高并发下数据库压力变大，造成服务器崩溃



##### 解决方案

###### 1）构建多级缓存架构

​		nginx+redis+其他缓存等

###### 2）使用锁或队列

​		用加锁或者队列的方式来保证不会有大量的线程对数据库一次性读写，但高并发性能低

###### 3）设置过期标志更新缓存

​		记录缓存数据是否过期(设置提前量)，如果过期会触发通知另外的线程在后台去更新key

###### 4）将缓存失效时间分散开

​		避免key集体过期



## 4、分布式锁

分布式锁对集群都有效



##### 分布式锁的主流实现方案：

1.基于数据库实现分布式锁

2.基于缓存（Redis），性能最高

3.基于Zookeeper，可靠性最高



##### 基于缓存的方式

1、使用setnx命令上锁，通过del释放锁

2、锁一直没有释放，需要设置过期时间

3、上锁后突然出现异常（崩溃），无法设置过期时间

​		上锁的时候同时设置过期时间

```
set users 10 nx ex 12
```



#####  Java代码测试

```java
@RestController
public class RedisController {

    @Autowired
    private  RedisTemplate redisTemplate;

    @GetMapping("testLock")
    public void testLock(){
        //1、获取锁
        Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock","111",3, TimeUnit.SECONDS);
        //2、获取锁成功，查询num的值
        if(lock){
            Object value = redisTemplate.opsForValue.get("nums");
            if(StringUtils.isEmpty(value)){
                return;
            }
            //2有值就转换成int
            int num = Integer.parseInt(value+"");
            //3.把redis的nums+1
            redisTemplate.opsForValue.set("num",++num);
            //4.释放锁
            redisTemplate.delete("lock");
        }else{
            //获取锁失败，间隔时间重试
            try{
                Thread.sleep(100);
                testLock();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



##### 存在的问题1

![image-20220708111259799](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220708111259799.png)



##### 优化1

###### 1、使用uuid

uuid表示不同的操作

```
set lock <uuid> nx ex 10
```

释放锁的时候，判断当前的uuid是否和上锁的uuid一样

这就只能删除自己的锁了



##### 存在的问题2

删除操作缺乏原子性

a比较uuid之后，锁过期了，这时候b获得锁了，等a删除锁的时候可能会删除b的锁

![image-20220708112534864](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220708112534864.png)



##### 优化2

使用LUA脚本



##### 成为分布式锁的条件：

1、互斥性。任何时刻只有一个客户端能持有锁

2、不会发生死锁。即使有一个客户端在持有锁期间发生崩溃而没有主动解锁，锁也会自动过期删除，保证后续客户端能加锁

3、加锁和解锁必须是同一个客户端

4、加锁和解锁必须具有原子性