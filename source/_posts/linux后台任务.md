---
title: 'linux后台任务'
date: "2018/04/20"
tags: [ubuntu]
categories: [linux]
copyright: true
---
1、通过 jobs  命令可以看到所有后台的所有的任务。

2、执行命令后面加 & ，命令会在后台执行。但是输入输出仍然会出现在终端上，所以这个任务还和这个终端关联这的。

3、通过 ctrl + z 终止的命令，可以用 pg 命令使其继续执行。

4、nobup命令可以把命令把终端彻底分开。如：
```
nohup ./install/qtalk/run.sh &
nohup ./install/qtalk/run.sh >qtalk_log.txt  2>&1 &
```
\>filename 规定输出目录文件，默认是 当前目录下的nobup.out
0、1和2分别表示标准输入、标准输出和标准错误信息输出，2>&1：即将错误信息重定向到标准输出

5、如果是已经用了&在后台的任务，现在想与终端脱离，就使用disown。

在终端中启动的任务是属于这个终端的子进程，终端（父进程）关闭子进程就会关闭。所以一般使用的办法是修改任务的父进程。

参考：https://yq.aliyun.com/ziliao/54263