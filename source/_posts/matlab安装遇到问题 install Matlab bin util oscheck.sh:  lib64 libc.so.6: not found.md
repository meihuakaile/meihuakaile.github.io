---
title: "matlab安装遇到问题/install/Matlab/bin/util/oscheck.sh: /lib64/libc.so.6: not found"
date: "2018/04/08" 
tags: ['matlab']
categories: ['caffe安装&&问题&&解决']
copyright: true
---
1./home/cl/install/Matlab/bin/util/oscheck.sh: /lib64/libc.so.6: not found  

解决：

    
定位出lib.so的位置  
    
    locate libc.so


然后软连接：

    
    
    ln -s /lib/x86_64-linux-gnu/libc.so.6 /lib64/libc.so.6

  
2.出现找不到jre的情况，  
解决：也不算解决，重新安装，选择32位的matlab就可以正常安装了。  
  

