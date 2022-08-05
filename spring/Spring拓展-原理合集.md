# Spring拓展-原理合集

# 1、BeanFactoryPostProcessor

BeanFactoryPostProcessor就是BeanFactory的后置处理器，在创建完BeanFactory之后执行。

###### 在加载配置文件之后，都要刷新容器，这一步就是创建各种bean。

![image-20220613142539947](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613142539947.png)

###### 还要执行在容器中注册的BeanFactoryPostProcessor的方法

![image-20220613142706493](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613142706493.png)

###### 具体逻辑就是拿到List，里面都是注册的BeanFactoryPostProcessor，按照PriorityOrdered、Ordered、和默认的顺序执行。

#### 结论：

###### 首先从IOC容器中找到所有类型是BeanFactoryPostProcessor的组件，然后再来执行他们其中的方法，而且是在初始化创建其他组件前面执行。

# 2、BeanDefinitionRegistryPostProcessor

###### 它是BeanFactoryPostProcessor的一个子接口

![image-20220613145905437](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220613145905437.png)

###### 执行时机是在所有的bean定义信息将要被加载，但bean实例还未创建的时候，这个后置处理器允许我们修改bean定义注册中心。

实际应用：自己可以实现一个BeanDefinitionRegistryPostProcessor：

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    /**
     * 这个BeanDefinitionRegistry就是bean定义信息的保存中心，这个注册中心里存储了所有的bean定义信息
     * 以后，BeanFactory就是按照BeanDefinitionRegistry里面保存的每一个bean定义信息来创建bean实例的
     *
     * bean定义信息包括哪些？
     *      单例还是多例的，bean的类型，bean的id
     *      也就是说，这些信息都是存在BeanDefinitionRegistry里面的
     *
     * @param beanDefinitionRegistry
     * @throws BeansException
     */
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry beanDefinitionRegistry) throws BeansException {
        System.out.println("postProcessBeanDefinitionRegistry...bean的数量" + beanDefinitionRegistry.getBeanDefinitionCount());
        //接下来给容器中生成一个对象
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(Blue.class).getBeanDefinition();
        beanDefinitionRegistry.registerBeanDefinition("helloBlue",beanDefinition);
    }

    /**
     *
     * @param configurableListableBeanFactory
     * @throws BeansException
     */
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println("MyBeanDefinitionRegistryPostProcessor..bean的数量" + configurableListableBeanFactory.getBeanDefinitionCount());
    }
}
```

###### 运行顺序：BeanDefinitionRegistryPostProcessor是优先于BeanFactoryPostProcessor执行的，而且可以利用它再给容器中额外添加一些组件。

###### 所以，总的流程就是：

​	1）创建IOC容器

​	2）调用刷新方法refresh()

​	3）从IOC容器中拿到所有的BeanDefinitionRegistryPostProcessor组件，并以此执行他们的postProcessBeanDefinitionRegistry()方法，然后再执行他们的postProcessBeanFactory()方法

​	4）再从IOC容器中拿到所有的BeanFactoryPostProcessor组件，并依次执行他们的postProcessBeanFactory()方法