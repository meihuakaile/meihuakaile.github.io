---
title: 'hive小文件合并'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
有多少个reducer（mapper）输出就会生成多少个输出文件，根据shuffle/sort的原理，每个文件按照某个值进行shuffle后的结果。
### 小文件带来的问题
HDFS的文件元信息，包括位置、大小、分块信息等，都是保存在NameNode的内存中的。每个对象大约占用150个字节，因此一千万个文件及分块就会占用约3G的内存空间，一旦接近这个量级，NameNode的性能就会开始下降了。

此外，HDFS读写小文件时也会更加耗时，因为每次都需要从NameNode获取元信息，并与对应的DataNode建立连接。对于MapReduce程序来说，小文件还会增加Mapper的个数，job作为一个独立的jvm实例，每个job只处理很少的数据，其开启和停止的开销可能会大大超过实际的任务处理时间，浪费了大量的调度时间。当然，这个问题可以通过使用CombinedInputFile和JVM重用来解决

### 分析解决方案
解决小文件的问题可以从两个方向入手：
1. 输入合并。即在Map前合并小文件。这个方法即可以解决之前小文件数太多，导致mapper数太多的问题；还可以防止输出小文件合数太多的问题（因为mr只有map时，mapper数就是输出的文件个数）。
2. 输出合并。即在输出结果的时候合并小文件。

**_对于输出结果为压缩文件形式存储的情况，如果使用输出合并，则必须配合SequenceFile来存储，否则无法进行合并。_**

### 输入文件合并
文件合并失效，且job只有map时，map的个数就是文件个数；通过控制map大小控制map个数，以控制输出文件个数。
`set hive.hadoop.supports.splittable.combineinputformat=true;` 开关
`set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;` 执行Map前进行小文件合并
`set mapred.max.split.size=2048000000;` 2G 每个Map最大输入大小  
`set mapred.min.split.size.per.node=2048000000;` 一个节点上split的至少的大小 ，决定了多个data node上的文件是否需要合并
`set mapred.min.split.size.per.rack=2048000000;` 一个交换机下split的至少的大小，决定了多个交换机上的文件是否需要合并
MR-Job 默认的输入格式 FileInputFormat 为每一个小文件生成一个切片。
CombineFileInputFormat 通过将多个“小文件”合并为一个”切片”（在形成切片的过程中也考虑同一节点、同一机架的数据本地性），让每一个 Mapper 任务可以处理更多的数据，从而提高 MR 任务的执行速度。

原理（未细看）：https://www.cnblogs.com/skyl/p/4754999.html
### 输出文件合并
**_文件合并 和 压缩 并存时会失效。而且文件合并对`orc`格式的表（orc本身就已经压缩）不起作用。_**
`set hive.merge.mapfiles = true `#在Map-only的任务结束时合并小文件
`set hive.merge.mapredfiles = true` #在Map-Reduce的任务结束时合并小文件
`set hive.merge.size.per.task = 256*1000*1000` #合并后每个文件的大小，默认256000000
`set hive.merge.smallfiles.avgsize=16000000 `#平均文件大小，是决定是否执行合并操作的阈值，默认16000000
触发合并的条件：
根据查询类型不同，相应的mapfiles/mapredfiles参数必须需要打开，即前两个参数根据场景必须有一个为true；
结果文件的平均大小需要小于avgsize参数的值。

合并过程：结果文件进行合并时会执行一个额外的map-only脚本，mapper的数量是文件总大小除以size.per.task参数所得的值。
### 解决Hive创建文件数过多的其他方法
动态分区好用，但是会产生很多小文件。原因就在于，假设初始有N个mapper,最后生成了m个分区，最终会有多少个文件生成呢？答案是N*m,是的，每一个mapper会生成m个文件，就是每个分区都会对应一个文件，这样的话你算一下。所以小文件就会成倍的产生。
怎么解决这个问题，通常处理方式也是像上面那样，让数据尽量聚到少量reducer里面。但是有时候虽然动态分区不会产生reducer,但是也就意味着最后没有进行文件合并,我们也可以用distribute by rand()这句来保证数据聚类到相同的reducer。

https://www.iteblog.com/archives/1533.html


### 参看
https://blog.csdn.net/yycdaizi/article/details/43341239