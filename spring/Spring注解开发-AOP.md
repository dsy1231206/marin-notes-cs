# Spring注解开发-AOP

# 1、使用AOP

###### AOP是在程序的运行期间动态地将某段代码切入到指定方法，指定位置进行的编程方式。底层是用动态代理实现的。

##### 1）导入AOP依赖

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.12.RELEASE</version>
</dependency>
```

##### 2）定义目标类

###### 即想要切面作用在哪个类或者哪个类的方法上，几乎可以是任何类

##### 3）定义切面类

###### 在切面类中定义想要切面执行的方法，动态地感知目标类方法的运行情况。

###### 给切面类加上注解@Aspect

AOP中的通知方法：

前置通知@Before：目标方法运行前执行

后置通知@After：目标方法运行后执行

返回通知@AfterReturning：目标方法返回之后执行

异常通知@AfterThrowing：目标方法运行出现异常之后运行

环绕通知@Around：动态代理，手动推进目标方法运行

##### 4）在切面类中定义切入点

```java
@Pointcut("execution(public int com.dsy.spring.aop.MathCalculator.*(..))")
    public void pointCut(){}
```

###### PointCut就是定义出来的公共切入点表达式，方法名随便写，但是方法内什么都不能写

##### 5）使用切入点

​		1、本类中使用

​		在方法上添加AOP注解，例如：

```java
@Before("pointCut()")
```

​		2、在其他类中使用

​		就要在通知注解中写方法的全限定名了，例如：

```
@After("com.dsy.spring.aop.LogAspects.pointCut()")
```

##### 6）把切面类和目标类加入到IOC容器

###### 可以使用XML配置文件或配置类将切面类和目标类注册到容器中

使用配置类，要加上@EnableAspectJAutoProxy，开启基于注解的AOP模式

##### 7）如果想在切面类中获取到目标方法的参数？

方法参数中加上JoinPoint，JoinPoint一定在参数列表的第一位，否则Spring无法识别

JoinPoint对象里封装了目标方法的信息

```java
@Before("pointCut()")
    public void logStart(JoinPoint joinPoint){

        //System.out.println("方法运行...@Before");

        Object[] args = joinPoint.getArgs(
        System.out.println(joinPoint.getSignature().getName() + " ...@Before...{" + Arrays.asList(args) + "}");
    }
```

# 2、@EnableAspectJAutoProxy

###### 在配置类上添加@EnableAspectJAutoProxy注解，就可以开启注解版的AOP功能

###### 只能加在类上，引入了AspectJAutoProxyRegistrar组件，这个组件又实现了ImportBeanDefinitionRegistrar，说明向容器中注册了一个组件。

![image-20220609133217153](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220609133217153.png)

###### 即：添加@EnableAspectJAutoProxy后，会向容器中注册AnnotationAwareAspectJAutoProxyCreator，注解装配模式的AspectJ切面自动代理创建器。

![image-20220609135047465](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220609135047465.png)

它既是一个BeanPostProcessor，又是一个Aware。这两个接口都与Bean的初始化有关。

通过图上的继承关系，得知这个AnnotationAwareAspectJAutoProxyCreator实现了两个接口：

1）BeanPostProcessor：后置处理器，在bean初始化前后做事情

2）BeanFactoryAware：自动注入BeanFactory

###### 注册AnnotationAwareAspectJAutoProxyCreator：它本身是一个BeanPostProcessor，注册就是拿到所有的BeanPostProcessor，然后调用beanFactory的addBeanPostProcessor()方法将BeanPostProcessor注册到BeanFactory中。

### 过程：

##### 1、添加@EnableAspectJAutoProxy，会向容器中注册一个AnnotationAwareAspectJAutoProxyCreator，它既是一个BeanPostProcessor，又实现了BeanFactoryAware注入了beanFactory。

##### 2、在创建容器的refresh()方法中有一行代码：

```java
registerBeanPostProcessors(beanFactory);
```

即注册bean的后置处理器，最终定位到这里：![image-20220609150531982](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220609150531982.png)

即先拿到BeanPostProcessor类型的后置处理器，然后按照是否实现了PriorityOrdered、Ordered进行分类，决定了他们的工作顺序。



###### AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前，会有一个拦截，因为它是InstantiationAwareBeanPostProcessor这种类型的后置处理器，然后会调用它的postProcessBeforeInstantiation()方法

# 3、AnnotationAwareAspectJAutoProxyCreator有什么作用？

###### 在每一个bean创建之前，调用postProcessBeforeInstantiation()方法

##### 1）先来判断bean是否在advisedBeans这个Set集合中

##### 2）再来判断当前bean是否为基础类型，或者是否为切面（标注了@Aspect注解）

![image-20220610100235464](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610100235464.png)

###### 基础类型？基础类型就是实现了Advice、PointCut、Advisor以及AopInfrastructureBean这些接口。

![image-20220610100341070](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610100341070.png)

##### 3）最后判断是否需要跳过

![image-20220610100631667](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610100631667.png)

先拿到增强器的集合，findCandidateAdvisors()。

###### 什么是增强器？标注了@Before、@After这些注解的通知方法。比如程序里自己写的第一个增强器就是logStart()。

这里是获取到这些通知方法，包装成一个Advisor，判断增强器是否是AspectJPointCutAdvisor类型的。

##### 4）创建完对象之后，调用postProcessAfterInitialization()方法

判断一下bean是否在earlyProxyReferences集合里，再调用wrapIfNecessary()。

什么情况是需要包装的？

![image-20220610101224763](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610101224763.png)

看到下面那一行代码：Object[] specificInterceptors = getAdcicesAndAdvisorsForBean(bean.getClass(),beanName,null);

###### 这一行代码就得到了目标类可用的增强器，这些增强器是要拦截目标方法执行的

##### 5）由于上面得到了目标bean可用的增强器，那么就会走if下面的代码，将bean放入advisedBean这个Set集合中

![image-20220610102429022](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610102429022.png)

##### 6）如果这个bean需要增强，则创建当前bean的代理对象

![image-20220610102441973](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610102441973.png)

调用代理工厂类来创建一个代理对象：

![image-20220610103122273](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610103122273.png)

![image-20220610103126893](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610103126893.png)

![image-20220610103133812](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610103133812.png)

###### 如果当前类是有实现接口的，那么就由jdk来创建动态代理，如果当前类没有实现接口，那么就使用cglib来创建动态代理。

所以，wrapIfNecessary()方法调用完之后，最终会给容器中返回当前组件使用cglib增强了的代理对象。

# 4、目标方法的拦截逻辑

##### 根据ProxyFactory对象获取将要执行的目标方法的拦截器链

##### 	先创建一个List集合，来保存所有拦截器

##### 	遍历所有的增强器，转化为Interceptor

##### 如果有拦截器链，就要执行拦截器链，否则直接执行目标方法

# 5、拦截器链的执行过程

new出代理对象后，就调用其proceed()方法。

![image-20220610132701819](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610132701819.png)

执行proceed()，先是看currentInterceptorIndex这个变量的值。

有两种情况直接进入if语句之中：

1）没有获取到拦截器链，即index = 0，则调用invokeJoinpoint()方法，里面调用了动态代理对象的invoke()，直接执行目标方法

2）当前索引 = 拦截器总数 - 1，执行目标方法

###### 拦截器的invoke()方法就是代理对象的proceed()方法。每一次执行proceed()，当前拦截器的索引都会自增一次。直到拿出最后一个拦截器为止。



###### 锁定到最后一个拦截器时，调用invoke()方法，此时逻辑发生了改变：

![image-20220610133334948](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610133334948.png)

这里调用了前置通知，之后再调用proceed()，但index已经等于拦截器总数-1，所以进入if条件语句，调用目标方法本身。并且目标方法调用完后会返回到上一个拦截器（AfterJAfterAdvice）。

![image-20220610133731778](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610133731778.png)

在finally语句块中执行后置通知，如果目标方法没有抛异常，则调用返回通知；如果目标方法抛出异常，则调用异常通知。

![image-20220610133849648](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220610133849648.png)



###### 链式获取每一个拦截器，然后执行拦截器的invoke()方法，每一个拦截器等待下一个拦截器执行完成并返回，再来执行其invoke()方法。保证了通知方法与目标方法的执行顺序。

# 6、AOP原理总结

### 1、利用@EnableAspectJAutoProxy注解开启AOP功能

### 2、这个功能怎么开启的？

​		通过@EnableAspectJAutoProxy注解向IOC容器中注册一个AnnotationAwareAspectJAutoProxyCreator组件

### 3、AnnotationAwareAspectJAutoProxyCreator组件是一个后置处理器

### 4、这个后置处理器怎么工作的？

​		首先，在创建IOC容器的时候，会调用refresh()方法来刷新容器，而在刷新容器的过程中有一步是用来注册后置处理器的：

```java
registerBeanPostProcessors(beanFactory); // 注册后置处理器，在这一步会创建AnnotationAwareAspectJAutoProxyCreator对象
```

​		在刷新容器的过程中还有一步是来完成BeanFactory的初始化工作的：

```java
finishBeanFactoryInitialization(beanFactory); // 完成BeanFactory的初始化工作。所谓的完成BeanFactory的初始化工作，其实就是来创建剩下的单实例bean的。
```

​		显然剩下的单例bean就包括自定义的MathCalculator类(目标类)和LogAspectJ(切面类)，这两个bean就是在这里创建的

###### 		创建这两个组件的过程中，最核心的就是AnnotationAwareAspectJAutoProxyCreator会来拦截这两个组件的创建过程。

###### 		怎么拦截？组件创建完成后，会判断组件是否需要增强。需要的话会把切面里面的通知方法包装成增强器，然后创建一个代理对象。（如果目标类实现了接口，使用jdk动态代理，否则使用cglib代理）

### 5、执行目标方法

​		此时是代理对象来执行目标方法

​		使用CglibAopProxy类的intercept()方法来拦截目标方法的执行：

​			得到目标方法的拦截器链，其实就是每一个通知方法被包装成了拦截器，即MethodInterceptor

​			利用链式机制，依次进入每一个拦截器中执行

###### 			最终执行结果：

###### 				目标方法正常执行：前置通知-->目标方法-->后置通知-->返回通知

###### 				目标方法出现异常：前置通知-->目标方法-->后置通知-->异常通知

### 简短总结AOP原理：

###### 		在配置类上标注@EnableAspectJAutoProxy，这个注解会向容器中注册一个AnnotationAwareAspectJAutoProxyCreator组件，这个组件是一个BeanPostProcessor，那么在容器创建目标类和切面类的时候，就会执行postProcessBeforeInitialization()方法。

###### 		在组件创建完成后，这个BeanPostProcessor会判断组件是否需要增强，如果需要增强，就把通知方法（增强器）都包装成拦截器，然后创建一个代理对象。执行目标方法的时候是代理对象来执行，并得到拦截链，按照拦截链的顺序执行目标方法。





PS：通过配置类获取容器的过程：

<img src="C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220609150214079.png" alt="image-20220609150214079" style="zoom:150%;" />

调用无参构造，把主配置类注册进去，然后调用refresh()方法刷新容器，就是把容器中所有的bean都创建出来。![image-20220609150255851](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220609150255851.png)

