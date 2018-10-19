---
title: 'hive出错'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
### 易错
hive cli 有tab补全的功能，因此，如果hql里有tab时，会出现`Display all 479 possibilities? (y or n)`的询问。
`left/right join on where`时注意条件放在on之后还是where之后，结果会不同。 

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

### 使用SQL2011保留字出错
报错：`Failed to recognize predicate 'xxx'. Failed rule: 'identifier' in column specification`
原因：我的HQL中出现`row`作为字段名。在Hive1.2.0版本开始增加了如下配置选项，默认值为true：
`hive.support.sql11.reserved.keywords`该选项的目的是：是否启用对SQL2011保留关键字的支持。 启用后，将支持部分SQL2011保留关键字。
解决方法（1）：放弃`row`，换一个关键字。
解决方法（2）：弃用对保留关键字的支持。`set hive.support.sql11.reserved.keywords = false ;`
解决方法（3）：弃用对保留关键字的支持。在conf下的hive-site.xml配置文件中修改配置选项：
```xml
<property>
    <name>hive.support.sql11.reserved.keywords</name>
    <value>false</value>
</property>
```
总结自：https://blog.csdn.net/SJF0115/article/details/73244762

### join on中比较大小报错
报错：`Both left and right aliases encountered in JOIN '1'`
原因：两个表join的时候，不支持两个表的字段 非相等 操作。Hive 不支持所有非等值的连接，因为非等值连接非常难转化到 map/reduce 任务。
解决：可以把不相等条件拿到 where语句中。

### 使用distinct报错
报错`HIVE: cannot recognize input near 'distinct' '('`
原因：hive的语法中select的字段分两种，all和distinct，默认是all，加distinct的字段必须放在所有查询字段的最前面（mysql也是）。
总结自：https://stackoverflow.com/questions/38794766/hive-cannot-recognize-input-near-distinct

### memory limits
报错：`is running beyond physical memory limits. Current usage: 2.0 GB of 2 GB physical memory used; 9.9 GB of 40 GB virtual memory used.`
分析：`2.0 GB of 2 GB physical memory used` 物理内存溢出 OOM为out of memory
解决：设置mapper和reducer 物理内存和虚拟内存
`set mapreduce.map.memory.mb=10240;`  container的内存 运行mapper的容器的物理内存，1024M = 1G
`set mapreduce.map.java.opts='-Xmx7680M';`  jvm堆内存
`set mapreduce.reduce.memory.mb=10240;`
`set mapreduce.reduce.java.opts='-Xmx7680M';`
在yarn container这种模式下，map/reduce task是运行在Container之中的，所以上面提到的mapreduce.map(reduce).memory.mb大小**_都大于_**mapreduce.map(reduce).java.opts值的大小。mapreduce.{map|reduce}.java.opts能够通过Xmx设置JVM最大的heap的使用，**_一般设置为0.75倍的memory.mb，因为需要为java code等预留些空间_**。

来源于网络：虚拟内存的计算由 物理内存 和 yarn-site.xml中的yarn.nodemanager.vmem-pmem-ratio制定。
`yarn.nodemanager.vmem-pmem-ratio`是 一个比例，默认是2.1   虚拟内存 = 物理内存 × 这个比例 
yarn.nodemanager.vmem-pmem-ratio 的比率，默认是2.1.这个比率的控制影响着虚拟内存的使用，当yarn计算出来的虚拟内存，比在mapred-site.xml里的mapreduce.map.memory.mb或mapreduce.reduce.memory.mb的2.1倍还要多时，会被kill掉。

参考：https://blog.csdn.net/yisun123456/article/details/81327372
### join出错
报错`FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.mr.MapredLocalTask`
一个join操作导致的，原因不明。
网上查到解决办法：`SET hive.auto.convert.join=false;` 此操作的原理看上面“join原理、调优”章节里的“map端的join”。
参考：https://www.cnblogs.com/MOBIN/p/5702580.html

### jobname过长
报错`hive java.io.IOException: Could not find status of job:job_15416269223_3720383`
hive的运行原理：当任务执行结束后，对于每一个jobid，会根据一定规则，生成两个文件，一个是*.jhist,另一个是*.conf.这两个文件记录了这个job的所有执行信息，这两个文件是要写入到jobhistory所监控的路径的。
这部分是根据hive的jobname来决定的，默认是从hql中的开头和结尾截取一部分，如果sql开头或结尾有中文注释的话，会被截取进来，并进行url编码。导致作业的信息名变的非常长，超过了namenode所允许的最大的文件命名长度。导致任务无法写入historyserver。hive在historyserver无法获得这个job的状态，报开头的错误。
这里提供一个简单的解决办法：`set  hive.jobname.length=10;`

原文：https://blog.csdn.net/zhoudetiankong/article/details/52126760

### hive启动出错 $HADOOP_HOME
`Cannot find hadoop installation: $HADOOP_HOME or $HADOOP_PREFIX must be set or hadoop must be in the`
解决办法：
在hive-env.sh（conf目录下）中添加hadoop的目录：
`export HADOOP_HOME=/home/work/hadoop/hadoop-2.6.0/`
之后`source hive-env.sh` 就ok了

### 配置spark后hive启动出错
`ls: cannot access /usr/local/spark/lib/spark-assembly-*.jar: No such file or directory`
https://blog.csdn.net/Gpwner/article/details/73457108

### 缺分号
`Hive Warning: Value had a \n character in it`
遇到这个错误一般是hive query的格式问题， 哪里缺了个分号  `；`
我这次是几个set语句后面忘记加分号了。