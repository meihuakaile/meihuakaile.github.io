---
title: hive笔记
tags:
  - hadoop
  - hive
categories:
  - hadoop
copyright: true
date: 2018-04-19
---

架构在Hadoop之上，提供简单的sql查询功能，可以**_将sql语句转换为MapReduce任务进行运行(增删改查)_**。
所有的增删改查操作都是应用在hdfs上的。Hive 中所有的数据都存储在 HDFS 中，Hive 中包含以下数据模型：Table，External Table，Partition，Bucket。
hive是一个数据仓库工具，作用是可以将结构化的数据文件映射为一张数据库表，并提供简单查询功能，可以将sql语句转化为Mapreduce任务进行，是在Hadoop上的数据库基础架构。
**Hive 不是一个关系数据库/实时查询和行级更新的语言.**

Hadoop是一个开源框架来存储和处理大型数据在分布式环境中。它包含两个模块，一个是MapReduce，另外一个是Hadoop分布式文件系统（HDFS）:
* **MapReduce**：它是一种并行编程模型在大型集群普通硬件可用于处理大型结构化，半结构化和非结构化数据。
* **HDFS**：Hadoop分布式文件系统是Hadoop的框架的一部分，用于存储和处理数据集。它提供了一个容错文件系统在普通硬件上运行。

hive的安装需要安装MySQL,因为hive 默认的数据库是Derby数据库，其与MySQL数据库比较存在缺陷，比如不可以执行两个并发的Hive CLI。
**_Hive 将元数据存储在数据库中，如 mysql、derby。Hive 中的元数据包括表的名字，表的列和分区及其属性，表的属性（是否为外部表等），表的数据所在目录等。_**

Hive只是一个客户端，在安装时，我们可以在Hadoop集群中，选择一台安装Hive。Hive没有集群的概念，但是可以搭建Server/Client端。
MapReduce任务(job)的启动需要消耗较长时间，所以Hive的查询延时比较严重。在传统数据库中秒级的任务，在Hive仍需要更长时间。Hive适用不需要实时响应查询的数据仓库程序，不需要记录级别的增删改
Hive不支持事务。提交查询和返回结果可能有很大的延时，此时选用NoSQL数据库，Hbase等。
### Hive框架的作用
（1）可以让不懂java的数据分析人员使用hadoop进行数据分析；
（2）MapReduce开发非常繁琐复杂，使用hive可以提高效率。
（3）统一的元数据管理，可与impala/spark共享元数据。

### hive在hdfs上的文件结构
```
　　    数据仓库的位置                数据库目录           表目录          表的数据文件
　　/user/hive/warehouse             /test.db             /row_table       /hive_test.txt
```
Hive的数据都是存储在HDFS上的，默认有一个根目录，在hive-site.xml中，由参数hive.metastore.warehouse.dir指定。
default是默认的数据库：指的就是这个/user/hive/warehouse路径，因此表就直接在这个目录下
参考：https://www.cnblogs.com/xningge/p/8439970.html

### queuename
hadoop相关。通过`set mapreduce.job.queuename`可以查看当前定义队列名。
队列是跟用户对应的，哪个用户要执行，需要指定哪个队列。

### 元数据
Hive中表和分区的所有元数据都存储在Hive的元存储（Metastore）中。
元数据使用JPOX（Java Persistent Objects）对象关系映射解决方案进行持久化，所以任何被JPOX支持的存储都可以被Hive使用。
大多数商业关系型数据库和许多开源的数据存储都被支持，所以就可以被Hive使用存储元数据。Hive支持三种不同的元存储服务器，分别为：内嵌式元存储、本地元存储、远程元存储，每种存储方式使用不同的配置参数，

内嵌式元存储：主要用于单元测试，在该模式下每次只有一个进程可以连接到元存储，Derby是内嵌式元存储的默认数据库。
本地模式：每个Hive客户端都会打开到数据存储的连接并在该连接上请求SQL查询。
远程模式：所有的Hive客户端都将打开一个到元数据服务器的连接，该服务器依次查询元数据。

参考：https://blog.csdn.net/skywalker_only/article/details/26219619（三种元数据存储方式）
http://www.cloudera.com/documentation/cdh/5-1-x/CDH5-Installation-Guide/cdh5ig_hive_metastore_configure.html

