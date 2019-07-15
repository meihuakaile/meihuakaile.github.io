---
title: "ERROR 1 HY000 Can't create or write to file 'user.txt' Errcode 13 - Permission denied"
date: "2018/04/19"
tags: [mysql]
categories: ['mysql']
copyright: true
---
把mysql的自动导出数据的目录修改之后的报错“ERROR 1 (HY000): Can't create/write to file '/home/chenliclchen/mysql/user.txt' (Errcode: 13 - Permission denied)”

看着是权限的问题，然后各种的chmod chown的命令修改权限和 拥有者。怎么着都不行。 最后的办法是执行：sudo aa-status ，看到如下：
```
apparmor module is loaded.
22 profiles are loaded.
22 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/evince
   /usr/bin/evince-previewer
   /usr/bin/evince-previewer//sanitized_helper
   /usr/bin/evince-thumbnailer
   /usr/bin/evince-thumbnailer//sanitized_helper
   /usr/bin/evince//sanitized_helper
   /usr/bin/ubuntu-core-launcher
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/cups/backend/cups-pdf
   /usr/lib/lightdm/lightdm-guest-session
   /usr/lib/lightdm/lightdm-guest-session//chromium
   /usr/sbin/cups-browsed
   /usr/sbin/cupsd
   /usr/sbin/cupsd//third_party
   /usr/sbin/ippusbxd
   /usr/sbin/mysqld     ！！！！！要是有这个表示mysql被限制了执行下面绿色的命令
   /usr/sbin/tcpdump
   webbrowser-app
   webbrowser-app//oxide_helper
0 profiles are in complain mode.
3 processes have profiles defined.
3 processes are in enforce mode.
   /sbin/dhclient (920) 
   /usr/sbin/cups-browsed (778) 
   /usr/sbin/mysqld (26226) 
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```
If mysqld is included in enforce mode, then it is the one probably denying the write. Entries would also be written in /var/log/messages when AppArmor blocks the writes/accesses.
之后 修改文件： sudo vim /etc/apparmor.d/usr.sbin.mysqld， 修改如下：
```
 # Allow pid, socket, socket lock file access
/var/run/mysqld/mysqld.pid rw,
/var/run/mysqld/mysqld.sock rw,
/var/run/mysqld/mysqld.sock.lock rw,
/run/mysqld/mysqld.pid rw,
/run/mysqld/mysqld.sock rw,
/run/mysqld/mysqld.sock.lock rw,
/home/chenliclchen/mysql/ r,     #####添加的两行，添加想要保存导出数据的文件夹地址
/home/chenliclchen/mysql/* rw,   #####
```
最后再sudo /etc/init.d/apparmor reload 这样整个的流程完成。可以把导出的数据保存到/home/chenliclchen/mysql/目录下了。

参考：http://blog.csdn.net/Silver_sail/article/details/8166193