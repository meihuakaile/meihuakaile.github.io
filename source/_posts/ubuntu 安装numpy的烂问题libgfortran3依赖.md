---
title: "ubuntu 安装numpy的烂问题libgfortran3依赖"
date: "2018/04/08"
tags: ['libgfortran3依赖', 'numpy安装']
categories: ['caffe安装&&问题&&解决']
copyright: true
---

### 尝试安装---失败  
```
sudo apt-get install python-numpy
```
然后出错：  
```
错误 http://cn.archive.ubuntu.com/ubuntu/ trusty-updates/main libgfortran3 i386 4.8.4-2ubuntu1~14.04  
404  Not Found [IP: 112.124.140.210 80] 
```
然后尝试用它的提示“  apt-get update  或者加上  \--fix-missing”来修复。没有作用。    
之后提示：  
```
“libblas3gf : 依赖: libblas3 但是它将不会被安装  
libgfortran3 : 依赖: gcc-4.8-base (= 4.8.2-19ubuntu1) 但是 4.8.4-2ubuntu1~14.04正要被安装  
E: 有未能满足的依赖关系。请尝试不指明软件包的名字来运行“apt-get -f install”(也可以指定一个解决办法)。”  
```
用  sudo apt-get install libblas3 libblas3gf  可以修复第一个依赖，第二个仍然不行。

### step1 换源  
  
开始换了163的源，出现下面错误，这种情况最好换源。 
```
W: 无法下载 bzip2:/var/lib/apt/lists/partial/mirrors.163.com_ubuntu_dists_trusty-security_main_i18n_Translation-en  Hash 校验和不符  
W: 无法下载 bzip2:/var/lib/apt/lists/partial/mirrors.163.com_ubuntu_dists_trusty-security_universe_i18n_Translation-en  Hash 校验和不符  
W: 无法下载 bzip2:/var/lib/apt/lists/partial/mirrors.163.com_ubuntu_dists_trusty-updates_main_source_Sources  Hash 校验和不符”
```
换源：  
```
     su -登录root用户。
     cd /etc/apt
     cp sources.list sources.list.old
```
打开sources.list，删除原来所有内容，把[网站](http://chenrongya.blog.163.com/blog/static/8747419620143185103297)中清华大学的源拷进去。之后执行：
```   
     apt-get update
```
### step2 安装numpy    
```
su XX(你的用户名  )
sudo apt-get -f install
sudo apt-get install gcc-4.8-base libgfortran3 
sudo apt-get install python-numpy
```
好了，python一下就可以import numpy as np。  
  
numpy的效率很高，是用c++写的。python里可以添加c++代码，大家都知道的吧，所以中间才有依赖g++的情况（我是这样想得）。  
  
在vim里删除文件全部内容的命令(‘:%d’)