### 分区
hive引入partition和bucket的概念，中文翻译分别为分区和桶，这两个概念都是把数据划分成块，分区是粗粒度的划分桶是细粒度的划分，这样做为了可以让查询发生在小范围的数据上以提高效率。
在Hive Select查询中一般会扫描整个表内容，会消耗很多时间做没必要的工作。有时候只需要扫描表中关心的一部分数据，因此建表时引入了partition概念。
分区表指的是在创建表时指定的partition的分区空间。如果需要创建有分区的表，需要在create表的时候调用可选参数partitioned by。
一个表可以拥有一个或者多个分区，每个分区以文件夹的形式单独存在表文件夹的目录下。
分区是以字段的形式在表结构中存在，通过describe table命令可以查看到字段存在，但是该字段不存放实际的数据内容，仅仅是分区的表示。

### 创建单分区/多分区
关于分区维度的选择，我们应该尽量选取那些**_有限且少量的数值集_**作为分区，例如国家、省份就是一个良好的分区，而城市就可能不适合进行分区。
分区建表分为2种，一种是单分区，也就是说在表文件夹目录下只有一级文件夹目录。另外一种是多分区，表文件夹下出现多文件夹嵌套模式。
a、单分区建表语句：`create table day_table (id int, content string) partitioned by (dt string);`单分区表，按天分区，在表结构中存在id，content，dt三列。
b、双分区建表语句：`create table day_hour_table (id int, content string) partitioned by (dt string, hour string);`双分区表，按天和小时分区，在表结构中新增加了dt和hour两列。多个分区意味着多级目录。
分区是数据表中的一个列名，但是这个列并不占有表的实际存储空间。它作为一个虚拟列而存在。
### 查看/增加/删除分区
`show partitions table_name [partition(...)] ` 查看表所有的分区；加上可选`[partition(...)]`可以查看指定分区是否存在。
`alter table xxx add [if not exist] partition (dt='2018-05-22')`  对分区名是dt的表增加分区
`alter table table_name drop partition (dt='2018-05-22')` 删除分区
`alter table table_name partition(dt='...') set localtion '...' ` 修改分区地址（不会修改/删除旧的分区数据）
`alter table table_name drop [if exist] partition (dt='...') ` 删除分区。如果是内部表，还会删除数据。
**_当外部表是分区表时，只有建立对应的分区，才能查到数据. 删除内部表的分区会删除相应的数据。_**

### Buckets 桶
假设我们有一张地域姓名表并按城市分区。那么很有可能，北京分区的人数会远远大于其他分区，该分区的数据I/O吞吐效率将成为查询的瓶颈。如果我们对表中的姓名做分桶，将姓名按哈希值分发到桶中，每个桶将分配到大致均匀的人数。
分桶解决的是数据倾斜的问题。

Hive采用**_对列值哈希，然后除以桶的个数求余_**的方式决定该条记录存放在哪个桶当中。
对指定列计算 hash，根据 hash 值切分数据，目的是为了并行，每一个 Bucket 对应一个文件。将 user 列分散至 32 个 bucket，首先对 user 列的值计算 hash，对应 hash 值为 0 的 HDFS 目录为：/warehouse/app/dt=20100801/ctry=US/part-00000；hash 值为 20 的 HDFS 目录为：/warehouse/app/dt=20100801/ctry=US/part-00020

#### 创建桶表
`create table table_name() clustered by(col_0) into bucket_num buckets;`  
创建表，按照col_0分桶，有bucket_num个桶。
`set hive.enforce.bucketing = true;` 强制桶的个数和表定义相同，否则实际桶的个数和reducer一样。

#### 插入数据
由文件导入数据时，需要一种中间表，详细看下面‘文件导入数据’小节。

#### 数据查询
`select * from table_name tablesample(bucket x out of y on col_0);`
x:从第x桶开始抽取数据。
y:是总桶数的因数或倍数。 从x开始，分隔y个桶取数。
col_0:分桶的列。
eg：共有4个桶，y=2，x=2，会取第2、4个桶的数据；按照col_0分桶，会取col_0列的哈希值除以4余数是1、3的列。

