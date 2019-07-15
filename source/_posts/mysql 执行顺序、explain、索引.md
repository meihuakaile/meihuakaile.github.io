---
title: 'mysql 执行顺序、explain、索引'
date: "2018/04/19"
tags: [mysql]
categories: ['mysql']
copyright: true
---
### 用ubuntu命令读取数据库中信息。
```
mysql -h 127.0.0.1 -u root -p XXXX -P 3306 -e "select * from table"  > /tmp/test/txt
```
-h 后面是主机； -u后面是用户名； -p后面是数据库名字 ； -P后面是端口号 ； -e就是mysql的select语句 ； 最后是重定向到一个文件。 

### int(num) varchar(num)
int的num只是定长与存储无关，长度小于num时，左边补空对齐；大于num时，不管。存储长度默认是4字节。eg： int（1）代表显示宽度是1，而不是存储长度。
varchar的会截断数据，是数据长度。但在长度小于255时，存储的长度会是num+1；大于255时，长度会num+2；如原始字符串是‘qwertasdf‘，varchar(5)，最终的存储的字符串是qwert五位，数据库中存储是六位。
char基本类似与varchar，只是存储时不会多存储一位。

### 常用命令
explain sql1；查看sql1的性能。第 5 条解释了它的两个重要参数 rows 和filtered。
show create table table1； 查看table1的建表语句。（建库语句类似）
show index from table1; 查看table1的索引。 第7列是索引的大小。
grant 权限 on 数据库对象 to 用户

上面的语句都可以通过添加\G来使原本的显示按行排列。
eg，explain select * from flight_qmq_pay_order_20171204 order by insert_time desc limit 10\G;
### sql执行顺序
对于一条sql语句来说，执行顺序是这样的：
1、from子句组装来自不同数据源的数据；
(join条件筛选)
2、where子句基于指定的条件对记录行进行筛选；
3、group by子句将数据划分为多个分组；
4、使用聚集函数进行计算；
5、使用having子句筛选分组；
6、计算所有的表达式；
7、使用order by对结果集进行排序；
8、select 集合输出。

由于select在group by之后执行，如果select中出现count等函数都是针对group之后的分组计算。
### explain参数
它主要是对mysql性能评估。下面加粗的字体是几个比较重要的参数。
官网有它的所有参数解释：https://dev.mysql.com/doc/refman/5.7/en/explain-output.html
**_rows：_** 是执行sql语句需要检查的行。对于InnoDB表格，这个数字是一个估计，并不总是准确的。
filtered：在把命令写成explain extended sql时才会出现，意思是输出的行占寻找输出行时搜索的其他行的百分比。 最终输出的行  /  读的行。 也就是说，rows 显示了检查的行数， rows× filtered/ 100显示了与之前的表搜索的行数。
       **_可以查看一个索引的的扫描范围，rows可以看扫描的行数，扫描行数越少效率越高_** 
- id：查询编号
- select_type：
  SIMPLE：    简单查询
  PRIMARY：   最外层的select
  SUBQUERY：  子查询内层查询的第一个select
  DERIVED：   子查询派生表的select
- table：表名
- type：
  index： 全索引扫描
  const： 通过主键访问
  all：   全表扫描,没有使用索引
  range： 索引返回扫描，常见于使用>,<,is null,between ,in ,like等运算符的查询中。
  ref：   索引扫描，结果可能有多个匹配值
  eq_ref：索引扫描，唯一索引匹配值（唯一） 

访问效率：const>eq_ref>ref>range>index>all

- possible_keys： 可能使用到的索引
- key： 最终使用的索引
- key_len： 索引的长度（使用到的）
- rows： 扫描行数
- extra：
  impossible where noticed after reading const tables: mysql优化器通过分析发现不可能存在的结果
  **using index**： 所需要的数据只需要在index即可全部获得而不需要再到表中取数据
  using index for group-by： 当query中使用了group by或者distinct子句的时候，如果分组字段也在索引中，extra中出现该信息
  _using filesort_： query中包含order by，且无法利用索引完成排序操作的时候，mysqlquery optimizer不得不选择相应的排序算法来实现。（并不一定带白哦磁盘顺序）
  _using temporary_： 使用临时表时出现，主要常见于group by和order by等操作中
  using where： 不是读取表的所有数据，或者不是仅仅通过索引就可以获取所有需要的数据，则会出现using where信息

**_extra_**：**_Using filesort， Using temporary说明索引不好需要改进，Using index 说明sql的执行效率很可以_**
**_type_**：连接类型。一个好的sql语句至少要达到range级别。杜绝出现all级别。
**_key_**：使用到的索引名。如果没有选择索引，值是NULL。

