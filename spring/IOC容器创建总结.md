# IOC容器创建总结

### 1、Spring IOC容器在启动的时候，会先保存所有注册进来的bean的信息，将来BeanFactory就会按照这些bean的定义信息为我们创建对象

###### 如何来编写Bean的定义信息呢？

​	1）通过XML配置文件来注册bean

​	2）通过@Service、@Component、@Bean等注解来注册信息

### 2、当IOC容器中有保存一些bean的定义信息的时候，会在合适的时机创建这些bean

###### 	1）在用到某个bean的时候，在统一创建所有剩下的单例bean之前，比如后置处理器这些组件都会调用getBean()方法创建出来，创建好之后保存在容器里

###### 	2）统一创建所有剩下的单例bean时

### 3、每一个bean创建完成之后，都会有各种各样的后置处理器来进行处理，以此来增强bean的功能。

###### 使用@Autowired注解可以完成自动注入，因为Spring中有一个专门处理这个注解的后置处理器AutowiredAnnotationBeanProcessor

### 4、Spring的事件驱动模型

###### ApplicationListener：事件监听

###### ApplicationEventMulticaster：事件派发器