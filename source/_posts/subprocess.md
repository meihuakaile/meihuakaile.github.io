---
title: subprocess
tags: ['subprocess']
categories: ['python']
copyright: true
---
python调用shell比较常用的是subprocess，其他可参看http://www.cnblogs.com/thinker-lj/p/3860123.html

subprocess的参数：

| 名字       | 意义     |
| :------:    | :-------- |
| bufsize  | 	设置缓冲，负数表示系统默认缓冲，0表示无缓冲，正数表示自定义缓冲行数  |
| stdin    | 程序的标准输入句柄，NONE表示不进行重定向，继承父进程，PIPE表示创建管道    |
| stdout   | 程序的标准输出句柄，参数意义同上    |
| stderr   | 程序的标准错误句柄，参数意义同上，特殊，可以设置成STDOUT，表示与标准输出一致    |
| shell    | 为True时，表示将通过shell来执行    |
| cwd      | 用来设置当前子进程的目录    |
| env      | 设置环境变量，为NONE表示继承自父进程的    |
| universal_newlines     | 将此参数设置为True，Python统一把这些换行符当作’/n’来处理。    |


例子，打开qtalk：
```python
# -*- coding:utf-8 -*-
# __author__='chenliclchen'
 
import subprocess
 
def execute(cmd):
    print cmd, "is ready executing...."
    result = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, shell=True)
    stderrinfo, stdinfo = result.communicate()
    print "errinfo:", stderrinfo
    print "info: ", stdinfo
    print "code: ", result.returncode
 
cmd = "~/install/qtalk/run.sh &"
execute(cmd)

```
