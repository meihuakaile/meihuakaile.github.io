---
title: 'hive-set设置'
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
`set hive.exec.reducers.bytes.per.reducer` 每个reduce任务处理的数据量，默认为1000^3=1G

###  设置作业优先级
`mapred.job.priority=VERY_HIGH | HIGH | NORMAL | LOW | VERY_LOW`

### 动态分区
`set hive.exec.dynamic.partition=true;`
`set hive.exec.dynamic.partition.mode=nonstrict;` 设置可以动态分区；因为严格模式下，不允许所有的分区都被动态指定。（详细使用看上面“导出数据到表”章节）
`set hive.exec.max.dynamic.partitions=100;` 默认是1000；在所有执行的MR节点上，一共可以创建最大动态分区数
`set hive.exec.max.dynamic.partitions.pernode=100;`  (上面参数也要加上)默认是100；在每个执行MR的节点上，最大可以创建多少个动态分区。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。

动态分区参考：http://lxw1234.com/archives/2015/06/286.htm

### 并发优化
`set hive.exec.parallel=true;`  开启任务并行执行
`set hive.exec.parallel.thread.number=8;`  同一个sql允许并行任务的最大线程数

job之间没有前后依赖的都可以并行执行。
### 内存
```sql
set mapreduce.map.memory.mb=20480;
set mapreduce.map.java.opts='-Xmx20480M';  
set mapreduce.reduce.memory.mb=20480;
set mapreduce.reduce.java.opts='-Xmx20480M';
```
`set mapreduce.map.memory.mb`  container的内存 运行mapper的容器的物理内存，1024M = 1G
`set mapreduce.map.java.opts`  jvm堆内存
在yarn container这种模式下，map/reduce task是运行在Container之中的，所以上面提到的mapreduce.map(reduce).memory.mb大小**_都大于_**mapreduce.map(reduce).java.opts值的大小。mapreduce.{map|reduce}.java.opts能够通过Xmx设置JVM最大的heap的使用，**_一般设置为0.75倍的memory.mb，因为需要为java code等预留些空间_**。

来源于网络：虚拟内存的计算由 物理内存 和 yarn-site.xml中的yarn.nodemanager.vmem-pmem-ratio制定。
`yarn.nodemanager.vmem-pmem-ratio`是 一个比例，默认是2.1   虚拟内存 = 物理内存 × 这个比例 
yarn.nodemanager.vmem-pmem-ratio 的比率，默认是2.1.这个比率的控制影响着虚拟内存的使用，当yarn计算出来的虚拟内存，比在mapred-site.xml里的mapreduce.map.memory.mb或mapreduce.reduce.memory.mb的2.1倍还要多时，会被kill掉。

参考：https://blog.csdn.net/yisun123456/article/details/81327372

### 压缩

![](yasuo.png)
![](yasuo1.png)
`set hive.exec.compress.intermediate=true;` 中间数据map压缩，不影响最终结果。但是job中间数据输出要写在硬盘并通过网络传输到reduce，传送数据量变小，因为shuffle sort（混洗排序）数据被压缩了。
`set mapred.map.output.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;` 为中间数据配置压锁编解码器 ，通常配置Snappy更好。
压缩Map的输出，这样做有两个好处：
a)压缩是在内存中进行，所以写入map本地磁盘的数据就会变小，大大减少了本地IO次数
b) Reduce从每个map节点copy数据，也会明显降低网络传输的时间
注：数据序列化其实效果会更好，无论是磁盘IO还是数据大小，都会明显的降低。

`set hive.exec.compress.output=true;`  打开job最终输出压缩的开关，设置之后必须设置下面这行，否则还是没有压缩效果
`set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;`  设置压缩类型
`set mapred.output.compression.type=BLOCK;` 大文件压缩仍然会耗时，而且影响mapper并行（mapper并行和文件的个数有关），这个设置，使大的文件可以分割成小文件进行压缩

gzip是一种数据格式，默认且目前仅使用deflate算法压缩data部分；deflate是一种压缩算法。
**_gzip不支持切片，切片参数都不管用。如果要压缩成gzip格式，做好控制在170M，mr的效果是最好的。_**
这种处理文件压缩的能力并非是hive特有的，实际上，使用了hadoop的TextInputFormat进行处理，它可以识别后缀名是.deflate或.gz的压缩文件，并可以轻松处理。
hive无需关心底层文件是否是压缩的，以及如何压缩的。

