# Spring注解开发-属性注入

# 一、@Value

### 1、@Value源码

![image-20220607182158757](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220607182158757.png)

可以用在变量、方法、参数、注解上

### 2、用法

###### 不通过配置文件来注入属性

```java
@Value("marin")
private String name;//注入普通字符串
```

```java
@Value("#{systemProperties['os.name']}")
private String systemPropertiesName;//注入操作系统属性
```

###### 通过配置文件来注入

1）创建配置文件，xxx.properties

2）在配置类上使用@PropertiesSource(value = {"classpath:/xxx.properties"})

```java
@PropertySource(value = {"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {

    @Bean
    public Person person(){
        return new Person();
    }
}
```

###### 读取外部配置文件中的key/value，保存到运行的环境变量中

就可以在实体类的属性上使用@Value("${person.nickName}")  // 获取环境变量中的value

# 二、@PropertySource加载配置文件

### 1、@PropertySource

###### 通过@PeopertySource注解可以将properties配置文件中的key/value存储到Spring的Environment中，Environment接口提供了方法读取配置文件中的值，参数是properties文件中的key。也可以使用${}占位符来获取value

![image-20220608094956568](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608094956568.png)

###### 可以同时引用多个配置文件，例如：

```java
@PropertySource(value = {"classpath:/user.properties","classpath:/person.properties"})
```

### 2、@PropertySources

![image-20220608095328971](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608095328971.png)

###### 注解里面有一个属性，PropertySource类型的数组，所以就可以这样加载配置文件：

```java
@PropertySources(value = {
	@PropertySource(value = {"classpath:/person.properties"}),
	@PropertySource(value = {"classpath:/user.properties"})
})
```

### 3、给属性注入值

###### 1）如果使用XML配置文件读取person.properties文件中的值，那么需要在XML配置文件中引入context命名空间，导入properties文件。

```xml
<context:property-placeholder location="classpath:person.properties" />

<!-- 注册组件 -->
<bean id="person" class="com.meimeixia.bean.Person">
    <property name="age" value="18"></property>
    <property name="name" value="liayun"></property>
    <property name="nickName" value="${person.nickName}"></property>
</bean>

```

###### 2）使用注解

在配置类上方添加@PropertySource注解把配置文件中的key/value读取到系统环境中，在实体类上添加@Value("${xxx.xxx}")来读取value

```java
@Value("${person.nickName}")
private String nickName; // 昵称
```

###### 3）使用Environment读取

```java
    /**
     * 使用Environment来获取配置文件的值
     */
    @Test
    public void test02(){
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfigOfPropertyValues.class);
        ConfigurableEnvironment environment = context.getEnvironment();
        String nickName = environment.getProperty("person.nickName");
        System.out.println(nickName);
    }
```