参考：https://www.cnblogs.com/david97/p/8072164.html
### 索引注意
**_避免like’%xxx%’_**，使用like ‘xxx%’。mysql可以使用前缀索引扫描。
**_匹配条件字符串要加单引号_**。否则可能会导致全表扫描，数字匹配的时候不要加单引号。
**_order by后的内容尽量放到索引中_**。Where后的内容是=而不是范围性的，就可以和order by一起组成索引。如果where后的是主键id，则可以只给order by建立索引。
**_不要使用子查询_**，因为内部查询的结果在临时表中，没有索引可用。
**_避免有where status=1这样的操作，status的值范围小，一般都会造成全表扫表_**。尽量增加一些时间或者主键id的字段控制。

**_innodb一般都是行锁_**，这个一般指的是sql用到索引的时候，行锁是加在索引上的，不是加在数据记录上的，**_如果sql没有用到索引，仍然会锁定全表_**。
in: 包含的值不宜过多。mysql对in关键字进行了优化，把所有的值放在数组中，且自动对in里的值进行了排序。但是如果数值太多，消耗还是会很大。例如有in(1,2,3,4)这样连续的数值最好用between代替。
select* : 增加很多不必要的消耗（cpu、io、内存、网络带宽）；减少了覆盖索引的可能性，而且如果数据库结构发生了改变，前面的代码也需要相应的改变。

### 易错
update a set order_id=405 and date=’’ where id=’’    **_不应该用and而用逗号。这里会当做一个表达式返回一个值再赋值给order_id_**。
date：一般使用timestamp，5.6.4之后可以使用datetime类型。timestamp会采用系统调用，并发量高的情况下会造成系统自旋锁，建议使用datetime。
Text 比varchar存更多的东西。不能存null，或者默认为null。

### 出错码
28000，用户名或密码错误。
42000，权限问题。

### 物理分页/逻辑分页
物理分页：数据库本身提供的分页方式，比如mysql的limit。但不同数据库不同。
逻辑分页：使用游标（next）分页，先把数据都查出来再分页。
mybatis的RowBound是逻辑分页。
逻辑分页是把数据库的压力放在了应用中，因为是在应用中分的页。

### 权限管理
mysql里用户信息存在`mysql.user`里，通过看这个表可以看到所有用户的权限信息。
通过`SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;`可以查到所有的账户。
这里表里`user+host`必须是唯一的。
`show grants` 查看当前用户的权限。

#### create 创建用户
`create user 'username'@'host' [identified by [password] 'password1' ]` 创建新的用户。没有`[identified by [password] 'password1' ]`代表该用户不需要密码；加上`[password]`表示后面的密码`password1`用password加密。
加密的原因：`create user`语句的操作会被记录到服务器日志文件或者操作历史文件中。有那个文件的人可以可以看到用户的密码，因此需要加密。
如何加密：使用password加密。先使用`select password('password1')`算出加密后的哈希值密码，再用加密后的密码代替上面的`password1`.
#### grant 授权
相比与`create`需要先创建，再授权；`grant`可以直接创建用户并授权。
```mysql
GRANT priv_type [(column_list)] [, priv_type [(column_list)]] ...
    ON [object_type] {tbl_name | * | *.* | db_name.*}
    TO user [IDENTIFIED BY [PASSWORD] 'password']
        [, user [IDENTIFIED BY [PASSWORD] 'password']] ...
    [REQUIRE
        NONE |
        [{SSL| X509}]
        [CIPHER 'cipher' [AND]]
        [ISSUER 'issuer' [AND]]
        [SUBJECT 'subject']]
    [WITH with_option [with_option] ...]
```
简单的，`grant priv on 库.表 to 'username'@'host' [identified by [password] 'password1' ]` 给用户`'username'@'host'`授权。
当需要给多个机器权限时，就需要多个grant语句，此时如果是同一网段的，可以host采用`192.23.%`；甚至可以直接用`%`代表所有机器。

`GRANT USAGE ON` 加空的权限，只能连库什么都没法做。

#### 直接修改表加权限
不管是`create`还是`grant`，其实都是在操纵`user`表。
`INSERT INTO mysql.user(host,user,password,[privilegelist]) VALUES ('host','username',password('password'),privilegevaluelist)`
emmmm，我的mysql没有`password`字段。。。。

#### 删除用户
`drop user ‘username’@‘host’ `/`delete from mysql.user where user='username' and host='host'`

参考：https://www.cnblogs.com/lyhabc/p/3822267.html 

### `Lock wait timeout exceeded;`
`ERROR 1205 (HY000) at line 1: Lock wait timeout exceeded; try restarting transaction`
`show full processlist;`
`kill id;` 杀掉锁住的id进程。

参考：https://blog.csdn.net/wp1603710463/article/details/51721894/