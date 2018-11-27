---
title: 'hive数据倾斜'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
# group by数据倾斜
`SET hive.groupby.skewindata=true;` 当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作.
~~从上面group by语句可以看出，这个变量是用于控制负载均衡的。当数据出现倾斜时，如果该变量设置为true，那么Hive会自动进行负载均衡~~
~~比如A日志表与B码表join，但是A中的关联字段id仅是B中id的一小部分，这时候很容易出现reduce阶段倾斜，大量的reduce空跑，因为这些空跑的reduce分到的B的id在A中不存在。~~
`set hive.map.aggr=true;` 在mapper端部分聚合，相当于Combiner 。Map-Side聚合（一般在聚合函数sum,count时使用）。


特别的有`select count(distinct name) from user group by uid;` 即count distinct + （group by）的情况。
改变为`select count(name) from (select uid, name from user group by uid, name)t group by uid`.
主要是把count distinct改变成group by。
```sql
select split(uid, '_')[0], sum(names) from
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

# join 数据倾斜
1、`null=null`结果是null，即false；''关联得上，但是两者都会产生不必要的脏数据，可以在join之前把key为null/''的值去掉。
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
    Select * from log a 
　　left outer Join bmw_users b 
   on case when a.user_id is null then concat(‘dp_hive’,rand()) 
　　   else a.user_id end = b.user_id; 
   ```
   
2、`group by`操作放在join之前，减少join的笛卡尔积大小。
3、`set hive.skewjoin.key=100000;` hive 在运行的时候没有办法判断哪个 key 会产生多大的倾斜，所以使用这个参数控制倾斜的阈值，如果超过这个值，新的值会发送给那些还没有达到的 reduce。对full outer join无效。

