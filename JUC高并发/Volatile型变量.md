# Volatile型变量

##### 概述Volatile关键字的作用：

###### 当一个变量被定义成volatile之后，它将具备两项特征：

###### 1、第一项是保证此变量对所有线程的可见性，这里的可见性是指 当一个线程修改了这个变量的值，新值对于其他线程是立即得知的，而普通变量不能做到这一点。普通变量的值在线程间传递均需要通过主内存来完成。

但基于volatile变量的运算并不能说就是并发下线程安全的。虽然volatile变量并不存在一致性问题（从物理存储角度看，各个线程工作内存中的volatile变量也会存在不一致的情况，但每次使用之前都需要刷新，执行引擎看不到不一致的情况），但Java里面的运算符操作并不是原子性的，这也导致volatile变量的运算在并发下也是不安全的。

在不符合以下规则的场景中，我们仍然需要通过加锁synchronized、java.util.concurrent包中的锁或原子类来保证原子性：

​	1）运算结果并不依赖于volatile变量的当前值，或者确保只有一条线程来修改它

​	2）变量不需要与其他的状态变量共同参与不变约束。

例如：

```java
volatile boolean shutdownRequested;

public void shutdown(){
	shutdownRequested = true;
}

public void doWork(){
	while(!shutdownRequested);
	//业务逻辑
}
```



###### 2、volatile的第二个语义是禁止指令重排序优化

普通的变量仅会保证在该方法执行过程中所有依赖赋值结果的地方都能获取到正确的结果，但不能保证变量赋值操作的顺序与程序代码中的顺序一致。

例如以下伪代码：

```java
Map configOptions;
char[] configText;
//此变量必须是volatile
volatile boolean initialized = false;

//假设以下代码在线程A中执行
//模拟读取配置信息，全部读取完成后才能通过其他线程配置可用
configOptions = new HashMap();
configText = readConfigFile(fileName);
processConfigOptions(configText,configOptions);
initialized = true;

//假设以下代码在线程B中执行
while(!initialized){
	sleep();
}
doSomethingWithConfig();
```

如果没有禁止指令重排序，线程A中可能先把initialized置为true，线程B中可能就获取到错误的信息。

而使用volatile关键字可以避免这种情况。



##### 解释：

为什么使用volatile修饰的变量能够具有可见性？

例如一段经典的单例代码：

```java
private volatile static Singleton instance;

public static Singleton getInstance(){
	if(instance == null){
		synchronized(Singleton.class){
			if(instance == null){
				instance = new Singleton();
			}
		}
	}
	return instance;
}

public static void main(String[] args){
	Singleton.getInstance();
}
```

这段代码编译后的字节码如下图：

![image-20220817104641676](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220817104641676.png)

相对于普通的变量，volatile修饰的变量在赋值后会多执行一个lock addl $0x0,{%esp}操作，这个操作相当于一个内存屏障，指重排序时不能将后面的指令重排序到内存屏障之前的位置。

这里的关键在于lock前缀，它的作用是将本处理器的缓存写入内存，该写入动作也会引起其他处理器或别的内核无效化其缓存。

所以，通过这个操作，可以让volatile变量的修改对其他处理器立即可见。



###### 但有些操作是无法任意排序的，例如地址A中的值先加5，再乘3，这种操作是不允许任意排序的。lock指令把修改同步到内存时，就代表之前的操作都已经完成，这样就形成了【指令重排序无法越过内存屏障】的效果。