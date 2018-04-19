---
title: 'sudo cd为什么不能够执行'
date: "2018/04/19"
tags: [ubuntu]
categories: ['ubuntu']
copyright: true
---
问题复现： 开始想cd到某个文件夹，但是爆出“没有权限”；
         之后想直接用“sudo cd”，会爆出“找不到命令”

原因：cd不是一个应用程序而是Linux内建的命令，而sudo仅仅只对应用程序起作用。例如，sudo qtalk只意味着以root权限运行qtalk程序。

解决方法1：使用sudo -i命令提升用户权限

> sudo -i
> cd /var/lib/mysql-files

解决方法2：使用sudo -s命令打开特殊shell

> sudo -s
> cd /var/lib/mysql-files

上面都可以使用exit命令退出

参考：http://blog.csdn.net/u014717036/article/details/70338463