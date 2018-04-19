---
title: 'mysql语句'
date: "2018/04/19"
tags: [mysql]
categories: ['mysql']
copyright: true
---
select * from table1 a inner join table2 b on a.id=b.id;

注意mysql语句的执行顺序，正确的使用别名。

### sql join
![](2.png)
#### （ inner）join on
输出两张表中的列。
select * from table1 inner join table2 on 条件
#### table1 left join table2 on 条件
左表的数据会全部输出来，没有对应数据的补null。（如上面的语句，“全部”体现在id上，即on之后的比较，本来1中是只有在两个id相等时才会select。但是如果是本例中会把左边表的id相关数据全部输出，如左表有id=3右表没有，但是还是会把左表等于3的那列的select数据输出来，右表没有相关数据只能补null）
#### ...right join ....on....
右表的数据会全部输出来，没有的补null。
https://www.cnblogs.com/dinglinyong/p/6656315.html
#### UNION
用于合并两个或多个 SELECT 语句的结果集。
UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。每条 SELECT 语句中的列的顺序必须相同。
UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。
UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。
优化建议：能使用union all，就不要使用union。因为union还要对数据进行排序后筛除重复的。比较费时。

### in/exit
以优化角度考虑，一般以小表驱动大表。in语句是先执行子语句，exit是后执行子语句；
因此，如果子语句是小表就用in，是大表就用exit；
in语法：select * from 表A where id in (select id from 表B)
exit语法：select * from 表A where exists(select * from 表B where 表B.id=表A.id)

### limit优化

数据库数据很多的时候会发现分页查询会越来越慢，使用一个id控制就会变快很多（但其实思考到，加入表里数据并不规整有删除的情况，可能会无法使用）。
如，select id,name from product limit 866613, 20。——》 select id,name from product where id> 866612 limit 20；

### 导入导出数据

linux命令mysql 的参数-e可以后面可以直接跟mysql语句。
使用-e在终端执行使在导入/出命令前加上“set character_set_database=utf8;” 可以有效避免中文乱码。
导入数据： 
```
LOAD DATA INFILE '/home/chenliclchen/mysql/tableExport.txt' INTO TABLE `atp_event`(
event_name, business, event_type, start_time, end_time, event_area, description); 
```
导入数据出错参考：ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
导入中文乱码时参考：[导入数据乱码](/2018/04/19/ERROR 1290 HY000  The MySQL server is running with the --secure-file-priv option so it cannot execute this statement)

导出数据： 
```
select * into outfile '/home/chenliclchen/mysql/t.txt' fields terminated by ',' from atp_event;  
```
### coalesce()
返回第一个不为null的值

### count(null) 是0
如图，来源于网络的例子
![](1.png)
### 日期
#### DATE_FORMAT(date,format)
date 参数是合法的日期。format 规定日期/时间的输出格式。
#### DATE_SUB(date,INTERVAL expr type)  
DATE_ADD(date,INTERVAL expr type)
date 参数是合法的日期表达式。expr 参数是您希望添加的时间间隔。type 是MINUTE/HOUR/DAY/WEEK 等。
#### DATEDIFF(date1,date2)
date1 和 date2 参数是合法的日期或日期/时间表达式。 返回两个日期之间的天数。
#### NOW(),CURDATE(),CURTIME()
当前日期+时间，日期，时间

### format(number, length)
number是浮点数。length约束浮点数小数位数。
### round(x[, d]) 
x指要处理的数，d是指保留几位小数。用于数据的四舍五入。d默认为0。d可以是负数，这时是指定小数点左边的d位整数位为0,同时小数位均为0。
### LEFT(str,len)/RIGHT(str,len)
LEFT(str,len)
返回字符串str的最左面len个字符。

RIGHT(str,len)
返回字符串str的最右面len个字符。