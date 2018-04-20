---
title: 'crontab定时任务'
date: "2018/04/20"
tags: [ubuntu]
categories: [ubuntu]
copyright: true
---
# crontab参数
-e 编辑
-l 查看定时任务
-r 删除定时任务
-ir 删除前提醒

# crontab使用
定时文件格式：
```
minute hour day-of-month month-of-year day-of-week commands
```
用空格隔开，和spring的定时任务类似，有*等特殊字符
如：
```
*/1 * * * * echo "hello" >> ~/hello.log      每一分钟发送一个“hello”到hello.log文件
```
如果想定时的执行某个shell文件，注意一定要是绝对路径，shell文件后还可以跟参数，如：
```
*/1 * * * * /home/chenliclchen/study/crontab/test.sh  /home/chenliclchen/study/crontab/result.log >> /home/chenliclchen/study/crontab/hello.log  
# 每分钟执行test.sh文件 result.log是test.sh的参数，然后把输出结果输出到hello.log文件里。
```
如何让上面的定时命令执行起来，有两种方法：
1、把上面的定时任务写在cron文件里，如crontest.cron文件
执行`crontab ./crontest.cron`就加入了上面的定时任务 ，此时已经不关crontest.cron文件的事了，如果修改了这个文件又想使定时任务真的有效需要再次执行这条命令。
之后就可以通过`crontab -l` 查看定时任务了；
也可以通过 -e的编辑定时任务，注意此时编辑并不会更新到上面的crontest.cron文件里。

2、通过 -e的直接编辑定时任务，把上面的定时任务命令写入。
# cron服务命令
（1）查看cron状态

                          sudo service cron status　
（2）开启cron

                         sudo /etc/init.d/cron start
（3）关闭cron

                        sudo /etc/init.d/cron stop
（4）重启cron

                       sudo /etc/init.d/cron restart

# 定时提醒订饭的例子
每天中午11点每隔5分钟提醒订饭。
（1）写一个文件eat.cron，内容（notify-send命令在cron下不会启动消息弹窗。需要在notify-send命令执行之前添加`export DISPLAY=:0.0.`）：
```
*/5 11 * * * export DISPLAY=:0.0 && notify-send ['订饭啦'] "提醒大家订饭"
```
（2）执行crontab ./eat.cron 添加定时任务
（3）执行 sudo /etc/init.d/cron  start  启动定时任务
（4）查看是否添加成功  crontab -l  显示（1）中的代码
（5）执行 sudo /etc/init.d/cron  stop  停止定时任务

参考：https://www.cnblogs.com/kaituorensheng/p/4494321.html
