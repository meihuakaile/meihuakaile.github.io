---
title: ubuntu 64位安装wps（亲测可用）
date: "2018/04/08"
tags: ['ubuntu wps安装']
categories: ['linux']
copyright: true
---
ubuntu 14.04 64位安装wps  
1.在[官网](http://community.wps.cn/download/)下载.我的是wps-office_9.1.0.4751~a15_i386.deb  
2.切换到下载目录  
3.sudo apt-get install ia32-libs*  
4.sudo dpkg -i --force-architecture  wps-office_9.1.0.4751~a15_i386.deb  
5.会提示错误，错误信息：  
Selecting previously unselected package wps-office.  
。。。  
Errors were encountered while processing:  
wps-office  
6.命令修正：sudo apt-get -f install  
  
  
注释：(dpkg -i *.deb)deb文件安装   （-r）是文件卸载