总结自：https://www.cnblogs.com/MrFee/p/hive_bucket.html
https://blog.csdn.net/m0_37534613/article/details/55258928
### 建表
![](1.jpeg)
* PARTITIONED 表示的是分区，不同的分区会以文件夹的形式存在，在查询的时候指定分区查询将会大大加快查询的时间。
* CLUSTERED表示的是按照某列聚类，例如在插入数据中有两项“张三，数学”和“张三，英语”，若是CLUSTERED BY name，则只会有一项，“张三，(数学，英语)”，这个机制也是为了加快查询的操作。
* STORED是指定排序的形式，是降序还是升序。
* BUCKETS是指定了分桶的信息，这在后面会单独列出来，在这里还不会涉及到。
* ROW FORMAT是指定了行格式字段，如行、列的分隔符，`ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LINES TERMINATED BY '\n'`
* STORED AS是指定文件的存储格式。Hive中基本提供两种文件格式：SEQUENCEFILE和TEXTFILE，序列文件是一种压缩的格式，通常可以提供更高的性能，默认是TEXTFILE。
* LOCATION指的是在HDFS上存储的位置。

`create [external] table table_name1 like table_name2 [location hdfs_path]` 创建一个和表2结构一样的表

### (内部)表
表其实就是hdfs目录
Hive中的表和关系型数据库中的表在概念上很类似，每个表在HDFS中都有相应的目录用来存储表的数据，这个目录可以通过${HIVE_HOME}/conf/hive-site.xml配置文件中的hive.metastore.warehouse.dir属性来配置，这个属性默认的值是/user/hive/warehouse（这个目录在HDFS上），我们可以根据实际的情况来修改这个配置。
如果我有一个表wyp在cl库中，那么在HDFS中会创建/user/hive/warehouse/cl.db/wyp目录（这里假定hive.metastore.warehouse.dir配置为/user/hive/warehouse）；wyp表所有的数据都存放在这个目录中。这个例外是外部表。

参考：https://www.jianshu.com/p/dd97e0b2d2cf
### 外部表
数据源不在我们这里，由别人提供，或者其他工具提供。
指向已经在 HDFS 中存在的数据，可以创建 Partition。它和 Table 在元数据的组织上是相同的，而实际数据的存储则有较大的差异。
### (内部表)/外部表 区别
外部表在建表时多了‘EXTERNAL’： CREATE EXTERNAL TABLE
（1）、在导入数据到外部表，数据并没有移动到自己的数据仓库目录下，也就是说外部表中的数据并不是由它自己来管理的！而表则不一样；
（2）、在删除表的时候，Hive将会把属于表的元数据和数据全部删掉；而删除外部表的时候，Hive仅仅删除外部表的元数据，数据是不会删除的！
（3）、表有创建过程和数据加载过程（这两个过程可以在同一个语句中完成），在加载数据的过程中，实际数据会被移动到数据仓库目录中；之后对数据对访问将会直接在数据仓库目录中完成。删除表时，表中的数据和元数据将会被同时删除。
外部表只有一个过程，加载数据和创建表同时完成（CREATE EXTERNAL TABLE ……LOCATION），实际数据是存储在 LOCATION 后面指定的 HDFS 路径中，并不会移动到数据仓库目录中。

外部表适用场景：源表，需要定期将外部数据映射到表中。
使用场景例子：
每天将收集到的网站日志定期流入HDFS文本文件，一天一个目录；
在Hive中建立外部表作为源表，通过添加分区的方式，将每天HDFS上的原始日志映射到外部表的天分区中；
在外部表（原始日志表）的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过SELECT+INSERT进入内部表。


### 修改表
所有通过`alter`，修改的只是表的元数据，表里存的数据并不会改变。

#### 改变location
通过修改表DDL：`alter table t_m_cc set location 'hdfs://heracles/user/video-mvc/hive/warehouse/t_m_cc'`
直接修改hive 的meta info: `update DBS set DB_LOCATION_URI = replace(DB_LOCATION_URI,"oldpath","newpath")`
                        `update SDS  set location =replace(location,'oldpath,'newpath')`

