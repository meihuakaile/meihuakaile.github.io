---
title: 'ERROR 1290 HY000  The MySQL server is running with the --secure-file-priv option so it cannot execute this statement'
date: "2018/04/19"
tags: [mysql]
categories: ['mysql']
copyright: true
---
在使用into outfile 进行数据导出时报的错：
ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement。

原因：在MySQL 5.7.6版本之后，导入文件只能在secure_file_priv指定的文件夹下
在mysql中使用  show variables like '%secure%'; 看到结果
![](1.png)
因此，我们可以选择把数据导出到 ‘/var/lib/mysql-files’ 的文件下。这时可能你想进入到这个文件夹下面看看，可能会出现cd不进去的情况。解决如下：
[sudo cd为什么不能够执行](/2018/04/19/sudo cd为什么不能够执行/)

或者你不想这样做，你想把数据导出你想导出的目录，可以这样：
修改文件：vim  /etc/mysql/mysql.conf.d/mysqld.cnf   也有的是  vim /etc/mysql/mysql.conf，我是因为后面的文件不存在才使用的前面的那个文件。（此处，我的理解是，我先看了mysql下有个文件，名字是“mysql.cnf”，它的里面是两行include，以c语言的语法是，它包含了另外两个文件，因此在另外两个文件夹下找自己需要的'[mysqld]'，当然不要一个一个的看，用grep命令： grep -nr "mysqld" ./）
**_在\[mysqld\] 下边写上 secure\_file\_priv=/home/chenliclchen/mysql/   后面的路径替换成你想保存的路径。_**

然后重启mysql ：  service  mysql restart
但是要注意你的要保存的那个路径必须是有读写权限的，这也是个问题。 修改之后文件路径还必须是绝对路径。
这段参考：http://blog.csdn.net/learner_lps/article/details/65448098

前面的全部修改之后，执行之前的导出数据的命令，报错“ERROR 1 (HY000): Can't create/write to file '/home/chenliclchen/mysql/user.txt' (Errcode: 13 - Permission denied)”
[点击获取解决方案](/2018/04/19/ERROR 1 HY000 Can't create or write to file 'user.txt' Errcode 13 - Permission denied/)