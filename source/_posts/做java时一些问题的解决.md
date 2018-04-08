---
title: 做java时一些问题的解决
tags: ['问题&&解决']
categories: ['java', '问题解决&&安装']
copyright: true
---
1、java.lang.classnotfoundexception（这个是在做hadoop时遇到）  
  
因为我的jdk安装了openJDK，重新安装sunJDk就行了。  
  
  
2.改了java home，但是java -version仍然是原来的版本  
解决：其实java -version是不准的，它只会显示你电脑安装的最新版本。  
  
  

3.这个也是hadoop时出现的错
``` 
16/01/01 21:48:01 WARN mapred.LocalJobRunner: job_local_0001

java.lang.NullPointerException  
at org.apache.hadoop.io.serializer.SerializationFactory.getSerializer(Serializ
ationFactory.java:73)  
at org.apache.hadoop.mapred.MapTask$MapOutputBuffer.<init>(MapTask.java:965)  
at
org.apache.hadoop.mapred.MapTask$NewOutputCollector.<init>(MapTask.java:674)  
at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:756)  
at org.apache.hadoop.mapred.MapTask.run(MapTask.java:370)  
at org.apache.hadoop.mapred.LocalJobRunner$Job.run(LocalJobRunner.java:212)  
16/01/01 21:48:01 INFO mapred.JobClient:  map 0% reduce 0%  
16/01/01 21:48:01 INFO mapred.JobClient: Job complete: job_local_0001  
16/01/01 21:48:01 INFO mapred.JobClient: Counters: 0  
```
解决：问题出的很挫，主要是因为把输出的intWritable写成了integer。  
  
  

4.对字符串使用replaceAll()方法替换 * ? + / | 等字符的时候会报以下异常：Dangling meta character '*' near index 0

  

这主要是因为这些符号在正则表达示中有相应意义。  
  
只需将其改为 [*] 或 //* 即可。（但其实我在split中是\\\\*）