#### 修改表名
`alter table old_name rename to new_name` 改表名

#### 修改列
```mysql
alter table table_name change column old_field_name new_field_name field_type
comment '...'
after field_name;
```
修改列名、注释、位置。如果要挪到第一个位置，只需要用`first`代替`after field_name`。
`alter table table_name add columns(..., ...)`添加新的字段。
`alter table table replace columns (..., ..., ...);`删除/替换列

#### 修改表属性
`alter table table_name set tblproperties(...)` 可以增加新的表属性，或者修改已经存在的属性，但是无法删除属性

### 自定义表的存储格式
`inputformat`对象将输入流分割成记录；`outputformat`对象将记录格式化为输出流（如查询的输出结果）；一个SerDe在读数据时将记录解析列，在写数据时将列编码成记录。
SerDe决定了记录是如何分解成字段的（反序列化过程），以及字段是如何写入到存储中的（序列化过程）。

### 集合数据类型
array、map、struct三种。好处是处理p/t级数据时，减少寻址，快。坏处是增大数据冗余等。

### 时间类型
`Timestamps`类型可以是
- （1）以秒为单位的整数；
- （2）带精度的浮点数，最大精确到小数点后9位，纳秒级；
- （3）java.sql.Timestamp格式的字符串 YYYY-MM-DD hh:mm:ss.fffffffff

`Date` 只支持YYYY-MM-DD格式的日期，其余写法都是错误的，如需带上时分秒，需使用timestamp。

### if
If 函数语法: if(boolean testCondition, T valueTrue, T valueFalseOrNull)
返回值: T
说明:  当条件testCondition为TRUE时，返回valueTrue；否则返回valueFalseOrNull
举例：
```mysql
hive> select if(1=2,100,200) from dual;
200
```

### 执行外部命令
hadoop命令：
把命令行里的hadoop去掉。
如直接执行`dfs -ls ...;`此种方法相叫hadoop的命令更为高效，hadoop是新开一个jvm线程执行，前者在当前线程执行。

其他命令，以!开始，以;结束，不能用管道、文件补全、用户交互等操作。
如`!echo 'li';`

### NVL
`NVL( str, replace_with)`  
str为NULL, 则NVL函数返replace_with值，否则返str值

### CONCAT(str1, str2,...)
连接字符串，如果参数中有null，返回结果也会是null，因此可以结合上面的方面使用。

### CONCAT_WS(separator, str1, str2,...)
它是一个特殊形式的 CONCAT()。第一个参数是剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间
这个函数会跳过分隔符参数后的任何 NULL 和空字符串，但是**跳过空字符串后还是会有多余的分隔符存在**（非常鸡肋啊）。

### collect_set()
是 Hive 内置的一个聚合函数, 它返回一个消除了重复元素的对象集合, 其返回值类型是 array 。
把group by值一样的分组由列变成行，即变成数组，可以用下标访问。
collect_set()方法把group by一样的组里的数据组成一个数组。数组从0开始，如果直接select数组，是[item1, ..., itemn]的格式。
如`select collect_set(uname) unames ....group by uid`，把同一个uid的uname组成数组， 通过别名unames[ind]访问数据。
`concat_ws(',',collect_set(cast(col_0 as string))) ` 两个一起使用把列变成由逗号分割的行。

### explode
上面的collect_set是把列变成行，explode是把行变成列。
`explode(array)` 把数组里的数据变成列形式。经常与split一起用。
例如，在wordcount中有`explode(split(line, ' '))` 或`explode(split(line, '\\s'))`

### 【is null】 = 【 = null】？、【is not null】 = 【 <> null】？
hive 里（包括IF函数与Where条件里）判断是否为NULL要用 is null或 is not null ，不能使用 <> null 或 = null（虽然不报错）

### size()
size()方法返回数组的长度。

### insert
1.insert into是增加数据
2.insert overwrite是删除原有数据然后在新增数据，如果有分区那么只会删除指定分区数据，其他分区数据不受影响

### rand
语法: rand(),rand(int seed)
返回值: double
说明:返回一个0到1范围内的随机数。如果指定种子seed，则会等到一个稳定的随机数序列

