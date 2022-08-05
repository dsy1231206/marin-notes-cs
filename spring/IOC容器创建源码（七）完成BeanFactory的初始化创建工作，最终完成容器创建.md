# IOC容器创建源码（七）完成BeanFactory的初始化创建工作，最终完成容器创建

###### 在for循环调用getBean()创建完所有的单例bean后，就会继续下面的方法：

![image-20220615155006583](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615155006583.png)

###### 判断是否有bean实现了SmartInitializingSingleton接口，如果有的话还要执行afterSingletonInstantiated()方法。

###### 初始化时就有一个EventListenerMethodProcessor实现了这个接口

![image-20220615155232084](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615155232084.png)

###### 最后就是完成BeanFactory的初始化创建工作。

![image-20220615155301573](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615155301573.png)

###### 	1）初始化生命周期相关的后置处理器

![image-20220615155320680](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615155320680.png)

###### 	2）回调生命周期处理器的onRefresh()方法

![image-20220615155502593](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615155502593.png)

###### 	3）发布容器刷新完成事件

![image-20220615155507161](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615155507161.png)