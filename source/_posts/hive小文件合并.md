---
title: 'hive小文件合并'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
解决小文件的问题可以从两个方向入手：
1. 输入合并。即在Map前合并小文件
2. 输出合并。即在输出结果的时候合并小文件

**_对于输出结果为压缩文件形式存储的情况，如果使用输出合并，则必须配合SequenceFile来存储，否则无法进行合并。_**

### 输入文件合并
文件合并失效，且job只有map时，map的个数就是文件个数；通过控制map大小控制map个数，以控制输出文件个数。
`set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;` 执行Map前进行小文件合并
`set mapred.max.split.size=2048000000;` 2G 每个Map最大输入大小
`set mapred.min.split.size=2048000000;`  
`set mapred.min.split.size.per.node=2048000000;` 一个节点上split的至少的大小 ，决定了多个data node上的文件是否需要合并
`set mapred.min.split.size.per.rack=2048000000;` 一个交换机下split的至少的大小，决定了多个交换机上的文件是否需要合并
MR-Job 默认的输入格式 FileInputFormat 为每一个小文件生成一个切片。
CombineFileInputFormat 通过将多个“小文件”合并为一个”切片”（在形成切片的过程中也考虑同一节点、同一机架的数据本地性），让每一个 Mapper 任务可以处理更多的数据，从而提高 MR 任务的执行速度。

https://www.cnblogs.com/skyl/p/4754999.html

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



