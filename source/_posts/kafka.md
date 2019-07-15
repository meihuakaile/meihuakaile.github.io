---
title: 'kafka'
date: "2018/10/28"
tags: [kafka]
categories: [hadoop]
copyright: true
---
一个主题topic有多个分区。
一个消费者可以消费多个 分区。但是一个分区只能有一个消费者。
当以下事件发生时，Kafka将会进行一次分区分配：
- 同一个ConsumerGroup内新增消费者；
- 消费者离开当前所属的ConsumerGroup，包括shutsdown或crashes；
- 订阅的主题新增分区；

如果你的分区数是N，那么最好线程数也保持为N，这样通常能够达到最大的吞吐量。超过N的配置只是浪费系统资源，因为多出的线程不会被分配到任何分区。
--num.streams N 消费者线程数量
## Producer分区机制
当指定partition key的时候，分配partition的策略：
- hash：由消息所提供的key来进行hash，然后分发到对应的partition。这是默认使用的partition机制。
- 自定义：自己实现partition接口，并在配置中用参数`partitioner.class`指定这个实现。

当没有指定partition key的时候，分配partition的策略：
随机：把每个消息随机分发到一个partition中。在10分钟内，该partition不会切换。所以，当producer数目小于partition时，在一定时间内会有部分partition没有收到数据。

## Consumer消费策略
Kafka提供的两种Consumer消费Partition的分配策略：range和roundrobin，由参数partition.assignment.strategy指定，默认是range策略。
### range策略
Range策略是**_对每个主题而言_**的，首先对同一个主题里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。在我们的例子里面，排完序的分区将会是0, 1, 2, 3, 4, 5, 6, 7, 8, 9；消费者线程排完序将会是C1-0, C2-0, C2-1。然后将partitions的个数除于消费者线程的总数来决定每个消费者线程消费几个分区。如果除不尽，那么前面几个消费者线程将会多消费一个分区。在我们的例子里面，我们有10个分区，3个消费者线程， 10 / 3 = 3，而且除不尽，那么消费者线程 C1-0 将会多消费一个分区，所以最后分区分配的结果看起来是这样的：
```
C1-0 将消费 0, 1, 2, 3 分区
C2-0 将消费 4, 5, 6 分区
C2-1 将消费 7, 8, 9 分区
```
假如我们有11个分区，那么最后分区分配的结果看起来是这样的：
```
C1-0 将消费 0, 1, 2, 3 分区
C2-0 将消费 4, 5, 6, 7 分区
C2-1 将消费 8, 9, 10 分区
```
假如我们有2个主题(T1和T2)，分别有10个分区，那么最后分区分配的结果看起来是这样的：
```
C1-0 将消费 T1主题的 0, 1, 2, 3 分区以及 T2主题的 0, 1, 2, 3分区
C2-0 将消费 T1主题的 4, 5, 6 分区以及 T2主题的 4, 5, 6分区
C2-1 将消费 T1主题的 7, 8, 9 分区以及 T2主题的 7, 8, 9分区
```
Range策略中一个很明显的短板就是对于多topic的情况，可能会存在某一个消费线程压力过大的问题，无法做到真正的均衡。

### roundrobin策略
使用RoundRobin策略有两个前提条件必须满足：
- 同一个Consumer Group里面的所有消费者的num.streams必须相等；
- 每个消费者订阅的主题必须相同。

所以这里假设前面提到的2个消费者的num.streams = 2。
RoundRobin策略的工作原理：将**_所有主题的分区_**组成 TopicAndPartition 列表，然后对 TopicAndPartition 列表按照 hashCode 进行排序。在例子里面，加入按照 hashCode 排序完的topic-partitions组依次为T1-5, T1-3, T1-0, T1-8, T1-2, T1-1, T1-4, T1-7, T1-6, T1-9，我们的消费者线程排序为C1-0, C1-1, C2-0, C2-1，最后分区分配的结果为：最后按照round-robin风格将分区分别分配给不同的消费者线程。
```
C1-0 将消费 T1-5, T1-2, T1-6 分区；
C1-1 将消费 T1-3, T1-1, T1-9 分区；
C2-0 将消费 T1-0, T1-4 分区；
C2-1 将消费 T1-8, T1-7 分区；
```
如果所有consumer实例有相同的consumer group，那么这个就像传统的队列，负载均衡到所有consumer上。假如多个consumer实例都有多个线程，且属于同一个group，那么一个topic的所有partition会均匀分配给所有线程。
接收消息的顺序只能保证一个partition之内是有序的，一个consumer接收多个partition的话是无法保证消息全局有序的，即consumer接收的消息的顺序可能跟producer发送的顺序不同。

## 参考
https://blog.csdn.net/WangQYoho/article/details/76169514
http://lsr1991.github.io/2015/07/09/kafka-partition-mechanism/