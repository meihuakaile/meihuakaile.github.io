---
title: 'WordCount与MapReduce计数器'
date: "2018/11/20"
tags: [hadoop]
categories: ['hadoop']
---
代码地址：https://github.com/meihuakaile/mr_test
# map实现
```java
package com.mr.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Counter;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class WordCountMapper extends Mapper<LongWritable, Text
        , Text, IntWritable> {

    /**
     * map方法是提供给map task进程来调用的，map task进程是每读取一行文本来调用一次我们自定义的map方法
     * map task在调用map方法时，传递的参数：
     * @param key 一行的起始偏移量LongWritable作为key
     * @param value 一行的文本内容Text作为value
     * @param context
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] words = value.toString().split(" ");
        if (words.length == 1){
//            自定义计数器
            context.getCounter("lineCounter", "one").increment(1);
//            枚举计数器
            context.getCounter(LOG_LINE_COUNTER.ONE).increment(1);
        }
        else if (words.length == 2){
            Counter two = context.getCounter("lineCounter", "two");
            two.setValue(100);
            two.increment(1);
            context.getCounter(LOG_LINE_COUNTER.TWO).increment(1);
        }else if (words.length == 3){
            context.getCounter("lineCounter", "three").increment(1);
            context.getCounter(LOG_LINE_COUNTER.THREE).increment(1);
        }else{
            context.getCounter("lineCounter", "too long").increment(1);
            context.getCounter(LOG_LINE_COUNTER.TOO_LONG).increment(1);
        }
        for(String word: words){
            context.write(new Text(word), new IntWritable(1));
        }
    }

    public static enum LOG_LINE_COUNTER{
        ONE, TWO, THREE, TOO_LONG
    };
}

```
# reduce实现
```java
package com.mr.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    /**
     * reduce方法提供给reduce task进程来调用
     * reduce task会将shuffle阶段分发过来的大量kv数据对进行聚合，聚合的机制是相同key的kv对聚合为一组
     * 然后reduce task对每一组聚合kv调用一次我们自定义的reduce方法
     * @param key 一组kv中的key
     * @param values
     */
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;
        for (IntWritable value: values){
            count += value.get();
        }
        context.write(key, new IntWritable(count));
    }
}
```
# job
```java
package com.mr.wordcount;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Counter;
import org.apache.hadoop.mapreduce.Counters;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.log4j.Logger;

import java.io.IOException;

/**
 * WordCountJob
 *
 * @author chenliclchen
 * @date 18-11-9 下午8:46
 */
public class WordCountJob {
    static Logger logger = Logger.getLogger(WordCountJob.class);

    public static int main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration, "test");

//        指定本job所在的jar包
        job.setJarByClass(WordCountJob.class);
//        设置wordCountJob所用的mapper逻辑类为哪个类
        job.setMapperClass(WordCountMapper.class);
//        设置wordCountJob所用的reducer逻辑类为哪个类
        job.setReducerClass(WordCountReducer.class);

//        设置map阶段输出的kv数据类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

//        设置最终输出的kv数据类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

//        设置要处理的文本数据所存放的路径
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        boolean result = job.waitForCompletion(true);
//        自定义计数器用的比较广泛，特别是统计无效数据条数的时候，我们就会用到计数器来记录错误日志的条数。
        Counters counters = job.getCounters();
        Counter one = counters.findCounter(WordCountMapper.LOG_LINE_COUNTER.ONE);
        Counter three = counters.findCounter("lineCounter", "three");
        logger.info("枚举, name: " + one.getName() + " value: " + one.getValue());
        logger.info("自定义, name: " + three.getName() + " value: " + three.getValue());

//        与shell是否成功返回结果保持一致
        return result? 0: 1;
    }
}
```
# 计数器
计数器参考：https://www.cnblogs.com/codeOfLife/p/5521356.html

MapReduce 的计数器Counter用以观察mr执行的详细信息，如进度等。mr自带了很多计数器，如job个数、map个数、
map输入行数等（可以参看上面链接）。
我主要用的是自定义计数器，计数器从另一方便面来说就是日志。特别是在我们记录无效数据时特别有用（比如解析多个日志文件时，统计解析失败的条数。毕竟，如果数据量很大，不能就throw掉不管吧）。

自定义计数器经常用的几个方法：
1、定义计数器
   1)枚举声明计数器
```java
// 自定义枚举变量Enum 
Counter counter = context.getCounter(Enum enum)
```
  2)自定义计数器
```java
// 自己命名groupName和counterName 
Counter counter = context.getCounter(String groupName,String counterName)
```

2、为计数器赋值
   1)初始化计数器
```java
counter.setValue(long value);// 设置初始值
```
   2)计数器自增
`counter.increment(long incr);// 增加计数`

3、获取计数器的值
   1) 获取自定义/枚举计数器的值

```java
Counters counters = job.getCounters();
Counter one = counters.findCounter(WordCountMapper.LOG_LINE_COUNTER.ONE);
Counter three = counters.findCounter("lineCounter", "three");
logger.info("枚举, name: " + one.getName() + " value: " + one.getValue());
logger.info("自定义, name: " + three.getName() + " value: " + three.getValue());
```
  2) 获取内置计数器的值
```java
job.waitForCompletion(true); 
Counters counters=job.getCounters(); 
// 查找作业运行启动的reduce个数的计数器，groupName和counterName可以从内置计数器表格查询（前面已经列举有） 
Counter counter=counters.findCounter("org.apache.hadoop.mapreduce.JobCounter","TOTAL_LAUNCHED_REDUCES");// 假如groupName为org.apache.hadoop.mapreduce.JobCounter，counterName为TOTAL_LAUNCHED_REDUCES 
long value=counter.getValue();// 获取计数值
```
  3) 获取所有计数器的值
```java
Counters counters = job.getCounters(); 
for (CounterGroup group : counters) { 
  for (Counter counter : group) { 
    System.out.println(counter.getDisplayName() + ": " + counter.getName() + ": "+ counter.getValue()); 
  } 
}
```
