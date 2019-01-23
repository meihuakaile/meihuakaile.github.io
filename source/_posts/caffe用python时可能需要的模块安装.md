---
title: caffe用python时可能需要的模块安装
date: "2018/09/08" 
tags: ['python', 'caffe']
categories: ['caffe安装&&问题&&解决']
copyright: true
---
<strong>!!!经验 [ 换源 ](/2018/04/08/ubuntu 安装numpy的烂问题libgfortran3依赖) 之后再装。  </strong>

#  Cython.Distutils
报错：
```
ImportError: No module named Cython.Distutils
```
解决：进入python后，输入命令：
``` 
from Cython.Distutils import build_ext  
```
仍然输出：
```
Traceback (most recent call last):  
File "", line 1, in   
ImportError: No module named Cython.Distutils  
```
则表示你的确没有Cython.Distutils。此时输入命令：
```
pip install cython
```
安装它时可能会提示你还没有安装pip，那就安它提示的命令安装pip后再安装即可。  
！！！！！！  
但实际上这个方法还是没有解决问题。换另一种方法：  
Just install Cython from [ http://cython.org/#download
](http://cython.org/#download) and install it using this command : 
```
 sudo python setup.py install
 sudo python -c 'import Cython.Distutils'
 ```
 and it will be installed and the error message will disappear.  

(晚上有遇到一个类似的问题，用pip安装仍然是fail，仔细看是权限问题，不知道上面那个问题是不是也是权限问题。)

今天再次遇到，简单解决：
```
sudo apt-get install Cython
```

(但是简单解决的前提不知道是不是因为我提前换了源，换源参考： [换源](/2018/04/08/ubuntu 安装numpy的烂问题libgfortran3依赖) )  

 
# easydict 
报错：
```
ImportError: No module named easydict  （类似与上面的错误）
```
解决：
```
    sudo pip install easydict
```
 
# sklearn.utils  
解决： if you have Python 2 you can install all these requirements by issuing:
```
 sudo apt-get install build-essential python-dev python-setuptools python-numpy python-scipy libatlas-dev libatlas3gf-base pip install --user --install-option="--prefix=" -U scikit-learn
```
或者
```
    sudo apt-get install python-sklearn 
```
使用下面命令测试sklearn是否安装正确：
```
nosetests -v sklearn 
```
# pandas
解决：
```
    sudo apt-get install python-pandas
```
 
# cv2
报错：
```
ImportError: No module named cv2
```
解决：
```
    sudo apt-get install python-opencv
```
# yaml
报错：
```
ImportError: No module named yaml
```
解决：
```
 sudo apt-get install python-yaml
 ```
# lmdb
报错：
```
 sudo pip install lmdb
 ```

# numpy
[ numpy ](/2018/04/08/ubuntu 安装numpy的烂问题libgfortran3依赖)  
# skimage.io | google.protobuf
[ skimage.io | google.protobuf.internal ](/2018/04/08/fast-rcnn安装及例子执行中的问题（一）)