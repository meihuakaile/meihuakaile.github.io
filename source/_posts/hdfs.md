---
title: 'hdfs'
date: "2018/10/28"
tags: [hdfs]
categories: [hadoop]
copyright: true
---
![](1.png)
### Namenode
- EditLog:对于 任何对文件系统元数据产生修改 的操作， Namenode 都会使用一种称为 EditLog 的事务日志记录下来。
- FsImage:整个文件系统的命名空间 ，包括数据块到文件的映射、文件的属性等，都存储在一个称为 FsImage 的文件中

### DataNode
Datanode 将 HDFS 数据以文件的形式存储在本地的文件系统中，它并不知道有关 HDFS 文件的信息。它把每个 HDFS 数据块存储在本地文件系统的一个单独的文件中。

当一个 Datanode 启动时，它会扫描本地文件系统，产生一个这些本地文件对应的所有 HDFS 数据块的列表，然后作为报告发送到 Namenode ，这个报告就是块状态报告。

### Secondary NameNode
Secondary NameNode 定期合并 fsimage 和 edits 日志，将 edits 日志文件大小控制在一个限度下。
Secondary NameNode处理流程

(1) 、 namenode 响应 Secondary namenode 请求，将 edit log 推送给 Secondary namenode ， 开始重新写一个新的 edit log 。
(2) 、 Secondary namenode 收到来自 namenode 的 fsimage 文件和 edit log 。
(3) 、 Secondary namenode 将 fsimage 加载到内存，应用 edit log ， 并生成一 个新的 fsimage 文件。
(4) 、 Secondary namenode 将新的 fsimage 推送给 Namenode 。
(5) 、 Namenode 用新的 fsimage 取代旧的 fsimage ， 在 fstime 文件中记下检查 点发生的时

### HDFS的安全模式
Namenode 启动后会进入一个称为安全模式的特殊状态。处于安全模式 的 Namenode 是不会进行数据块的复制的。 Namenode 从所有的 Datanode 接收心跳信号和块状态报告。块状态报告包括了某个 Datanode 所有的数据 块列表。每个数据块都有一个指定的最小副本数。当 Namenode 检测确认某 个数据块的副本数目达到这个最小值，那么该数据块就会被认为是副本安全 (safely replicated) 的;在一定百分比(这个参数可配置)的数据块被 Namenode 检测确认是安全之后(加上一个额外的 30 秒等待时间)， Namenode 将退出安全模式状态。接下来它会确定还有哪些数据块的副本没 有达到指定数目，并将这些数据块复制到其他 Datanode 上。