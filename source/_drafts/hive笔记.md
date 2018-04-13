---
title: hive笔记
date: "2018/04/13"
tags: ['hadoop', 'hive']
categories: ['hadoop']
copyright: true
---
架构在Hadoop之上，提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。
Hadoop是一个开源框架来存储和处理大型数据在分布式环境中。它包含两个模块，一个是MapReduce，另外一个是Hadoop分布式文件系统（HDFS）。
* **MapReduce**：它是一种并行编程模型在大型集群普通硬件可用于处理大型结构化，半结构化和非结构化数据。
* **HDFS**：Hadoop分布式文件系统是Hadoop的框架的一部分，用于存储和处理数据集。它提供了一个容错文件系统在普通硬件上运行。

**Hive 不是一个关系数据库/实时查询和行级更新的语言.**