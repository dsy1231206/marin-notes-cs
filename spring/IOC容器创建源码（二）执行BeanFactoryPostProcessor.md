# IOC容器创建源码（二）执行BeanFactoryPostProcessor

![image-20220615095027192](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615095027192.png)

![image-20220615095044032](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615095044032.png)

###### 这一步是执行BeanFactoryPostProcessor，beanFactory的后置处理器。

![image-20220615095138937](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615095138937.png)

![image-20220615100631368](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615100631368.png)

首先这里会走一个for循环，拿到所有的BeanPostPorcessor，再按是否实现了BeanDefinitionRegistryPostPorcessor分别放在两个list里面，一个是实现了前者的list，一个是常规的BeanPostProcessor的list。

![image-20220615100843126](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615100843126.png)

之后根据实现了PriorityOrdered、Ordered的优先级来执行postProcessBeanDefinitionRegistry()方法

![image-20220615101000740](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615101000740.png)

![image-20220615100953651](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615100953651.png)

###### BeanDefinitionRegistryPostProcessor是BeanFactoryPostProcessor的子接口，所以，接下来还要执行postProcessBeanFactory()方法

![image-20220615101936496](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615101936496.png)

###### 对于BeanDefinitionRegistryPostProcessor组件来说，postProcessBeanDefinitionRegistry()方法会先被调用，postProcessBeanFactory()方法会后被调用。



再执行BeanFactoryPostProcessor的方法

![image-20220615102119053](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615102119053.png)

invokeBeanFactoryPostProcessors方法最主要的核心作用就是执行了BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry和postProcessBeanFactory这俩方法，以及BeanFactoryPostProcessors的postProcessBeanFactory方法。
