---
title: 'hive-set设置总结'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
直接`set`命令可以看到所有变量值。
`set`单个参数，可以看见这个参数的值。

### 常用hiveconf
Hive相关的配置属性总结
`set hive.cli.print.current.db=true;` 在cli hive提示符后显示当前数据库。
`set hive.cli.print.header=true;` 显示表头。select时会显示对应字段。
`set hive.mapred.mode=strict;` 防止笛卡儿积的执行;如果对分区表查询，且没有在where中对分区字段进行限制，报错`FAILED: SemanticException [Error 10041]: No partition predicate found for Alias "test_part" Table "test_part"`；对应还有`nonstrict`模式（默认模式）。
`hive.support.sql11.reserved.keywords`该选项的目的是：是否启用对SQL2011保留关键字的支持。 启用后，将支持部分SQL2011保留关键字。

###  设置作业优先级
`mapred.job.priority=VERY_HIGH | HIGH | NORMAL | LOW | VERY_LOW`
### 手动指定队列
```bash
set mapreduce.job.queuename=hive;
```
### 手动指定job name
```bash
set mapreduce.job.name=uid_uname;
```
### 动态分区
```bash
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.max.dynamic.partitions=100;
set hive.exec.max.dynamic.partitions.pernode=100;
```

`hive.exec.dynamic.partition` 开启动态分区
`hive.exec.dynamic.partition.mode` 设置可以动态分区；因为严格模式下，不允许所有的分区都被动态指定。（详细使用看上面“导出数据到表”章节）
`hive.exec.max.dynamic.partitions` 默认是1000；在所有执行的MR节点上，一共可以创建最大动态分区数
`hive.exec.max.dynamic.partitions.pernode`  (上面参数也要加上)默认是100；在每个执行MR的节点上，最大可以创建多少个动态分区。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。

动态分区参考：http://lxw1234.com/archives/2015/06/286.htm

### 并发优化
```bash
set hive.exec.parallel=true;
set hive.exec.parallel.thread.number=8;
```
第一个参数：开启任务并行执行； 
第二个参数：同一个sql允许并行任务的最大线程数

job之间没有前后依赖的都可以并行执行。
### join/group by倾斜优化
```bash
set hive.map.aggr=true;
set hive.groupby.skewindata=true;
set hive.groupby.mapaggr.checkinterval=100000;
set hive.optimize.skewjoin=true;
set hive.skewjoin.key=100000;
```
参数解释：[hive数据倾斜](/2018/10/19/hive数据倾斜/)

### 解决小文件问题
详细[hive小文件合并](/2018/10/19/hive小文件合并/)
```bash
set hive.merge.mapfiles=true;
set hive.merge.mapredfiles=true;
set hive.merge.size.per.task=256*1000*1000;
set hive.merge.smallfiles.avgsize=16000000;
```
上面参数在 文件输出时合并。但是它们 和 压缩 并存时会失效，并对`orc`格式的表（orc本身就已经压缩）不起作用。

```bash
set hive.hadoop.supports.splittable.combineinputformat=true;
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
set mapred.max.split.size=2048000000;
set mapred.min.split.size.per.node=2048000000;
set mapred.min.split.size.per.rack=2048000000;
```
上面参数在 文件进入时合并文件，减少map个数。

```bash
set hive.exec.reducers.bytes.per.reducer=5120000000;
insert overwrite table test partition(dt)
select * from iteblog_tmp
DISTRIBUTE BY rand();
```
`DISTRIBUTE BY rand()` 强制产生reduce，`set hive.exec.reducers.bytes.per.reducer`控制reduce个数（reduce处理数据数量），两者一起使用控制小文件输出。

### 内存
```sql
set mapreduce.map.memory.mb=2048;
set mapred.child.map.java.opts='-Xmx2048M';
set mapreduce.map.java.opts='-Xmx2048M';  
set mapreduce.reduce.memory.mb=2048;
set mapred.child.reduce.java.opts='-Xmx2048m';
set mapreduce.reduce.java.opts='-Xmx2048M';
set yarn.app.mapreduce.am.resource.mb=3000;
set yarn.app.mapreduce.am.command-opts='-Xmx2048m';
```
`set mapreduce.map.memory.mb`  container的内存 运行mapper的容器的物理内存，1024M = 1G
`set mapreduce.map.java.opts`  jvm堆内存
`set yarn.app.mapreduce.am.resource.mb` app内存。am指 Yarn中AppMaster，针对MapReduce计算框架就是MR AppMaster，通过配置这两个选项，可以设定MR AppMaster使用的内存。  一般看hadoop日志时可以看到map/reduce，但是当没有map/reduce时就开始报`beyond memory limit`类似的错时，说明是am的内存不够。
在yarn container这种模式下，map/reduce task是运行在Container之中的，所以上面提到的mapreduce.map(reduce).memory.mb大小**_都大于_**mapreduce.map(reduce).java.opts值的大小。mapreduce.{map|reduce}.java.opts能够通过Xmx设置JVM最大的heap的使用，**_一般设置为0.75倍的memory.mb，因为需要为java code等预留些空间_**。

来源于网络：虚拟内存的计算由 物理内存 和 yarn-site.xml中的yarn.nodemanager.vmem-pmem-ratio制定。
`yarn.nodemanager.vmem-pmem-ratio`是 一个比例，默认是2.1   虚拟内存 = 物理内存 × 这个比例 
yarn.nodemanager.vmem-pmem-ratio 的比率，默认是2.1.这个比率的控制影响着虚拟内存的使用，当yarn计算出来的虚拟内存，比在mapred-site.xml里的mapreduce.map.memory.mb或mapreduce.reduce.memory.mb的2.1倍还要多时，会被kill掉。

参考：https://blog.csdn.net/yisun123456/article/details/81327372

### 压缩
```bash
set hive.exec.compress.output=true;
set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;
set mapred.output.compression.type=BLOCK;

set hive.exec.compress.intermediate=true;
set mapred.map.output.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;
```
前三个参数是输出压缩；
最后两个参数是map输出压缩。
详细[hive压缩](/2018/10/19/hive压缩/)

### reducer个数相关
`set mapreduce.job.reduces=15;` 指定reducer个数
`set hive.exec.reducers.bytes.per.reducer` 每个reduce任务处理的数据量，默认为1000^3=1G

### 推测式执行配置项
```bash
set mapred.map.tasks.speculative.execution=true;
set mapred.reduce.tasks.speculative.execution=true;
```
这是两个推测式执行的配置项,默认是true
所谓的推测执行，就是当所有task都开始运行之后，Job Tracker会统计所有任务的平均进度，如果某个task所在的task node机器配
置比较低或者CPU load很高（原因很多），导致任务执行比总体任务的平均执行要慢，此时Job Tracker会启动一个新的任务
（duplicate task），原有任务和新任务哪个先执行完就把另外一个kill掉，这也是我们经常在Job Tracker页面看到任务执行成功，但
是总有些任务被kill，就是这个原因。

### 关闭mapjoin
```bash
SET hive.auto.convert.join=false;
set hive.ignore.mapjoin.hint=false;
```

### mapreduce.task.io.sort.mb
`mapreduce.task.io.sort.mb` map shuffle时的内存  溢出

### 参考
https://blog.csdn.net/yycdaizi/article/details/43341239