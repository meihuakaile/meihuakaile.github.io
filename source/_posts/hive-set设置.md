---
title: 'hive-set设置'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
直接`set`命令可以看到所有变量值。
`set`单个参数，可以看见这个参数的值。
### set hiveconf
Hive相关的配置属性总结
`set hive.cli.print.current.db=true;` 在cli hive提示符后显示当前数据库。
`set hive.cli.print.header=true;` 显示表头。select时会显示对应字段。
`set hive.mapred.mode=strict;` 如果对分区表查询，且没有在where中对分区字段进行限制，报错`FAILED: SemanticException [Error 10041]: No partition predicate found for Alias "test_part" Table "test_part"`；对应还有`nonstrict`模式（默认模式）。
`set hive.exec.reducers.bytes.per.reducer` 每个reduce任务处理的数据量，默认为1000^3=1G

### 动态分区
`set hive.exec.dynamic.partition.mode=nonstrict;` 设置可以动态分区；因为严格模式下，不允许所有的分区都被动态指定。（详细使用看上面“导出数据到表”章节）
`set hive.exec.max.dynamic.partitions=100;` 默认是1000；在所有执行的MR节点上，一共可以创建最大动态分区数
`set hive.exec.max.dynamic.partitions.pernode=100;`  默认是100；在每个执行MR的节点上，最大可以创建多少个动态分区。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。

动态分区参考：http://lxw1234.com/archives/2015/06/286.htm
### 压缩

`set hive.exec.compress.output=true;`  打开最终输出压缩的开关，设置之后必须设置下面这行，否则还是没有压缩效果
`set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;`  设置压缩类型
`set mapred.output.compression.type=BLOCK;` 大文件压缩仍然会耗时，这个设置，大的文件可以分割成小文件进行压缩
