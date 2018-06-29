---
title: '调试shell'
date: "2018/04/20"
tags: [ubuntu]
categories: [linux]
copyright: true
---

# shell自带工具调试
shell自带的调试，用参数 -x 和 -n。

1、 bash -x shell文件
会对整个的shell文件执行。
有+的是代码，没有的echo输出。
在shell脚本中某段代码前后添加set -x和set +x可以只调试此段代码。

2、 bash -n shell文件
只检查脚本是否正确，不真的执行。

# bashdb
这是一个第三方工具，安装：`sudo apt-get install bashdb`
bashdb可以单步的执行，有很多的参数用于调试：
```
l 列出当前行以下的10行
/pat/ 向后搜索pat
？pat？ 
n 执行下一条语句，遇到函数，不进入函数里面执行，将函数当作黑盒
s n 单步执行n次，遇到函数进入函数里面
b 行号n 在行号n处设置断点
d 行号n 撤销行号n处的断点
c 行号n 一直执行到行号n处，如果没有写n参数，则直接执行到下一个断点处
R 重新启动
Finish 执行到程序最后
cond n expr 条件断点
print $a 表示显示变量a的值
clear 或者d，清除所有的断点
disable / enable 禁用、启用断点
skip [count] 跳过下面一些代码
return 跳出
```
bashdb --debug file 即可开始调试：
```
chenliclchen@chenliclchen-Latitude-E5440:~/study/crontab$ bashdb --debug ./test.sh
bashdb --debug ./test.sh
bash debugger, bashdb, release 4.3-0.91

Copyright 2002, 2003, 2004, 2006-2012, 2014 Rocky Bernstein
This is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.

(/home/chenliclchen/study/crontab/test.sh:3):
3:	log_prefix_name=$1
bashdb<0> 1
** Undefined command "1". Try "help".
bashdb<0> n
(/home/chenliclchen/study/crontab/test.sh:4):
4:	log_name=${log_prefix_name}`date +%Y-%m-%d --date="-1 day"`'.log.gz'
bashdb<1> n
(/home/chenliclchen/study/crontab/test.sh:5):
5:	threshold_count=$2
bashdb<2> n
(/home/chenliclchen/study/crontab/test.sh:6):
6:	mail_to=$3
bashdb<3> n
(/home/chenliclchen/study/crontab/test.sh:7):
7:	mail_content_file=$4
bashdb<4> b 10
Breakpoint 1 set in file /home/chenliclchen/study/crontab/test.sh, line 10.
bashdb<5> c
2017-12-10.log.gz
file is not exist
```

其他参考：http://blog.techbeta.me/2015/10/shell-debug/
