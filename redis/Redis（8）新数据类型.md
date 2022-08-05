# Redis（8）新数据类型



## 1、Bitmaps

![image-20220705102732371](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220705102732371.png)

把bitmaps想象成一个以位为单位的数组，数组的每个单元只能存储0或1，数组的下标在bitmaps中叫做偏移量



#### 命令

setbit <key> <offset> <value> ：设置Bitmaps中某个偏移量的值（0或1），偏移量从0开始

getbit <key> <offset>：取出某个偏移量对应的值

bitcount <key> [start end]：返回值为1的数量

######  -1表示最后一位，-2表示倒数第二位

bitop and(or/not/xor) <destkey> [key...]：将不同key里的值求交集，并集，非，异或并存储在destkey中



## 2、HyperLogLog

处理集合基数集问题时采用，例如数据集{1,3,5,7,5,7,8}，基数集就是{1,3,5,7,8}，基数是5

例如独立IP数，独立访客数等问题



#### 命令

pfadd <key> <element> [element...]：添加元素

pfcount <key> [key...]：数量

pfmerge <destkey> <sourcekey> [sourcekey...]：将一个或多个HLL合并后存在另一个HLL中



## 3、Geographic

地理位置，该类型就是元素的2维坐标，在地图上就是经纬度。



#### 命令

geoadd <key> <longitude> <latitude> <member> [longitude latitude member...]：添加地理位置（经度纬度名称）

###### 有效范围经度-180到180，纬度从-85到85



geopos <key> <member> [member...]：取经纬度



geodist <key> <member1> <member2> [m|km|ft | mi] ：取两个地方的直线距离 



georadius <key> <longitude> <latitude> radius m|km|ft|mi：指定经纬度为中心，取指定半径内的元素