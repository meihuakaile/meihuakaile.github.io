---
title: hive笔记
tags:
  - hadoop
  - hive
categories:
  - hadoop
copyright: true
date: 2018-04-19 00:00:00
---

架构在Hadoop之上，提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。
hive是一个数据仓库工具，作用是可以将结构化的数据文件映射为一张数据库表，并提供简单查询功能，可以将sql语句转化为Mapreduce任务进行，是在Hadoop上的数据库基础架构。
**Hive 不是一个关系数据库/实时查询和行级更新的语言.**

Hadoop是一个开源框架来存储和处理大型数据在分布式环境中。它包含两个模块，一个是MapReduce，另外一个是Hadoop分布式文件系统（HDFS）:
* **MapReduce**：它是一种并行编程模型在大型集群普通硬件可用于处理大型结构化，半结构化和非结构化数据。
* **HDFS**：Hadoop分布式文件系统是Hadoop的框架的一部分，用于存储和处理数据集。它提供了一个容错文件系统在普通硬件上运行。

hive的安装需要安装MySQL,因为hive 默认的数据库是Derby数据库，其与MySQL数据库比较存在缺陷。

### CONCAT_WS(separator, str1, str2,...)

它是一个特殊形式的 CONCAT()。第一个参数是剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间
### collect_set（）

把group by值一样的分组由列变成行，即变成数组，可以用下标访问。

concat_ws(',',collect_set(cast(col_0 as string)))  两个一起使用把列变成由逗号分割的行。

### 建表
![](1.jpeg)
* PARTITIONED 表示的是分区，不同的分区会以文件夹的形式存在，在查询的时候指定分区查询将会大大加快查询的时间。
* CLUSTERED表示的是按照某列聚类，例如在插入数据中有两项“张三，数学”和“张三，英语”，若是CLUSTERED BY name，则只会有一项，“张三，(数学，英语)”，这个机制也是为了加快查询的操作。
* STORED是指定排序的形式，是降序还是升序。
* BUCKETS是指定了分桶的信息，这在后面会单独列出来，在这里还不会涉及到。
* ROW FORMAT是指定了行的参数。还要指定列的信息，如ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LINES TERMINATED BY '\n'
* STORED AS是指定文件的存储格式。Hive中基本提供两种文件格式：SEQUENCEFILE和TEXTFILE，序列文件是一种压缩的格式，通常可以提供更高的性能。
* LOCATION指的是在HDFS上存储的位置。

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

### 分区
hive引入partition和bucket的概念，中文翻译分别为分区和桶，这两个概念都是把数据划分成块，分区是粗粒度的划分桶是细粒度的划分，这样做为了可以让查询发生在小范围的数据上以提高效率。
在Hive Select查询中一般会扫描整个表内容，会消耗很多时间做没必要的工作。有时候只需要扫描表中关心的一部分数据，因此建表时引入了partition概念。
分区表指的是在创建表时指定的partition的分区空间。如果需要创建有分区的表，需要在create表的时候调用可选参数partitioned by。
一个表可以拥有一个或者多个分区，每个分区以文件夹的形式单独存在表文件夹的目录下。
分区是以字段的形式在表结构中存在，通过describe table命令可以查看到字段存在，但是该字段不存放实际的数据内容，仅仅是分区的表示。

### 单分区/多分区
分区建表分为2种，一种是单分区，也就是说在表文件夹目录下只有一级文件夹目录。另外一种是多分区，表文件夹下出现多文件夹嵌套模式。

a、单分区建表语句：create table day_table (id int, content string) partitioned by (dt string);单分区表，按天分区，在表结构中存在id，content，dt三列。

b、双分区建表语句：create table day_hour_table (id int, content string) partitioned by (dt string, hour string);双分区表，按天和小时分区，在表结构中新增加了dt和hour两列。

### row_num()。

作用是按指定的列进行分组生成行序列。在ROW_NUMBER(a,b) 时，若两条记录的a，b列相同，则行序列+1，否则重新计数。
例子参考：https://blog.csdn.net/qq_31573519/article/details/78586205

### hive的默认数据分隔符^A
hive的默认数据分隔符是\001,也就是^A ，属于不可见字符。

最简单的方法就是用sed（**_注意这个^A是按CTRL+V+A打出来的，或者按下crtl+v然后再按下crtl+a就会出来^A(\001)_**，直接输入的^A是不行的。）
**_也不能通过复制粘贴的方式。前一个地方用的CTRL+V+A，复制粘贴后就失效，要重新CTRL+V+A。_**
例：sed -i 's/^A/|/g' 000000_0

### hive在hdfs上的文件结构
```
　　    数据仓库的位置                数据库目录           表目录          表的数据文件 
　　/user/hive/warehouse             /test.db             /row_table       /hive_test.txt 
```
default默认的数据库：指的就是这个/user/hive/warehouse路径
参考：https://www.cnblogs.com/xningge/p/8439970.html

### 内部表/外部表
外部表在建表时多了‘EXTERNAL’： CREATE EXTERNAL TABLE
（1）、在导入数据到外部表，数据并没有移动到自己的数据仓库目录下，也就是说外部表中的数据并不是由它自己来管理的！而表则不一样；
（2）、在删除表的时候，Hive将会把属于表的元数据和数据全部删掉；而删除外部表的时候，Hive仅仅删除外部表的元数据，数据是不会删除的！

### 读orc格式数据
hive-0.11版本中的使用方法为：hive --orcfiledump <location-of-orc-file>，其他版本的使用方法可以去官方文档中查找。

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