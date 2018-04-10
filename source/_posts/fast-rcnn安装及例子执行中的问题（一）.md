---
title: fast-rcnn安装及例子执行中的问题（一）
date: "2018/04/08"
tags: ['深度学习', 'fast rcnn']
categories: ['faster rcnn', 'fast rcnn问题解决&&安装']
copyright: true
---
[ 推荐安装文章 ](http://weibo.com/p/230418855a82cd0102vnjq)  

#### skimage.io
问题：ImportError: No module named skimage.io
解决：sudo pip install scikit-image  
接着出错：ImportError: No module named scipy  
再安装：sudo pip install scipy  
出错：error: library dfftpack has Fortran sources but no Fortran compiler found  
再安装：ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"  
sudo apt-get install gfortran  

  
**_补充_**
今天（原文半年之后）再次遇到，解决：

先 [ 换源 ](/2018/04/08/ubuntu 安装numpy的烂问题libgfortran3依赖)
再执行：
    
       sudo apt-get install python-skimage

#### google.protobuf
问题： 
```
from google.protobuf.internal import enum_type_wrapper  
ImportError: No module named google.protobuf.internal  
```
解决(must use sudo,and when it perform,it will auto install mavproxy next.)：
```
sudo pip install droneapi 
```
or you can look for this paper:https://github.com/dronekit/dronekit-python/issues/121

and there is another people say this:http://www.cnblogs.com/taokongcn/p/4341290.html

  

#### 内存出错
问题：在尝试demo时出现错误（具体英文没有记，大概意思是说多少多少内存不够，或者直接出现“已杀死”的字样）
解决：添加执行参数：./tools/demo.py --cpu --net caffenet  
原因：猜测是因为内存的原因导致caffe崩溃。

