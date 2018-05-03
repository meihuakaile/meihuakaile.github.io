---
title: 'join on和 where 条件放置区别'
date: "2018/05/03"
tags: [mysql]
categories: ['mysql']
copyright: true
---
执行顺序：先on条件再 where条件筛选。
on筛选之后会生成一个临时表；where在临时表上再进行筛选。
（inner）join on时和where的效果一样。
left/right on时与where效果不同。两表都为空时，显而易见left/right on时总会有结果出来，where可能会导致结果为空。
本应放在on的条件放在了where时会失去left/right操作，效果会等同于（inner）join on
### 例子
有两个表：
```mysql
mysql> select * from user;
+----+--------+--------+
| id | name   | pwd    |
+----+--------+--------+
|  1 | lilili | 123456 |
+----+--------+--------+
1 row in set (0.01 sec)

mysql> select * from hotel;
+----+-------+-------+
| id | level | name  |
+----+-------+-------+
|  1 |     2 | test1 |
|  2 |     2 | test2 |
|  3 |     2 | test3 |
|  4 |     1 | test3 |
|  5 |     2 | test2 |
+----+-------+-------+
5 rows in set (0.00 sec)
```
`select * from user u left join hotel h on u.id=h.id where h.level=1;`和
`select * from user u left join hotel h on u.id=h.id and h.level=1;`  区别
下面例子有三条命令。明显看到第二条命令在第一个上加了一个where条件导致输出为空。
但是第三条命令把第二条多加的where加到了on中判断，使得右表没有对应的数据全部以null填充。
```mysql
mysql> select * from user u left join hotel h on u.id=h.id;
+----+--------+--------+----+-------+-------+
| id | name   | pwd    | id | level | name  |
+----+--------+--------+----+-------+-------+
|  1 | lilili | 123456 |  1 |     2 | test1 |
+----+--------+--------+----+-------+-------+
1 row in set (0.00 sec)

mysql> select * from user u left join hotel h on u.id=h.id where h.level=1;
Empty set (0.00 sec)

mysql> select * from user u left join hotel h on u.id=h.id and h.level=1;
+----+--------+--------+------+-------+------+
| id | name   | pwd    | id   | level | name |
+----+--------+--------+------+-------+------+
|  1 | lilili | 123456 | NULL |  NULL | NULL |
+----+--------+--------+------+-------+------+
1 row in set (0.00 sec)
```
### 总结
如果需要不满足连接条件的行也在我们的查询范围内的话，我们就必需把连接条件放在ON后面，而不能放在WHERE后面，如果我们把连接条件放在了WHERE后面，那么所有的LEFT,RIGHT,等这些操作将不起任何作用，对于这种情况，它的效果就完全等同于INNER连接。
所有的连接条件都必需要放在ON后面，不然前面的所有LEFT,和RIGHT关联将作为摆设，而不起任何作用.

参考：https://blog.csdn.net/muxiaoshan/article/details/7617533