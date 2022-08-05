# IOC容器创建源码（五）初始化事件派发器

![image-20220615112255186](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615112255186.png)

初始化MessageSource之后，就要初始化事件派发器了ApplicationEventMulticaster()

![image-20220615112333666](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615112333666.png)

###### 先是查看容器中是否有id为applicationEventMulticaster的组件，如果有，就赋值给this.applicationEventmulticaster。很明显这里是没有的，走下面的else语句

###### new一个SimpleApplicationEventMulticaster，再把它注册到容器中

![image-20220615112522963](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615112522963.png)

###### 执行完成后，来到onRefresh()

![image-20220615112540188](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615112540188.png)

###### 里面也是空的，给子类使用

![image-20220615112556632](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615112556632.png)

![image-20220615112608275](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615112608275.png)

###### 先是获取所有的监听器，然后添加到事件派发器中。

###### 然后拿到所有的ApplicationListener组件，