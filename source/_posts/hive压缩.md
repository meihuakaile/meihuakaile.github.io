---
title: 'hive压缩'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---

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