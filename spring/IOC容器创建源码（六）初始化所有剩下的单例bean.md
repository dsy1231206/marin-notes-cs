# IOC容器创建源码（六）初始化所有剩下的单例bean

![image-20220615142027006](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615142027006.png)

![image-20220615142055750](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615142055750.png)

![image-20220615142148187](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615142148187.png)

finishBeanFactoryInitialization()方法的最后一行，才是初始化所有剩下的单例bean

![image-20220615142222012](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615142222012.png)

首先要遍历每一个bean的注册信息，bean的定义注册信息是用RootBeanDefinition来封装的。

会根据信息判断bean是否为抽象的，单例的，懒加载的

调用getBean()获取对象，getBean()里面又调用了doGetBean()：

![image-20220615142610938](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615142610938.png)

通过bean的名字来获取缓存中保存的单例bean：

![image-20220615142654573](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615142654573.png)

singletonObjects是DefaultSingletonBeanRegistry类里面的一个属性，是一个map集合

![image-20220615142747301](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615142747301.png)

###### 如果获取不到单例bean，那么就要创建bean。单实例bean的创建流程：

![image-20220615150817737](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615150817737.png)

使用BeanWrapper接口来接收我们创建的bean。

![image-20220615150929800](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615150929800.png)

创建完之后调用populateBean()方法进行属性赋值

总结一下：

​	1）在创建bean实例之前，会执行InstantiationAwareBeanPostProcessor这种类型的后置处理器中的两个方法，即postProcessBeforeInstantiation()和postProcessAfterInitialization()

​	2）创建bean实例

​	3）为bean属性赋值。在赋值过程中，会执行InstantiationAwareBeanPostProcessor这种类型的后置处理器中的两个方法，即postProcessAfterInstantiation和postProcessPropertyValues方法

​	4）初始化bean

![image-20220615152536089](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615152536089.png)

invokeAwareMethods()方法，是用来执行XxxAware接口中的方法的

![image-20220615152637778](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615152637778.png)

![image-20220615152702027](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615152702027.png)

执行后置处理器初始化前的方法(postProcessBeforeInitialization)

![image-20220615152800915](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615152800915.png)

之后就执行初始化方法，例如自定义的bean里面，可以指定init初始化方法，或者实现InitializingBean接口，实现其afterPropertiesSet()方法。

![image-20220615153048355](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615153048355.png)

![image-20220615153103868](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615153103868.png)

执行完初始化方法后，就来执行postProcessAfterInitialization方法

![image-20220615153310042](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615153310042.png)

最后就要注册bean的销毁方法（不是调用！）

![image-20220615153420854](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615153420854.png)

bean实例化完成之后，就要把单例bean加入到缓存中

![image-20220615153501153](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615153501153.png)

![image-20220615153513872](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615153513872.png)

所谓的IOC容器就是各种各样的map集合，这些map集合里保存了创建的所有组件和IOC容器的其他信息