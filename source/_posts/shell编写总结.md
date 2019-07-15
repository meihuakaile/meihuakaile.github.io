---
title: 'shell编写总结'
date: "2018/08/10"
tags: [ubuntu]
categories: [linux]
copyright: true
---
```bash
#!/bin/sh

a="ni n"
b="ni n"
c="nin"
d="ni"

if [ "$a" = "$b" ]; then
  echo 'a = b'
fi
if [ "$a" != "$c" ]; then
  echo 'a != c'
fi
if [ "$c" \> "$d" -o "$c" = "$d" ]; then
  echo 'c >= d '
fi
if [ "$a" \< "$b" -o "$a" = "$b" ]; then
  echo 'a <= b'
fi
```
`/bin/sh` 没有`[[]]`
字符串等于： `[ "$a" = "$b" ] ` 不能用 ==，等号前后的空格不能省
字符串不等： `[ "$a" != "$b" ]`
字符串小于： `[ "$a" \< "$b" ]`
字符串大于： `[ "$a" \> "$b" ]`
报错`“integer expression expected”` 时一般都是因为`[ "qw" -eq "as" ]` 等的操作。` -eq -ne -lt -gt`等只能是数字比较，不能用于字符串。

```bash
q=`expr 1 / 1`
w=$((1/1))
echo $q, $w
# 赋值操作等号前后不能有空格
# 加减乘除： （1）`expr 1 + 1` 加号前后必须有空格;乘法时。乘法符号前必须有\:`expr 1 \* 1`  (2)`$((1+1))` 加号前后可以不用空格
# 只要括号中的运算符、表达式符合C语言运算规则，都可用在$((exp))中，甚至是三目运算符。
# $() 和 `` 效果一样，执行命令
```

```bash
yesterday=`date -d "2018-08-08 1 days ago" +"%Y-%m-%d"`
tomorrow=`date -d "2018-08-08 1 days" +"%Y-%m-%d"`
yesterday1=`date -d "-1 days 2018-08-08" +"%Y-%m-%d"`
tomorrow1=`date -d "1 days 2018-08-08" +"%Y-%m-%d"`
today=`date +"%Y-%m-%d"`
echo $yesterday, $tomorrow, $yesterday1, $tomorrow1, $today
# 日期加减1： `date -d "2018-08-08 1 days ago" +"%Y-%m-%d"` 减时间的效果；去掉`ago`就是加时间的效果
# 日期加减2：`date -d "-1 days 2018-08-08" +"%Y-%m-%d"`  减时间的效果；把`-1`改成`1`/`+1`就是加时间的效果
# date是现在时间 ；上面的`2018-08-08`可以用`now`代替今天
```

```bash
z=1
x=1
v=2
if [ $z = $x ]; then
  echo "z = x"
fi
if [ $z != $v ]; then
  echo "z != v"
fi

# 数字等于： [ $z = $v ] 不能用==，等号前后的空格不能省
# 数字不等： [ $z != $v ]等号前后的空格不能省
```

多线程：
```bash
# run processes and store pids in array
for i in $n_procs; do
    ./procs[${i}] &
    pids[${i}]=$!
done

# wait for all pids
for pid in ${pids[*]}; do
    wait $pid
done
```