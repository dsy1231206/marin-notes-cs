# IOC容器创建源码（四）初始化MessageSource组件

![image-20220615111230248](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615111230248.png)

注册完所有的BeanPostProcessor之后，就要初始化MessageSource

![image-20220615111353438](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615111353438.png)

###### 1、先获取beanFactory，在之前已经准备好了

###### 2、然后判断容器中是否有一个叫MESSAGE_SOURCE_BEAN_NAME的组件，即messageSource，如果有就赋值给this.messageSource，很明显刚创建容器之后是没有这个组件的

###### 3、来到else语句：

​	Spring就会自己创建一个DelegatingMessageSource。这东西有什么用？

![image-20220615111614949](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615111614949.png)

它可以从一个配置文件里取出某一个key对应的值，特别是国际化配置文件，还能按区域信息获取

![image-20220615111725148](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615111725148.png)

最后，就把这个组件注册进BeanFactory里