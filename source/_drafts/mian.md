---
title: 'mian'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
### 头条：
之前做过的项目(不是数据研发嘛，逮着abtest一直问，都是一上来就开始问abtest，没有一点点防备)

算法题1：字符数组，找第一个只出现一次的字符。
写sql：
A表：uid  文章gid  时间date read（是否读取0/1）；
B表：文章gid  类型catergory
求每个文章类型 每天阅读量的次数。

算法题2：旋转数组，找某个数的下标
写sql：
A表：uid  文章gid read（是否读取0/1）；
B表：文章gid  类型catergory
求阅读量前10的文章类型


spark 懂哪些？
数据仓库 建模：原始日志、混洗日志————》ods——————》dw
ods层包含：（去重、去噪、去空、大小写、日期格式）、
           dim（字典表）（Dictionary Data Layer）、
           mysql-》hdfs,
           其他业务线数据分享

分层原因：https://www.cnblogs.com/benchen/p/6010265.html
1、空间换时间。避免用户直接操作最底层数据。
2、复杂问题简单化。出现问题时，只需要修改某写步骤，不需要全部修改。
3、便于处理业务变化。业务变化时，只需要调整底层数据。

偶尔会问一些基础的问题，如java虚拟机、gc。
java 虚拟机、gc:https://www.cnblogs.com/wjtaigwh/p/6635484.html
from where groupby select orderby 解释顺序
partition combine有哪些？

java题1:实现java单例，要求线程安全，只有在调用时才实例化。
概率题2:两个人下棋，一个人胜率是60%，一个是40%.问五局三胜还是三局两胜，你会更容易赢。
https://www.zybang.com/question/bec5fd59ae06ca1f39c3fa8b685adf8f.html
算法题3:序列化、反序列化二叉树。

### xindongfang：
row_number取前5个，如果有重复
sum sum
java的arrayList动态数组
spring的bean如何反射

### aiqiyi
算法题：
柱状图，最大面积
跳台阶
0-1矩阵，找孤岛
判断一个字符串通过移动能否 可以对称

spark的rdd，为什么是弹性的？
hadoop提交任务流程？
hive提交任务的流程？query enegine是怎么的？

### 网络
数据仓库建模经验？为何这样建模？
看不懂的sql：https://blog.csdn.net/liyong199012/article/details/20628627
id、logintime、logouttime  求当天哪分钟在线人数最多。
多次登陆 多次退出 某刻在线人数最多 sql:https://bbs.csdn.net/topics/392146405?page=1

连续数据：
```sh
hive> desc user_active;
uid			string
active_date	string
url			string 
hive> select * from user_active;
866091031298328	2019-01-01	url2
866091031298328	2019-01-22	url4
866091031298328	2019-01-02	url1
866091031298328	2019-01-20	url7
866091031298328	2019-01-03	1url3
866091031298328	2019-01-04	url6
866091031298328	2019-01-25	url5
865175046479931	2019-01-12	url66
865175046479931	2019-01-21	url44
865175046479931	2019-01-05	url11
865175046479931	2019-01-08	1url33
865175046479931	2019-01-20	url77
865175046479931	2019-01-01	url22
865175046479931	2019-01-22	url55
```

查询语句1：
```sql
select uid, active_date, url, move_date, diff
from 
    (
        select uid, active_date, url, move_date, datediff(move_date, active_date) diff
        from 
            (
                select uid, active_date, url, lead(active_date, 2, '1970-01-01') over(partition by uid order by active_date) move_date
                from user_active
            ) m 
    ) n 
where diff=2;
```
查询语句2：
```sql
select uid
from
(
  select uid, active_date, row_number() over(distribute by uid sort by active_date) r
  from user_active
  where uid is not null
)a
join
(
  select uid, active_date, row_number() over(distribute by uid sort by active_date) r
  from user_active
  where uid is not null
)b
on a.uid = b.uid
where b.r between a.r and a.r + 2
group by uid
```