### cast
作用：转换
格式 cast(col as type)

### binary(string|binary)
将输入的值转换成二进制  

### base64(binary bin)
将二进制bin转换成64位的字符串

###  设置作业优先级
mapred.job.priority=VERY_HIGH | HIGH | NORMAL | LOW | VERY_LOW

### row_num() over (...)。
从1开始，为每个分组的每条记录返回一个数字。
1例如，`ROW_NUMBER() OVER (ORDER BY xlh DESC)` 是先按照xlh列降序，再为降序以后的每条记录返回一个序号。
2例
数据库中有数据
```
empid       deptid      salary
----------- ----------- ---------------------------------------
1           10          5500.00
2           10          4500.00
3           20          1900.00
4           20          4800.00
5           40          6500.00
6           40          14500.00
7           40          44500.00
8           50          6500.00
9           50          7500.00
```
需求`根据部门分组，显示每个部门的工资等级`
sql：`SELECT *, Row_Number() OVER (partition by deptid ORDER BY salary desc) rank FROM employee`
结果：
```
empid       deptid      salary                                  rank
----------- ----------- --------------------------------------- --------------------
1           10          5500.00                                 1
2           10          4500.00                                 2
4           20          4800.00                                 1
3           20          1900.00                                 2
7           40          44500.00                                1
6           40          14500.00                                2
5           40          6500.00                                 3
9           50          7500.00                                 1
8           50          6500.00                                 2
```
例子参考：https://blog.csdn.net/biaorger/article/details/38523527
row_number()另一作用可以用来去除重复：先按分组字段分区，再通过 rownum = 1过滤即可。另外，去重还可以借助于group by。

### partition by与group by 的区别
后者是经典的使用，是对检索结果的保留行进行单纯分组，如果有sum函数，就是先分组再对每个分组求和；
前者类似虽然也具有分组功能，但同时也具有其他的功能，如果有sum函数，是先分组，再累加，会把分组里累加的过程输出。
```
表：
B  C  D  
02 02 1
02 02 13

select b,c,sum(d) e from a group by b,c;   
结果：
B   C  E  
02 02 13
SELECT b, c, d, SUM(d) OVER(PARTITION BY b,c ORDER BY d) e FROM a;  
结果：
B C E  
02 02 1  
02 02 13
```
从上面的例子中可以看到第二条语句的累加过程

**_hive中group by和mysql不同。mysql可以接受select处理后的别名作为group by，hive的group by不能接受。**_

### ORDER BY、SORT BY
ORDER BY为全局排序，会将所有数据送到同一个Reducer中后再对所有数据进行排序，对于大数据会很慢，谨慎使用
SORT BY为局部排序，只会在每一个Reducer中对数据进行排序，在每个Reducer输出是有序的，但并非全局排序（每个reducer出来的数据是有序的，但是不能保证所有的数据是有序的——即文件(分区)之间无序，除非只有一个reducer）
DISTRIBUTE BY 是控制map的输出被送到哪个reducer端进行汇总计算。注：HIVE reducer分区个数由mapreduce.job.reduces来决定，该选项只决定使用哪些字段做为分区依据，如果没通过DISTRIBUTE BY指定分区字段，则默认将整个文本行做为分区依据。分区算法默认是HASH，也可以自己实现。
注：这里DISTRIBUTE BY讲的分区概念是指Hadoop里的，而非我们HIVE数据文本存储分区。Hadoop里的Partition主要作用就是将map的结果发送到相应的reduce，默认使用HASH算法，不过可以重写

### find_in_set
集合查找函数: find_in_set
语法: find_in_set(string str, string strList) 
返回值: int
说明: 返回str在strlist第一次出现的位置，strlist是用逗号分割的字符串。如果没有找该str字符，则返回0
例子：`select find_in_set('de','ef,ab,de');` 返回3

### hive的默认数据分隔符^A
hive的默认数据分隔符是\001,也就是^A ，属于不可见字符。

最简单的方法就是用sed（**_注意这个^A是按CTRL+V+A打出来的，或者按下crtl+v然后再按下crtl+a就会出来/tmp/out目录(\001)_**，直接输入的^A是不行的。）
**_也不能通过复制粘贴的方式。前一个地方用的CTRL+V+A，复制粘贴后就失效，要重新CTRL+V+A。_**
例：sed -i 's/^A/|/g' 000000_0

