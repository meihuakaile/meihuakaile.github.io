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

### 动态分区
`set hive.exec.dynamic.partition=true;`
`set hive.exec.dynamic.partition.mode=nonstrict;` 设置可以动态分区；因为严格模式下，不允许所有的分区都被动态指定。（详细使用看上面“导出数据到表”章节）
`set hive.exec.max.dynamic.partitions=100;` 默认是1000；在所有执行的MR节点上，一共可以创建最大动态分区数
`set hive.exec.max.dynamic.partitions.pernode=100;`  默认是100；在每个执行MR的节点上，最大可以创建多少个动态分区。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。

动态分区参考：http://lxw1234.com/archives/2015/06/286.htm

### group by数据倾斜优化
`SET hive.groupby.skewindata=true;` 当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作.
~~从上面group by语句可以看出，这个变量是用于控制负载均衡的。当数据出现倾斜时，如果该变量设置为true，那么Hive会自动进行负载均衡~~
~~比如A日志表与B码表join，但是A中的关联字段id仅是B中id的一小部分，这时候很容易出现reduce阶段倾斜，大量的reduce空跑，因为这些空跑的reduce分到的B的id在A中不存在。~~
`set hive.map.aggr=true;` 在mapper端部分聚合，相当于Combiner 。Map-Side聚合（一般在聚合函数sum,count时使用）。

特别的有`select count(distinct name) from user group by uid;` 即count distinct + （group by）的情况。
改变为`select count(name) from (select uid, name from user group by uid, name)t group by uid`.
主要是把count distinct改变成group by。
```sql
select split(uid, '_')[0], sum(names) from
(
  select concat_ws('-', uid, substr(rand()*10, 1, 1)) uid, count(name) names
  from 
  (
    select uid, name
    from user
    group by uid, name
  )a
  group by concat_ws('-', uid, substr(rand()*10, 1, 1))
)b
group by split(uid, '_')[0]
```
### join 数据倾斜优化
1、`null=null`结果是null，即false，因此在join之前把key为null的值去掉。
   如果null有用不能去掉。办法（1）用union all
   ```sh
   Select * From log a 
　　　Join users b 
     On a.user_id is not null And a.user_id = b.user_id
　 Union all 
   Select * from log a　where a.user_id is null;
   ```
   办法（2）赋予null值新的随机值
   ```sh
    Select * from log a 
　　left outer Join bmw_users b 
   on case when a.user_id is null then concat(‘dp_hive’,rand()) 
　　   else a.user_id end = b.user_id; 
   ```
   
2、`group by`操作放在join之前，减少join的笛卡尔积大小。
3、`set hive.skewjoin.key=100000;` hive 在运行的时候没有办法判断哪个 key 会产生多大的倾斜，所以使用这个参数控制倾斜的阈值，如果超过这个值，新的值会发送给那些还没有达到的 reduce

### 并发优化
`set hive.exec.parallel=true;`  开启任务并行执行
`set hive.exec.parallel.thread.number=8;`  同一个sql允许并行任务的最大线程数

job之间没有前后依赖的都可以并行执行。
### 内存
`set mapreduce.map.memory.mb=10240;`  container的内存 运行mapper的容器的物理内存，1024M = 1G
`set mapreduce.map.java.opts='-Xmx7680M';`  jvm堆内存
`set mapreduce.reduce.memory.mb=10240;`
`set mapreduce.reduce.java.opts='-Xmx7680M';`
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

`set hive.exec.compress.output=true;`  打开job最终输出压缩的开关，设置之后必须设置下面这行，否则还是没有压缩效果
`set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;`  设置压缩类型
`set mapred.output.compression.type=BLOCK;` 大文件压缩仍然会耗时，而且影响mapper并行（mapper并行和文件的个数有关），这个设置，使大的文件可以分割成小文件进行压缩

这种处理文件压缩的能力并非是hive特有的，实际上，使用了hadoop的TextInputFormat进行处理，它可以识别后缀名是.deflate或.gz的压缩文件，并可以轻松处理。
hive无需关心底层文件是否是压缩的，以及如何压缩的。

### 输出文件合并
**_文件合并 和 上面的压缩 并存时会失效。而且文件合并对`orc`格式的表（orc本身就已经压缩）不起作用。_**
`set hive.merge.mapfiles = true `#在Map-only的任务结束时合并小文件
`set hive.merge.mapredfiles = true` #在Map-Reduce的任务结束时合并小文件
`set hive.merge.size.per.task = 256*1000*1000` #合并后每个文件的大小，默认256000000
`set hive.merge.smallfiles.avgsize=16000000 `#平均文件大小，是决定是否执行合并操作的阈值，默认16000000
触发合并的条件是：
根据查询类型不同，相应的mapfiles/mapredfiles参数必须需要打开，即前两个参数根据场景必须有一个为true；
结果文件的平均大小需要小于avgsize参数的值。

合并过程：结果文件进行合并时会执行一个额外的map-only脚本，mapper的数量是文件总大小除以size.per.task参数所得的值。

### 控制mapper大小
文件合并失效，且job只有map时，map的个数就是文件个数；通过控制map大小控制map个数，以控制输出文件个数。
`set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;` 执行Map前进行小文件合并
`set mapred.max.split.size=2048000000;` 2G 每个Map最大输入大小。
`set mapred.min.split.size=2048000000;`  
`set mapred.min.split.size.per.node=2048000000;` 一个节点上split的至少的大小 ，决定了多个data node上的文件是否需要合并
`set mapred.min.split.size.per.rack=2048000000;` 一个交换机下split的至少的大小，决定了多个交换机上的文件是否需要合并
MR-Job 默认的输入格式 FileInputFormat 为每一个小文件生成一个切片。
CombineFileInputFormat 通过将多个“小文件”合并为一个”切片”（在形成切片的过程中也考虑同一节点、同一机架的数据本地性），让每一个 Mapper 任务可以处理更多的数据，从而提高 MR 任务的执行速度。

https://www.cnblogs.com/skyl/p/4754999.html

### 小文件压缩
解决小文件的问题可以从两个方向入手：
1. 输入合并。即在Map前合并小文件
2. 输出合并。即在输出结果的时候合并小文件
对于输出结果为压缩文件形式存储的情况，如果使用输出合并，则必须配合SequenceFile来存储，否则无法进行合并。

### 关于Strict Mode 

 Hive中的严格模式可以防止用户发出（可以有问题）的查询无意中造成不良的影响。 将hive.mapred.mode设置成strict可以禁止三种类型的查询：
 1）、在一个分区表上，如果没有在WHERE条件中指明具体的分区，那么这是不允许的，换句话说，不允许在分区表上全表扫描。这种限制的原因是分区表通常会持非常大的数据集并且可能数据增长迅速，对这样的一个大表做全表扫描会消耗大量资源，必须要再WHERE过滤条件中具体指明分区才可以执行成功的查询。
 2）、第二种是禁止执行有ORDER BY的排序要求但没有LIMIT语句的HiveQL查询。因为ORDER BY全局查询会导致有一个单一的reducer对所有的查询结果排序，如果对大数据集做排序，这将导致不可预期的执行时间，必须要加上limit条件才可以执行成功的查询。
 3）、第三种是禁止产生笛卡尔集(full Cartesian product)。在JION接连查询中没有ON连接key而通过WHERE条件语句会产生笛卡尔集，需要改为JOIN...ON语句。  

### reducer个数
`set mapreduce.job.reduces=15;` 指定reducer个数

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
### 参考
https://blog.csdn.net/yycdaizi/article/details/43341239