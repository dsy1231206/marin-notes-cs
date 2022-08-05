# Spring注解开发-声明式事务

## 1、配置数据源和JdbcTemplete

```java
@Configuration
@PropertySource(value = "classpath:dbconfig.properties")
public class TxConfig {

    //注册c3p0数据源
    @Bean("txDataSource")
    public DataSource dataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("${mysql.user}");
        dataSource.setPassword("${mysql.password}");
        dataSource.setDriverClass("${mysql.driverClass}");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/spring_test");
        return dataSource;
    }
    
    @Bean
    public JdbcTemplate jdbcTemplate() throws PropertyVetoException {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
        return jdbcTemplate;
    }
}
```

###### JdbcTemplate组件：它是Spring提供的一个简化数据库操作的工具，能简化对数据库增删改查的操作。

###### 在创建JdbcTemplate对象的时候，把数据源传入JdbcTemplate，因为需要获取数据库连接。



也可以在把dataSource作为一个参数给这个方法，例如：

```java
@Bean
public JdbcTemplate jdbcTemplate(DataSource dataSource) throws Exception {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    return jdbcTemplate;
}
```

###### 因为被@Bean标注的方法在执行时，参数的值是从IOC容器中获取的。

## 2、创建Dao类，Service类

```java
@Repository
public class UserDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void insert(){
        String sql = "INSERT INTO tbl_user(username,age) VALUES (?,?)";
        String username = UUID.randomUUID().toString().substring(0,5);
        jdbcTemplate.update(sql,username,19);
    }
}
```

```java
@Service
public class UserService {

    @Autowired
    private UserDao userDao;

    public void insertUser(){
        userDao.insert();
        System.out.println("插入完成...");
    }
}
```

再创建一个测试类，从IOC容器中获取到UserService对象，执行方法能正常插入成功。

###### 开启事务：在方法上加上@Transactional，只要这个方法里有任何一句代码出问题，都会回滚所有操作。

```java
@Transactional
public void insertUser() {
	userDao.insert();
	// otherDao.other(); // 该方法中的业务逻辑势必不会像现在这么简单，肯定还会调用其他dao的方法
	System.out.println("插入完成...");
	
	int i = 10 / 0;
}
```

###### 但是，除了加上@Transactional之外，还要在配置类加上@EnableTransactionManagement，开启基于注解的事务管理功能。

如果是基于XML配置文件的话，需要在配置文件中添加一行配置：

```xml
<tx:annotation-driven/>
```

###### 

###### 除了加上以上两个注解之外，还缺少一个东西：

PlatformTransactionManager，基于平台的事务管理器，用得最多的一个实现类是DataSourceTransactionManager

需要在IOC容器中注入一个bean

```java
@Bean
    public PlatformTransactionManager platformTransactionManager() throws PropertyVetoException {
        return new DataSourceTransactionManager(dataSource());
    }
```

## 3、声明式事务原理的源码分析

#### 1）@EnableTransactionManagement

![image-20220613105240873](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613105240873.png)

###### 可以看到，这个注解给容器中引入了一个bean，类型是TransactionManagementConfigurationSelector

![image-20220613105314672](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613105314672.png)

###### 它又继承了一个类叫AdviceModeImportSelector

![image-20220613105354393](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613105354393.png)

###### 这个类又实现了一个接口，ImportSelector

###### 说明TransactionManagementConfigurationSelector就是给容器中导入一些组件的。

###### 

###### 在这个类里面会做一个switch判断，如果adviceMode是PROXY，那么就会返回一个String[]

```java
new String[] {AutoProxyRegistrar.class.getName(), ProxyTransactionManagementConfiguration.class.getName()};
```

###### 如果adviceMode是ASPECTJ，那么就返回如下String[]:

```java
new String[] {TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME};
```

​	

###### adviceMode是啥？它是一个枚举：

![image-20220613105822696](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613105822696.png)

###### 在@EnableTransactionManegement里就定义了一个枚举

![image-20220613105950040](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613105950040.png)

###### 所以，这个注解会给容器中导入两个组件：

###### AutoProxyRegistrar和ProxyTransactionManagementConfiguration



## 2、AutoProxyRegistrar

![image-20220613110117113](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613110117113.png)

###### 他实现了一个接口ImportBeanDefinitionRegistrar，所以这个组件就是向容器中注册bean的，最终会调用registerBeanDefinitions()方法向容器中注册bean。

![image-20220613111608790](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613111608790.png)

###### 最终会向容器中注入一个组件，自动代理创建器，即InfrastructureAdvisorAutoProxyCreator

###### 它同样也是一个后置处理器。利用后置器处理机制在对象创建之后进行包装，返回一个代理对象吗，并且代理对象里存有所有的增强器。代理对象执行目标方法，会利用拦截器的链式机制，依次进入每一个拦截器进行执行。

## 3、ProxyTransactionManagementConfiguration

###### 这是一个配置类，利用@Bean向容器中注册了多个组件，第一个组件就是BeanFactoryTransactionAttributeSourceAdvisor，是事务的核心内容。

![image-20220613112819923](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613112819923.png)

这就是向容器中注册一个事务增强器：BeanFactoryTransactionAttributeSourceAdvisor

注册事务增强器时，需要用到事务属性源：TransactionAttributeSource，可以看到是new了一个AnnotationTransactionAttributeSource，它的作用就是遇到@Transactional注解时，分析此事务注解。

![image-20220613134233568](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613134233568.png)



注册事务增强器时，还需要用到事务的拦截器

![image-20220613134320368](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613134320368.png)

该事务拦截器会将事务属性源放进去，还会将事务管理器放进去。

###### 事务拦截器源码：

![image-20220613134439381](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613134439381.png)

先是获取事务属性源，获取事务管理器：

![image-20220613134604489](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613134604489.png)

如果目标方法一切正常，就用事务管理器提交，如果不正常就结束事务，回滚![image-20220613134709917](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613134709917.png)

## 4、总结

###### 首先，使用AutoProxyRegistrar向Spring容器里注册一个后置处理器，这个后置处理器会负责包装代理对象。然后，使用ProxyTransactionMnagementConfiguration向容器中注册一个事务增强器，需要用到事务拦截器。最后，代理对象执行目标方法，在这个过程中，会执行拦截器链，每次执行目标方法时，如果出现了异常，那么就利用事务管理器回滚事务，如果一切正常，就用事务管理器提交事务。