---
title: 'hive数据倾斜'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
分析导致数据倾斜的数据：
https://blog.csdn.net/bitcarmanlee/article/details/51694101
https://blog.csdn.net/wisgood/article/details/77063606
# group by数据倾斜
倾斜原因：
`select count(distinct name) from user`时 使用distinct会将所有的name值都shuffle到一个reducer里面。
特别的有`select uid, count(distinct name) from user group by uid;` 即count distinct + （group by）的情况。

优化1-优化sql：
（1）主要是把count distinct改变成group by。
改变上面的sql为`select uid, count(name) from (select uid, name from user group by uid, name)t group by uid`.
（2）给group by 字段加随机数打散，聚合，之后把随机数去掉，再次聚合（有点类似下面的参数`SET hive.groupby.skewindata=true;`）：
```sql
select split(uid, '_')[0] uid, sum(names) from
(
  select concat_ws('-', uid, substr(rand()*10, 1, 1)) uid, count(name) names
  from 
  (
    select uid, name
    from user
    group by uid, name
  )a
  group by concat_ws('-', uid, substr(rand()*10, 1, 1))
)b
group by split(uid, '_')[0]
```
如果一条语句里看总记录条数以及去重之后的记录条数，没有办法过滤，有两个选择，要么使用两个sql语句分别跑，然后`full join`。

优化2-加参数：
`SET hive.groupby.skewindata=true;` 当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作.
~~从上面group by语句可以看出，这个变量是用于控制负载均衡的。当数据出现倾斜时，如果该变量设置为true，那么Hive会自动进行负载均衡~~
~~比如A日志表与B码表join，但是A中的关联字段id仅是B中id的一小部分，这时候很容易出现reduce阶段倾斜，大量的reduce空跑，因为这些空跑的reduce分到的B的id在A中不存在。~~
`set hive.map.aggr=true;` 在mapper端部分聚合，相当于Combiner 。Map-Side聚合（一般在聚合函数sum,count时使用）。
`set hive.groupby.mapaggr.checkinterval=100000；`--这个是group的键对应的记录条数超过这个值则会进行分拆,值根据具体数据量设置。
`hive.map.aggr.hash.min.reduction=0.5(默认)`预先取100000条数据聚合,如果聚合后的条数/100000>0.5，则不再聚合

# join 数据倾斜
1、null或者某个无效无效字符太多导致数据倾斜。
   `null=null`结果是null，即false，在join时关联不上，join之前去掉不影响结果；
   ''关联得上，但是不需要时产生不必要的脏数据，可以在join之前把key为null/''的值去掉。
   因为null关联不上如果null有用不能去掉，可以用下面两种方法。
   办法（1）用union all
   ```sh
   Select * From log a 
　　　Join users b 
     On a.user_id is not null And a.user_id = b.user_id
　 Union all 
   Select * from log a　where a.user_id is null;
   ```
   办法（2）赋予null值新的随机值
   ```sh
    Select * 
      from log a 
　　 left Join 
       bmw_users b 
     on case when a.user_id is null then concat('dp_hive',rand()) 
　　   else a.user_id end = b.user_id; 
   ```
   方法2比1好，效率上。处理某个值的数据倾斜时都可以尝试方法2。
2、Map输出key数量极少，导致reduce端退化为单机作业。
   **_尽量尽早地过滤数据，减少每个阶段的数据量;把where、group by等操作放在join之前。_**
   先对join的表去重，即把`group by`操作放在join之前，减少join的笛卡尔积大小。
3、Map输出key分布不均，少量key对应大量value，导致reduce端单机瓶颈。
   下面的参考链接用的是切片取样的方法。
   我一般使用上面2中的办法，尽早去重。
4、不同数据类型关联也会产生数据倾斜。 
   在on时强转`On a.auction_id = cast(b.auction_id as string);`
5、`set hive.skewjoin.key=100000;`join的键对应的记录条数超过这个值则会进行分拆,值根据具体数据量设置； hive 在运行的时候没有办法判断哪个 key 会产生多大的倾斜，所以使用这个参数控制倾斜的阈值，如果超过这个值，新的值会发送给那些还没有达到的 reduce。对`full outer join`无效。

reduce倾斜参考：https://www.cnblogs.com/skyl/p/4855099.html