https://m.2cto.com/kf/201611/566909.html
### 关于Strict Mode 

 Hive中的严格模式可以防止用户发出（可以有问题）的查询无意中造成不良的影响。 将hive.mapred.mode设置成strict可以禁止三种类型的查询：
 1）、在一个分区表上，如果没有在WHERE条件中指明具体的分区，那么这是不允许的，换句话说，不允许在分区表上全表扫描。这种限制的原因是分区表通常会持非常大的数据集并且可能数据增长迅速，对这样的一个大表做全表扫描会消耗大量资源，必须要再WHERE过滤条件中具体指明分区才可以执行成功的查询。
 2）、第二种是禁止执行有ORDER BY的排序要求但没有LIMIT语句的HiveQL查询。因为ORDER BY全局查询会导致有一个单一的reducer对所有的查询结果排序，如果对大数据集做排序，这将导致不可预期的执行时间，必须要加上limit条件才可以执行成功的查询。
 3）、第三种是禁止产生笛卡尔集(full Cartesian product)。在JION接连查询中没有ON连接key而通过WHERE条件语句会产生笛卡尔集，需要改为JOIN...ON语句。  

### reducer个数
`set mapreduce.job.reduces=15;` 指定reducer个数

### 推测式执行配置项
`mapred.map.tasks.speculative.execution=true`
`mapred.reduce.tasks.speculative.execution=true`
这是两个推测式执行的配置项,默认是true
所谓的推测执行，就是当所有task都开始运行之后，Job Tracker会统计所有任务的平均进度，如果某个task所在的task node机器配
置比较低或者CPU load很高（原因很多），导致任务执行比总体任务的平均执行要慢，此时Job Tracker会启动一个新的任务
（duplicate task），原有任务和新任务哪个先执行完就把另外一个kill掉，这也是我们经常在Job Tracker页面看到任务执行成功，但
是总有些任务被kill，就是这个原因。

### join只允许等值操作
```sql
SELECT c.*, t.* FROM c JOIN t 
ON (t.area1= c.cname OR t.area2 =c.cname OR t.area3 = c.cname)
WHERE t.time>='20140818' and t.time<='20140824' AND platform='pc'
GROUP BY t.time;
```
报错`FAILED: SemanticException [Error 10019]: Line 5:32 OR not supported in JOIN currently 'cname'`
hive 受限于 MapReduce 算法模型，只支持 equi-joins（等值 join），要实现上述的非等值 join，可以选择采用笛卡儿积（ full Cartesian product ）来实现。
笛卡尔积：
```sql
SELECT c.*, t.* FROM c JOIN t 
WHERE t.area1= c.cname OR t.area2 =c.cname OR t.area3 = c.cname and 
      t.time>='20140818' and t.time<='20140824' AND platform='pc'
GROUP BY t.time;
```
笛卡尔积是m×n的映射，耗时且非内存。
`or`时，采用`union all`代替：
```sql
select * from 
(
    SELECT c.*, t.* FROM c JOIN t on t.area1= c.cname
    WHERE t.time>='20140818' and t.time<='20140824' AND platform='pc'
    
    union ALL 
    
    SELECT c.*, t.* FROM c JOIN t on t.area2= c.cname
    WHERE t.time>='20140818' and t.time<='20140824' AND platform='pc'
    
    union ALL 
    
    SELECT c.*, t.* FROM c JOIN t on t.area3= c.cname
    WHERE t.time>='20140818' and t.time<='20140824' AND platform='pc'
)tmp
GROUP BY tmp.time;
```
此时使用`union all`，可以再上上面章节的并发优化`set hive.exec.parallel=true;` 

总结自：https://cloud.tencent.com/developer/article/1043838

### mapreduce.task.io.sort.mb
`mapreduce.task.io.sort.mb` 内存  溢出

Combiner会优化MapReduce的中间结果，所以它在整个模型中会多次使用。那么哪些场景才能使用Combiner呢？从这里分析，Combiner的输出是Reducer的输入，Combiner绝不能改变最终的计算结果。所以从我的想法来看，Combiner只应该用于那种Reduce的输入key/value与输出key/value类型完全一致，且不影响最终结果的场景。比如累加，最大值等。Combiner的使用一定得慎重，如果用好，它对job执行效率有帮助，反之会影响reduce的最终结果。
### 参考
https://blog.csdn.net/yycdaizi/article/details/43341239