来自网络：
在python中可以使用line.split('\x01')来进行切分，也可以使用line.split('\001')，注意其中是单引号
在java中可以使用split("\\u0001")来进行切分
### hive默认记录、字段分割符

|分隔符|描述|
|:---:|:---|
|\n|换行符。默认记录分隔符，一行一个记录。|
|^A|用于分割字段。可用八进制\001表示(看上节)|
|^B|用与分割array、struct、map的key-value对。可用八进制\002表示|
|^C|用于分割map的key、value对。可用八进制\003表示|

ROW FORMAT DELIMITED必须写在其他字段前，除了stored as。

### decimal
DECIMAL Hive 0.11.0引入，Hive 0.13.0开始，用户可以使用DECIMAL(precision, scale) 语法在创建表时来定义Decimal数据类型的precision和scale。
如果未指定precision，则默认为10。如果未指定scale，它将默认为0（无小数位）。
**_曾遇到这样的问题，创建的外部表没有指定精度，外部表指定的内部表有指定精度，从外部表查数据时仍然截断了小数部分。_**

### export LC_ALL=en_US.UTF-8
`export LC_ALL=en_US.UTF-8` 解决hive客户端调用脚本中文问题。

https://perlgeek.de/en/article/set-up-a-clean-utf8-environment

### get_json_object
`get_json_object(json_string,’$.str’)` 得到json字符串json_string的$.str节点的值，$指根节点。

### 导出数据到本地
hive的-e和-f参数可以用来导出数据。
-e 表示后面直接接带双引号的sql语句；而-f是接一个文件，文件的内容为sql语句。
（1）`hive -e "use test; select * from student where sex = '男'" > /tmp/output.txt`
（2）`insert overwrite local directory "/tmp/out" select cno,avg(grade) from sc group by(cno);`
（3）`insert overwrite local directory "/tmp/out" row format delimited fields terminated by ' ' select cno,avg(grade) from sc group by(cno);`

（2）也可以作为（1）中-e的参数执行。
（2）这条HQL的执行需要启用Mapreduce完成，运行完这条语句之后，将会在本地文件系统的/tmp/out目录下生成文件，这个文件是Reduce产生的结果（这里生成的文件名是000000_0），数据的分割使用的就是上面提到的^A
（3）通过加入`row format delimited fields terminated by ' ' `使的数据的分割是空格，而不是^A.
（1）中会直接保存成本地文件，把数据直接保存在/tmp/output.txt中，数据默认由空格分割。
（2）这条HQL的‘local’去掉，数据会被保存在hdfs系统的/tmp/out目录下。
（2）不能使用`insert into`或者`insert local`

### 导出数据到表
把表2的数据导出到表1：
(1)`insert into table_name1(...) select ... from table_name2`
(2)`insert overwrite table_name1(...) select ... from table_name2`
select 部分不能用括号，否则会被认为是表1的字段；
(...)中是表1的字段，可以省略； `select ...` 可以用`select *` 代替；
(1)是直接导入，(2)是覆盖原来数据导入。

导入到分区表：
(1)`insert into table_name1 partition(dt='2018-03-11') select ... from table_name2`
(2)`set hive.exec.dynamic.partition.mode=nonstrict;insert into table_name1 partition(dt) select ... from table table_name2`

同样可以把`into`换成`overwrite table`以达到覆盖的效果。注意：只会覆盖table_name2中存在的对应分区，table_name1中已经存在的分区，table_name2中没有是不会进行覆盖。 即，覆盖只是覆盖分区里的数据数据、追加分区，原分区不变。
(1)是导入一个分区的数据 `select ...`部分不用带dt(分区)的值。注意，如果表2也是分区表，此时用`select *`查出来的数据有分区字段
(2)是导入多个分区的表，执行前需要`set hive.exec.dynamic.partition.mode=nonstrict;`，因为严格模式下，不允许所有的分区都被动态指定，目的是为了防止生成太多的目录.此时`select ...`必须有dt分区的字段。
(2)是动态分区，不指定分区，一次可以导入多个分区。

