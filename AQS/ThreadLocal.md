# ThreadLocal

### 1、原理

###### 根据Thread类里面的源码：

```java
public class Thread implements Runnable {
    //......
    //与此线程有关的ThreadLocal值。由ThreadLocal类维护
    ThreadLocal.ThreadLocalMap threadLocals = null;

    //与此线程有关的InheritableThreadLocal值。由InheritableThreadLocal类维护
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    //......
}
```

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

每个Thread对象里都有一个ThreadLocalMap，最终的变量就是存在这个Map上的，ThreadLocal只是作为一个封装。

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    //......
}
```

比如在一个线程中声明了多个ThreadLocal，Thread内部都是使用仅有的那个ThreadLocalMap来存放数据，key就是那个ThreadLocal对象，value是使用ThreadLocal对象调用set()设置的值

![image-20220718110010209](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220718110010209.png)

###### ThreadLocalMap是ThreadLocal内部的静态类



## 2、扩展

###### 1）ThreadLocal的key如果是被弱引用持有，在GC之后这个key会被回收，key就会变成null

##### 四种引用类型：

强引用：经常new出来的对象就是强引用，GC永远不会回收强引用

软引用：使用SofrReference修饰的对象，在内存溢出时回收

弱引用：使用WeakReference修饰的对象，只要发生垃圾回收，弱引用就会被回收

虚引用：最弱的引用，唯一的作用就是用队列接收对象即将死亡的通知



![image-20220719093905391](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220719093905391.png)

###### 这样就是弱引用持有的ThreadLocal，GC之后会被回收



![image-20220719094709155](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220719094709155.png)

###### 这样就是被强引用持有，GC之后不会回收key



## 3、ThreadLocal为什么要设计成弱引用？

ThreadLocal用作ThreadLocalMap的key时，是被设计成弱引用的

###### （注意，WeakReference本身是一个强引用，它内部的（T reference）才是真正的弱引用字段，WeakReference是一个装弱引用的容器而已）

ThreadLocalMap是用来存放对象的，在一次线程的执行栈中，存放数据后方便在任意地方取得想要的值。

ThreadLocalMap本身没有为外界提供取出和存放数据的API，我们只能通过ThreadLocal类提供的API来间接从ThreadLocalMap取出数据。所以，当用不了key的时候，也就无法从map中取得数据了。

不能取出数据，Entry就没有用了。但是让Entry或value作为弱引用都是不合理的，所以，让key(ThreadLocal)作为弱引用，自动回收，key就变成null了。下次就可以判断Entry不为null而key为null判断Entry对象被清理掉了。

![image-20220719095557624](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220719095557624.png)



## 4、ThreadLocalMap.set()

![image-20220719135731456](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220719135731456.png)

###### 在HashMap中，如果遇到hash冲突的话，是将value挂在链表上，如果链表长度超过一定值会转为红黑树。

冲突：计算出来的位置i相同，并且计算key的hashCode也相同

###### 而在ThreadLocalMap中并没有HashMap的解决方案，而是依次向后线性查找，直到找到一个Entry为null的位置。

