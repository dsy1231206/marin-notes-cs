# IOC容器创建源码（三）BeanPostProcessor

![image-20220615104650812](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615104650812.png)

执行完所有的BeanFactoryPostProcessor之后，就要注册BeanPostProcessor了

![image-20220615104748040](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615104748040.png)

![image-20220615104751313](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615104751313.png)

按照分类的优先级顺序把BeanPostProcessor放到三个list里面去，然后再![image-20220615105611135](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615105611135.png)

把每一个后置处理器添加到beanFactory中

