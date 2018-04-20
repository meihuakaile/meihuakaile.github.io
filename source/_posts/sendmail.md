---
title: 'sendmail'
date: "2018/04/20"
tags: [ubuntu]
categories: [ubuntu]
copyright: true
---
# 安装
```
sudo apt-get install sendmail  
sudo apt-get install sendmail-cf
```
buntu下使用最常用的mail功能，需要安装mailutils，
安装命令：`sudo apt-get install mailutils `
# 配置
sendmail 默认只会为本机用户发送邮件，只有把它扩展到整个Internet，才会成为真正的邮件服务器。

打开sendmail的配置宏文件：/etc/mail/sendmail.mc
 `vi  /etc/mail/sendmail.mc`
找到如下行修改Addr=0.0.0.0，表明可以连接到任何服务器。： 
```
DAEMON_OPTIONS(`Family=inet,  Name=MTA-v4, Port=smtp, Addr=127.0.0.1')dnl
```
生成新的配置文件：
```
cd /etc/mail  
mv sendmail.cf sendmail.cf~      //做一个备份  
mv sendmail.mc > sendmail.cf   //>的左右有空格，提示错误没有安装sendmail-cf 
```
# 其他
中间发邮件一直发不出去（也有可能是发送太慢了），试了几种办法，最后不知道哪种达到效果.感觉是第二种，因为收到的邮件会显示由“XXX”代发，XXX就是主机名。
（1）执行： `/etc/init.d/sendmail start`
Starting sendmail: [ OK ] Starting sm-client: [ OK ]

（2）host查看主机名
修改host文件：vim /etc/hosts
原始的可能是： 127.0.0.1 localhost
改成： 127.0.0.1  localhost XXX(主机名)
# 使用
（1）mail youtoemail   之后回车，会提示Cc填抄送人；subject填主题；填完主题，回车写消息体body，写完ctrl+d再回车就可以发送了。
这种操作的前提时 必须有前面安装部分的 第三条指令。
还可以，mail -s 邮件标题 收件方邮箱 < content.txt           content.txt放发送邮件的body。  
（2）sendmail.  写一个文件，如email.tx：
```
To: XXX@qq.com
CC: XXXX@qq.com
From: handy<XXX@qq.com>
Subject: test

hello world!
```
然后执行cat email.txt | sendmail -t就可以发送邮件，这种邮件会显示发送方的邮件名。
sendmail另一种使用：
```
echo "content test" | sendmail  XXX@qq.com   
# 发送内容是 "content test"到XXX@qq.com  没有发件人，没有邮箱标题。
```

sendmail的更多用法参考：http://blog.csdn.net/kevinew/article/details/9147969 
参考：http://blog.csdn.net/xin_yu_xin/article/details/45115723