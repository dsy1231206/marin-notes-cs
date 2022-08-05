# Spring注解开发-Bean生命周期

###### Bean的生命周期：实例化-->填充属性-->初始化-->销毁

## 一、InitializingBean接口

![image-20220607101141875](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607101141875.png)

​	这个接口为bean提供了属性初始化后的处理方法afterPropertiesSet()，Bean类实现这个接口就能实现属性赋值之后的处理。

###### 	Spring提供的两种初始化方式：

###### 			1、实现InitializingBean接口（实现afterPropertiesSet()方法），直接调用

###### 			2、在配置文件或注解中通过指定init-method（通过反射完成）



## 二、DisposableBean接口

![image-20220607101616953](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607101616953.png)

​	在Bean销毁前，实现这个接口的bean会调用destroy()方法，或者使用配置文件或注解指定的destroy-method，实现生命周期结束前的工作。

###### 	这个接口只针对单例Bean，多实例的Bean生命周期不受Spring管，IOC容器关闭时多实例Bean也不会销毁。



# 三、@PostConstruct和@PreDestroy

### @PostConstruct

![image-20220607102210941](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607102210941.png)

这个注解是Java自己的注解，修饰一个非静态的void()方法，被这个注解修饰的方法会在服务器加载servlet的时候运行，并且只会运行一次。在构造函数之后，init()方法之前执行。



### @PreDestroy注解

![image-20220607102445328](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607102445328.png)

同样也是Java自己的注解，被这个注解修饰的方法会在服务器卸载serlvet的时候运行，并且只会被服务器调用一次。

###### 调用destroy() --> @PreDestroy --> destroy() --> bean销毁



# 四、BeanPostProcessor后置处理器

![image-20220607103251462](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607103251462.png)

这个接口里的两个方法，分别是在bean初始化前后执行的方法。当容器中存在多个BeanPostProcessor实现类时，会按照他们在容器中注册的顺序执行。对于自定义的BeanPostProcessor实现类，还可以让它实现Ordered类接口自定义排序。

###### 自定义一个BeanPostProcessor实现类，添加@Component注解将它添加到容器中，这样就能工作了。

###### Spring在调用initializeBean()方法之前，还调用populateBean()方法来为bean的属性赋值，赋值完成之后，在调用自定义的初始化方法(invokeInitMethods())之前，调用applyBeanPostProcessorsBeforeInitialization()，完成BeanPostProcessor接口的执行。

###### -----> doCreateBean()：Spring获取单例Bean的方法

###### 		--->populateBean()：为Bean属性赋值

###### 		--->initializeBean()：初始化

###### 				--->applyBeanPostProcessorsBeforeInitialization()

###### 				--->invokeInitMethods()：执行自定义初始化方法

###### 				--->applyBeanPostProcessorsAfterInitialization()

# 五、BeanPostProcessor底层

![image-20220607142459546](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607142459546.png)

#### 1、其中一个实现类ApplicationContextAwareProcessor

这个类的作用是向组件中注入IOC容器

![image-20220607142540180](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607142540180.png)

要使用ApplicationContextAwareProcessor向组件中注入IOC容器，可以让组件实现ApplicationContextAware接口。

```java
public class Dog implements ApplicationContextAware {

    private ApplicationContext applicationContext;
    
    public Dog() {
        System.out.println("dog...constructor");
    }
    
    @PostConstruct
    public void init(){
        System.out.println("dog...@PostConstructor");
    }
    
    @PreDestroy
    public void destroy(){
        System.out.println("dog...@PreDestroy");
    }

    /*这个ApplicationContext就是IOC容器对象
        在Dog中定义一个ApplicationContext成员变量，在setApplicationContext中赋值
        这样在Dog类中就可以使用ApplicationContext对象了
         */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

查看ApplicationContextAwareProcessor源码，看到在bean初始化之前，先对bean的类型进行判断，如果实现了ApplicationContextAware接口，那么就会调用invokeAwareInterfaces(bean)，

###### invokeAwareInterfaces()源码：

![image-20220607143556635](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607143556635.png)

由于接口实现了ApplicationContextAware，就会执行第121行代码：((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);

所以在自己定义的类中，通过实现接口的方法setApplicationContext()，就可以直接接收到ApplicationContext对象了。

### 2、BeanValidationPostProcessor类

###### 这个类主要对bean进行校验操作，源码如下：

![image-20220607143924645](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607143924645.png)

都是调用doValidate(bean)对bean进行校验

### 3、InitDestroyAnnotationBeanPostProcessor类

###### 主要用来处理@PostConstructor和@PreDestroy注解，代码中标注了这两个注解，Spring通过InitDestroyAnnotationBeanPostProcessor来知道什么时候执行标注的方法。

![image-20220607144305442](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607144305442.png)

首先拿到关于Bean生命周期的注解，保存在LifecycleMetadata中，之后调用metadata的invokeInitMethod()方法，通过反射来调用标注了注解的方法。

### 4、AutoWiredAnnotationBeanPostProcessor类

