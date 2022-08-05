# Spring拓展-ApplicationListener

## 1、概述

![image-20220614140400349](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614140400349.png)

###### 如果要写一个监听器，那么就要实现这个借口，而接口中的泛型就是要监听的事件，即ApplicationEvent及其子事件。

## 2、ApplicationListener的作用

###### 监听IOC容器中发布的一些事件，只要事件发生就会触发监听器的回调。

## 3、ApplicationListener的用法

自定义一个监听器，实现ApplicationListener接口，并注册到容器中

```java
//要加入到IOC容器中
@Component
public class MyApplicationListener implements ApplicationListener<ApplicationEvent> {
    //当容器中发布事件，这个方法就会被触发
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("收到事件:" + event);
    }
}
```

那么，比如启动IOC容器时，就会触发这个事件：

例如：收到事件:org.springframework.context.event.ContextRefreshedEvent

说明容器刷新完毕

同时一旦容器关闭，就会发布ContextClosedEvent事件

###### 那么可以自己发布事件吗？

###### 	1）写一个监听器来监听某个事件，当然监听的事件一定是ApplicationEvent及其子类

###### 	2）把监听器加入到容器中

###### 	3）只要容器中有相关事件发布，就能监听到这个事件

###### 	4）自己发布一个事件：

![image-20220614141338613](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614141338613.png)

## 4、ApplicationListener的原理

### 	1）完成容器刷新事件：

​		容器创建时会调用refresh()方法刷新容器，在这个方法的最后会调用一个finishRefresh()方法，而在这个方法里面又会调用一个publishEvent()

![image-20220614141635454](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614141635454.png)

![image-20220614141639164](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614141639164.png)

![image-20220614142326142](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614142326142.png)

![image-20220614142332505](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614142332505.png)

最终底层的实现逻辑就是：拿到所有的ApplicationListener，遍历每一个ApplicationListener；

有一个if判断，会判断getTaskExecutor方法能不能返回一个Executor对象，如果能的话，就用Executor的异步执行功能使用多线程的方式异步派发事件，如果不能获取，就用同步方式直接执行ApplicationListener的方法

invokeListener():

![image-20220614142556554](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614142556554.png)

![image-20220614142605748](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614142605748.png)

最终就是回调ApplicationListener的onApplicationEvent()方法，就是我们自己实现的那个方法

### 2）关于事件多播器

###### 怎么拿到的？同样还是创建IOC容器的refresh()方法中

![image-20220614143956611](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614143956611.png)

![image-20220614144012205](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614144012205.png)

###### 先会判断一下容器中是否有applicationEventMulticaster的组件，如果没有，就new一个SimpleApplicationEventMulticaster注册到容器中。

怎么将监听器注册到容器？

![image-20220614144157221](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614144157221.png)

![image-20220614144201666](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614144201666.png)

## 5、@EventListner注解

![image-20220614150246373](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614150246373.png)

使用方法：自己定义一个监听方法，在方法上加上@EventListner注解

```java
@EventListener(classes = ApplicationEvent.class)
    public void listen(ApplicationEvent event){
        System.out.println("UserService...listen..." + event);
    }
```

##### 原理：

参考EventListenerMethodProcessor这个类，这个类是一个处理器，用来解析方法上的@EventListener注解的



![image-20220614151735476](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614151735476.png)

![image-20220614151751584](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614151751584.png)

这个类会实现SmartInitializingSingleton接口，有一个afterSingletonsInsstantiated()方法

这个方法什么时候执行呢？

![image-20220614151556616](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614151556616.png)

###### 上面这个方法会实例化所有还没有实例化的bean

![image-20220614151604835](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614151604835.png)

![image-20220614151610627](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220614151610627.png)

###### 在所有单例bean实例化之后调用afterSingletonsInstantiated()方法