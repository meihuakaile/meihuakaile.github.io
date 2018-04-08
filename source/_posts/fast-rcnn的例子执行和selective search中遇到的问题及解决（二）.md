---
title: fast-rcnn的例子执行和selective search中遇到的问题及解决（二）
tags: ['fast rcnn', 'selective search']
categories: ['faster rcnn', 'fast rcnn问题解决&&安装']
copyright: true
---
1.出现了EnvironmentError: MATLAB command 'matlab' not found.Please add 'matlab'
to yourPATH.这种错误，说明没把matlab的路径添加到环境变量中，  
解决：下面的语句设置环境变量：  
export PATH=$PATH:"/home/cl/install/MATLAB/bin"  

  

2.这个问题在selective search的执行过程中出的问题。github上的那个用python调用MATLAB的例子。在执行那句调用MATLAB时报
的错。问题是你的MATLAB没起来。解决方案如下。  

问题：
```python
selective_search_rcnn({'/home/cl/examples/images/00001.jpg'},
'/tmp/tmpZ0B5zO.mat')

Traceback (most recent call last):  
File "../python/detect.py", line 168, in  
main(sys.argv)  
File "../python/detect.py", line 139, in main  
detections = detector.detect_selective_search(inputs)  
File "/home/hank/Projects/caffe/python/caffe/detector.py", line 119, in
detect_selective_search  
cmd='selective_search_rcnn'  
File
"/usr/lib/python2.7/selective_search_ijcv_with_python/selective_search.py",
line 39, in get_windows  
shlex.split(mc), stdout=open('/dev/null', 'w'), cwd=script_dirname)  
File "/usr/lib/python2.7/subprocess.py", line 710, in init  
errread, errwrite)  
File "/usr/lib/python2.7/subprocess.py", line 1327, in _execute_child  
raise child_exception  
OSError: [Errno 2] No such file or directory  
```
解决：在~/.bashrc中添加你的matlab的bin路径（例如，我的是export
PATH=$PATH:"/home/cl/install/Matlab/bin"）。我记得source是没有用的，需要重启才可以  
  