### 文件导入数据到表
`load data [local] inpath 'file1.txt' [overwrite] into table table_name [partition(partcol=val)]`
`local` 决定文件是来自本地还是hdfs。
`overwrite` 决定是否要覆盖。
`load`命令不支持动态分区，必须指定分区。(可以把数据先转到非分区表，再利用上面小节“导出数据到表”的方法把非分区表的数据导入到分区表)。不指定分区，会报错`FAILED: SemanticException org.apache.hadoop.hive.ql.metadata.HiveException: MetaException(message:Invalid partition key & values; keys [dt, ], values [])`
load不能加载桶表数据，只能从另一张表加载数据。(和动态分区的解决方案一样，建一个中间表作为过渡表)。

### 同时插入多个表
```mysql
from (select ... from test limit 1) t
insert into table test1 select ...
insert into table test3 select ...
```
从test中查数同时插入到test1、test3。每个select都必须存在，可以用*

### 自定义UDF
网上介绍了四中方法。只验证过第一种。
方法（1）最常用也最不被喜欢的方法。
```mysql
add jar testUDF-0.0.1-SNAPSHOT.jar;
create temporary function zodiac as "com.hive.udf.UDFZodiacSign";
```
之后就可以在sql里直接使用`zodiac()`。但是这种方法只存在在当前会话中。
每次会话都要重新add、create。（下面的.hiverc文件可以解决每次都要add、create问题）

其他方法：https://www.cnblogs.com/chushiyaoyue/p/6632090.html?utm_source=itdadao&utm_medium=referral
### .hiverc文件
网上说在`${HIVE_HOME}/bin`目录下（我目前遇到别人部署的hive是在用户目录下）
（`ls -a`命令查看隐藏文件）
它是在hive启动的时候被调用，可以在里面定义常用的参数。
写到这个是因为，还可以把上面加载udf的最常用最不被喜欢的第一种方法的add、create语句写到.hiverc文件里，这样每次启动hive时都默认加载了udf方法。

### -i
-i 参数可以指定一个hive启动就被调用的文件。对，默认就是上面的.hiverc文件！

### set变量

|命名空间|使用权限|描述|
|:------|:---|:-------|
|hivevar|可读可写|hive 0.18.0 版本及之后。用户自定义变量|
|hiveconf|可读可写|hive相关的配置属性|
|system|可读可写|java相关的配置属性|
|env|只可读|shell环境定义的环境变量|

hivevar例子：
```
set hivevar:dd='aa';
select ${hivevar:dd}
```
hivevar的前缀可以省略，但是可能会找不到变量，不建议省略。
system、env的前缀不能省。

上面是在hive的终端里，另一种是在shell里使用。在实践中使用时，`create_table.sql`需要用`"${hivevar:dd}"`,即需要双引号。
另外`-hivevar`可用`--define`
`hive -hivevar dd='aa' -f ./create_table.sql`


直接`set`命令可以看到所有变量值。
`set`单个参数，可以看见这个参数的值。

### set hiveconf
Hive相关的配置属性总结
`set hive.cli.print.current.db=true;` 在cli hive提示符后显示当前数据库。
`set hive.cli.print.header=true;` 显示表头。select时会显示对应字段。
`set hive.mapred.mode=strict;` 如果对分区表查询，且没有在where中对分区字段进行限制，报错`FAILED: SemanticException [Error 10041]: No partition predicate found for Alias "test_part" Table "test_part"`；对应还有`nonstrict`模式。

压缩：
1、`mapreduce.map.output.compress`：map输出结果是否压缩
   `mapreduce.map.output.compress.codec`
2、`mapreduce.output.fileoutputformat.compress`：job输出结果是否压缩
   `mapreduce.output.fileoutputformat.compress.type`
   `mapreduce.output.fileoutputformat.compress.codec`  
eg,`set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec;`  设置job输出结果压缩成gz。


