# Spring注解开发-注入Spring底层组件

###### 自定义组件注入Spring底层的组件，比如ApplicationContext（IOC容器）、底层的BeanFactory等，只需要让自定义的组件实现XxxAware接口即可，就可以在自定义的组件内使用底层Spring组件对象。

# 1、XxxAware接口

![image-20220608135320095](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608135320095.png)

### 常用的XxxAware接口：

### 1）ApplicationContextAware接口

​		实现ApplicationContextAware接口，实现setApplicationContext()方法

​		每一个XxxAware接口都有一个对应的XxxAwareProcessor实现类，

​		这个实现类又实现了BeanPostProcessor，本质上是一个后置处理器

![image-20220608141411877](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608141411877.png)

![image-20220608141435240](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608141435240.png)

![image-20220608141444877](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608141444877.png)

在Bean初始化前就调用了setApplicationContext()方法，把底层组件注入。

# # 环境配置切换

## @Profile

![image-20220608144431560](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220608144431560.png)

###### 可以标注在方法（@Bean）上，配置类上

###### 如果标注在类上，那么在指定的环境下配置类才会生效

###### 如果没有标注@Profile，那么在任何环境下bean都会被加载进IOC容器

```java
@Configuration
@PropertySource("classpath:dbconfig.properties")
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${mysql.user}")
    private String user;

    private StringValueResolver valueResolver;

    private String driverClass;

    @Profile("test")
    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${mysql.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/spring_test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("default") //默认环境
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${mysql.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/spring_dev");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${mysql.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/spring_prod");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.valueResolver = resolver;
        driverClass = valueResolver.resolveStringValue("${mysql.driverClass}");
    }
}
```

