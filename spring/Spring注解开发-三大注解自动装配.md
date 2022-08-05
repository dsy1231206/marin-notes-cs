# Spring注解开发-三大注解自动装配

# 1、@Autowired

![image-20220608101553498](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608101553498.png)

###### 1、默认是优先按照类型去找组件，ByType，相当于调用了applicationContext.getBean(类名.class)

###### 2、如果找到了多个同类型的组件，那么就ByName，调用applicationContext.getBean("组件的id")

### @Qualifier

###### @Autowired默认是优先按类型进行匹配的，如果需要按名称进行装配，就需要配合@Qualifier

###### 明确指定有多个同类型组件时自动装配哪一个

![image-20220608101815623](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608101815623.png)

### @Primary

###### 使用@Autowired自动装配时，对于同一个接口可能有不同的实现类，就可以使用@Primary指定优先采用哪一个类

在@Bean处添加了@Primary注解的bean，会优先被装配

但如果需要具体指定，还是用@Qualifier

# 2、其他注解

#### @Resource

###### Java规范里的注解，默认按照名称进行装配，可以通过name属性指定

###### 虽然和@Autowired一样，都能实现自动装配，但不支持@Primary优先注入，也不能添加required = false属性

###### 如果需要指定注入哪个bean，添加属性

```java
@Resource(name = "bookDao2")
```

![image-20220608105220903](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608105220903.png)

#### @Inject

![image-20220608105344589](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608105344589.png)

###### 该注解默认根据参数名去寻找bean注入，支持Spring的@Primary注解优先注入

###### 和@Autowired一样都能自动装配，但不能添加required属性

需要导入依赖：

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

# 3、方法、构造器上的自动装配

###### @Autowired的注解上面有这样一行：

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
```

说明@Autowired注解还可以添加到实例方法、构造方法和参数上

###### 标注在方法和参数上，参数会从IOC容器中获取。使用最多的方式是@Bean+方法参数的形式，如果只有一个IOC容器中的对象作为参数，则可以省略@Autowired

