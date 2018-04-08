---
title: cmake安装遇到问题总结
tags: ['cmake']
categories: ['caffe安装&&faster rcnn安装问题&&解决']
copyright: true
---
推荐cmake安装文章 [ 点击打开链接
](http://www.cnblogs.com/emouse/archive/2013/02/22/2922940.html) 以及opencv例子执行。  

1、qmake: could not exec '/usr/lib/x86_64-linux-gnu/qt4/bin/qmake': No such
file or directory

解决：安装：  sudo apt-get install qt-sdk  (470M,so heavy)  
but some people use:  sudo apt-get install qt4-qmake  .don't konw if is ok.you
can try  
  
2、ImportError: No module named numpy.distutils  
解决：  sudo apt-get install python-numpy  
  
3.CMake Error at cmake_install.cmake:36 (FILE):  
file INSTALL cannot set permissions on  
"/usr/local/include/opencv2/opencv_modules.hpp"  
原因：没有权限  
解决：  sudo make install

