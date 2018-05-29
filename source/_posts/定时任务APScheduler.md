---
title: 定时任务APScheduler
tags: ['APScheduler']
categories: ['python']
copyright: true
---
# 定时任务APScheduler
两种使用方法，注解和代码，参考：http://www.jb51.net/article/117989.htm
三种定时，参考：https://www.cnblogs.com/luxiaojun/p/6567132.html

#### 安装
pip install APScheduler
#### 基础
（1）、触发器(trigger)
　　包含调度逻辑，每一个作业有它自己的触发器，用于决定接下来哪一个作业会运行，根据trigger中定义的时间点，频率，时间区间等等参数设置。除了他们自己初始配置以外，触发器完全是无状态的。
（2）、作业存储(job store)
　　存储被调度的作业，默认的作业存储是简单地把作业保存在内存中，其他的作业存储是将作业保存在数据库中。一个作业的数据讲在保存在持久化作业存储时被序列化，并在加载时被反序列化。调度器不能分享同一个作业存储。job store支持主流的存储机制：redis, mongodb, 关系型数据库,　内存等等
（3）、执行器(executor)
　　处理作业的运行，他们通常通过在作业中提交制定的可调用对象到一个线程或者进城池来进行。当作业完成时，执行器将会通知调度器。基于池化的操作，可以针对不同类型的作业任务，更为高效地使用cpu的计算资源。
（4）、调度器(scheduler)
　　通常在应用只有一个调度器，调度器提供了处理这些的合适的接口。配置作业存储和执行器可以在调度器中完成，例如添加、修改和移除作业。

我们在代码里只关注调度器。
这里简单列一下常用的若干调度器：

`BlockingScheduler`：仅可用在当前你的进程之内，与当前的进行共享计算资源
`BackgroundScheduler`:　在后台运行调度，不影响当前的系统计算运行
`AsyncIOScheduler`:　如果当前系统中使用了async module，则需要使用异步的调度器
`GeventScheduler`:　如果使用了gevent，则需要使用该调度
`TornadoScheduler`:　如果使用了Tornado, 则使用当前的调度器
`TwistedScheduler`:Twister应用的调度器
`QtScheduler`:　Qt的调度器
#### 注解使用
两种使用方法。只介绍注解，另一种可参考上面的链接。
```python
from apscheduler.schedulers.blocking import BlockingScheduler
sched = BlockingScheduler()

@sched.scheduled_job('cron', day_of_week='mon', hour=10)
def scheduled():
     do_something......
sched.start()
```
上面实现的是每周一`（day_of_week='mon'）`的10点`（hour=10）`执行scheduled方法。

#### 三种定时方法
一共有三种定时方法（只简单介绍，深入了解参考上面第2个链接）。
注解的第一个参数是trigger，它管理着作业的调度方式。它可以为date, interval或者cron。对于不同的trigger，对应的参数也相同。
(1)interval 是每\*都执行。如每分钟执行一次:
`@sched.scheduled_job('interval', minutes=1)`
(2)cron一般是每个定时时刻发生，比如3中例子。也可以实现（1）中的那种：
`@sched.scheduled_job('cron', minute='*/1')`
（3）date是定时只发生一个。