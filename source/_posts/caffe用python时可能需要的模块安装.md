---
title: caffe用python时可能需要的模块安装
tags: ['python', 'caffe']
categories: ['caffe安装&&faster rcnn安装问题&&解决']
copyright: true
---
!!!经验 [ 换源 ](http://blog.csdn.net/u010668907/article/details/50939838) 之后再装。  

1.ImportError: No module named Cython.Distutils

  
解决：尝试python进入python后，输入命令：from Cython.Distutils import build_ext  
仍然输出：Traceback (most recent call last):  
File "<stdin>", line 1, in <module>  
ImportError: No module named Cython.Distutils  
则表示你的确没有Cython.Distutils。此时输入命令：pip install
cython安装它这是可能会提示你还没有安装pip，那就安它提示的命令安装pip后再安装即可。  
！！！！！！  
但实际上这个方法还是没有解决问题。换另一种方法：  
Just install Cython from [ http://cython.org/#download
](http://cython.org/#download) and install it using this command  

    
    
    sudo python setup.py install

Then run the command  

    
    
    sudo python -c 'import Cython.Distutils'

  
and it will be installed and the error message will disappear.  

(晚上有遇到一个类似的问题，用pip安装仍然是fail，仔细看是权限问题，不知道上面那个问题是不是也是权限问题。)

今天再次遇到，简单解决：

    
    
    sudo apt-get install Cython

(但是简单解决的前提不知道是不是因为我提前换了源，换源参考： [
http://blog.csdn.net/u010668907/article/details/50939838
](http://blog.csdn.net/u010668907/article/details/50939838) )  

  

2  .  ImportError: No module named easydict  （类似与上面的错误）

解决：

    
    
    sudo pip install easydict

  

3.ubuntu python安装 sklearn.utils  
解决： if you have Python 2 you can install all these requirements by issuing:

    
    
    sudo apt-get install build-essential python-dev python-setuptools \
    python-numpy python-scipy \
    libatlas-dev libatlas3gf-base
    pip install --user --install-option="--prefix=" -U scikit-learn

或者

    
    
    sudo apt-get install python-sklearn 
    
    
    nosetests -v sklearn 测试sklearn

  
  

4.ubuntu python安装 pandas.io.parsers

解决：

    
    
    sudo apt-get install python-pandas

  

5.ImportError: No module named cv2

解决：

    
    
    sudo apt-get install python-opencv

  

6. ImportError: No module named yaml

    
    
    sudo apt-get install python-yaml

7.lmdb

    
    
    sudo pip install lmdb  

  

其他安装：

[ numpy ](http://blog.csdn.net/u010668907/article/details/50939838)  

[ 点击打开链接 ](http://blog.csdn.net/u010668907/article/details/50295379)

  

  
[ ](http://blog.csdn.net/u010668907/article/details/50295379)

