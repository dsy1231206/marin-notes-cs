# Redis（7）发布和订阅



## 1、什么是发布和订阅？

​	Redis发布订阅(pub/sub)是一种消息通信模式：发布者(pub)发送消息，订阅者(sub)接收消息

​	Redis客户端可以订阅任意数量的频道

![image-20220705102235899](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220705102235899.png)



## 2、命令行实现

###### 订阅：subscribe <channel>

打开一个客户端，订阅channel1

 subscribe channel1



###### 发送：publish <channel> <message>

打开另一个客户端，向channel1发布消息

publish channel1 hello