# SQL语句-insert

### 一、插入语句的四种方式：

#### 1、普通插入（全字段）

```sql
insert into table_name values(value1,value2...)
```

#### 2、普通插入（限定字段）

```sql
insert into table_name(col1,col2...) values(value1,value2...)
```

#### 3、多条一次性插入

```sql
insert into table_name(col1,col2...) values(valuea1,valuea2...),(valueb1,valueb2...)
```

#### 4、从另一个表导入

```sql
insert into table_name SELECT * FROM table_name2 [WHERE key = value]
```

例题：

![image-20220815152256947](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220815152256947.png)

有一张表exam_record，现在要精简表，只保留2021年前已经完成的记录，保存到表exam_record_before_2021中

```sql
insert into exam_record_before_2021(uid,exam_id,start_time,submit_time,score)
select uid,exam_id,start_time,submit_time,score from exam_record where year(submit_time) < '2021'
```



### 二、插入语句时遇到的一些问题

#### 1、某个unique字段重复，但需要替换它

###### 可以先删除该条数据，再插入新的

```sql
delete from examination_info where exam_id = 9003;
insert into examination_info (exam_id,tag,difficulty,duration,release_time) values(9003,'SQL','hard',90,'2021-01-01 00:00:00');
```

###### 或者可以使用replace into ...

和insert into 功能相似，replace into不同的在于：首先判断（主键索引或唯一索引）是否已经存在，如果存在就先删除这条数据，然后再插入新的数据，否则直接插入

要注意的是，表里必须有主键索引或者唯一索引，否则会直接插入数据，可能导致重复。