---
title: ubuntu下android真机调试配置及重启
tags: ['android', '真机调试']
categories: ['java', 'android', '问题解决&&安装']
copyright: true
---
真机配置  

1、不连接手机，在shell终端输入lsusb命令，连上手机，再输入lsusb，观察两次的不同，多出来的那个端口号就是你手机的。

2、打开/etc/udev/rules.d/51-android.rules,加上：  
SUBSYSTEM=="usb", ATTR{idVendor}=="12d1", MODE="0666"  
把12d1换成你的刚才看到的端口号。  
3、重启udev服务  
sudo service udev restart  
4、重启adb服务  
进入adb所在目录执行：  
sudo ./adb kill-server  

sudo ./adb start-server

  

重启：

sudo ./adb kill-server  
./adb devices  
./adb root (这一步很重要 )  