### tblproperties
`tblproperties` 主要的作用是以键值对的格式为表增加额外的文档说明。
（hive和像DymamoDB这样的数据库集成时，`tblproperties` 还有用作数据库连接的必要的元数据信息）
Hive会自动增加两个表属性：last_modified_by，保存最后修改这个表的用户的用户名；last_modified_time，保存最后一次修改的时间秒，但是如果用户没有手动定义任何的文档说明，这两个属性还是不会自动添加的。
`show tblproperties table_name` 查看表的`tblproperties`信息

### 命令
`describe [extended/formatted] table_name` 查看表信息，类似`desc`  可选的`[extended]`可以看到更详细的信息，`formatted`看更多信息，可读性强
`describe database [extended/formatted] database1` 查看库信息，可以看到库地址
`drop database database1 cascade/restrict`  库不为空时，一般不允许直接删除，`cascade`保证可以删除，默认是restrict
`show tables in data_base` 在别的库里查询库`data_base`所有的表

### 读时模式
hive是“读时模式”，对于存储文件的完整性、数据的格式是否和表匹配性等方面都没有支配能力。
只有在读数据时才会尽量的把hdfs的文件和表字段进行匹配。
我遇到的一个典型例子：hdfs文件里数据是3.5，hive表对应字段类型是`decimal`，这样导致读出来的数是4.（decimal没有指定小数精度时，默认是没有小数位）

### 易错
hive cli 有tab补全的功能，因此，如果hql里有tab时，会出现`Display all 479 possibilities? (y or n)`的询问。

### 读orc格式数据
hive-0.11版本中的使用方法为：`hive --orcfiledump <location-of-orc-file>`，其他版本的使用方法可以去官方文档中查找。

出错场景：用hive读orc时出错，使用hive版本是0.11，参考hive官网http://www.bieryun.com/2447.html 讲解Hive version 0.11 through 0.14都可以使用上面的方法读orc文件。
之后使用1.2.2的版本使用上面命令不会出错，可以select读数据。

使用时出错：
```
Exception in thread "main" com.google.protobuf.InvalidProtocolBufferException: Message missing required fields: columns[1].kind, columns[2].kind, columns[3].kind
at com.google.protobuf.UninitializedMessageException.asInvalidProtocolBufferException(UninitializedMessageException.java:81)
at org.apache.hadoop.hive.ql.io.orc.OrcProto$StripeFooter$Builder.buildParsed(OrcProto.java:5908)
at org.apache.hadoop.hive.ql.io.orc.OrcProto$StripeFooter$Builder.access$10700(OrcProto.java:5834)
at org.apache.hadoop.hive.ql.io.orc.OrcProto$StripeFooter.parseFrom(OrcProto.java:5779)
at org.apache.hadoop.hive.ql.io.orc.RecordReaderImpl.readStripeFooter(RecordReaderImpl.java:1108)
at org.apache.hadoop.hive.ql.io.orc.RecordReaderImpl.readStripe(RecordReaderImpl.java:1114)
at org.apache.hadoop.hive.ql.io.orc.RecordReaderImpl.<init>(RecordReaderImpl.java:94)
at org.apache.hadoop.hive.ql.io.orc.ReaderImpl.rows(ReaderImpl.java:242)
at org.apache.hadoop.hive.ql.io.orc.ReaderImpl.rows(ReaderImpl.java:236)
at org.apache.hadoop.hive.ql.io.orc.FileDump.main(FileDump.java:37)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:606)
at org.apache.hadoop.util.RunJar.main(RunJar.java:212)
```
分析：orc的表是别人建的，无法确定当初建表的hive的版本。google.protobuf是一个作为协议的包，类似于序列化。因此猜测是不同版本的hive的orc不一样导致压缩数据和解压数据无法连起来。

### 参考
参考：http://www.cnblogs.com/smartloli/p/4288493.html
https://www.jianshu.com/p/bd7820161a49?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation
更多hive看：https://www.iteblog.com/
hive 安装：https://www.jianshu.com/p/6108e0aed204
hive字符串：https://www.iteblog.com/archives/1639.html
hadoop常用命令：https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html#test
hive 常用总结（写的很好）：https://www.cnblogs.com/jiangzhengjun/p/6349226.html
mp调优：https://www.cnblogs.com/sunxucool/p/4459006.html
函数（时间、字符串、数值）：https://blog.csdn.net/duan19056/article/details/17758819