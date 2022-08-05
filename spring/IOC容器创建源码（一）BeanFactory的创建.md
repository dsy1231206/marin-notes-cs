# IOC容器创建源码（一）BeanFactory的创建

## 1、BeanFactory的创建和准备工作

![image-20220615093234675](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615093234675.png)

###### 核心就是refresh()方法

![image-20220615093304229](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615093304229.png)

#### 1）prepareRefresh()：刷新容器前的预处理工作

```java
protected void prepareRefresh() {
        this.startupDate = System.currentTimeMillis();
        this.closed.set(false);
        this.active.set(true);
        if (this.logger.isDebugEnabled()) {
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Refreshing " + this);
            } else {
                this.logger.debug("Refreshing " + this.getDisplayName());
            }
        }

        this.initPropertySources();
        this.getEnvironment().validateRequiredProperties();
        if (this.earlyApplicationListeners == null) {
            this.earlyApplicationListeners = new LinkedHashSet(this.applicationListeners);
        } else {
            this.applicationListeners.clear();
            this.applicationListeners.addAll(this.earlyApplicationListeners);
        }

        this.earlyApplicationEvents = new LinkedHashSet();
    }
```

###### 注意里面有一个earlyApplicationEvents，主要是保存一些容器中早期的事件，等事件派发器可用后，再把事件派发出去。

#### 2）obtainFreshBeanFactory()：获取BeanFactory对象

![image-20220615093917789](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615093917789.png)

![image-20220615093942294](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615093942294.png)

refreshBeanFactory()方法就是创建了一个BeanFactory对象（DefaultListableBeanFactory类型的），并设置好了一个序列化id，返回之后再用ConfigurationListableBeanFactory这个类型去接收beanFactory。

#### 3）prepareBeanFactory(beanFactory)：BeanFactory的预准备工作

![image-20220615094358807](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615094358807.png)

###### 这一步会设置非常多的东西，还会向容器中注入一些bean，比如说代表环境变量的bean

#### 4）postProcessBeanFactory(beanFactory)：BeanFactory准备工作完成后进行的后置处理工作

![image-20220615094753751](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220615094753751.png)

目前还是空的，是protected类型的方法，这就是给子类使用的方法，可以在里面拓展一些功能。

###### 目前IOC容器准备工作就做完了