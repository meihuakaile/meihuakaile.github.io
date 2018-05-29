---
title: "python+crontab（os）"
tags: ['os']
categories: ['python']
copyright: true
---
使用crontab定时执行python时，os.getcwd() 的返回结果是‘/root’(网上说还可能是'/home')，而不是python当前的目录。（网上说原因是cwd方法返回的是当前线程的路径，而线程是由crontab在执行的，因此，crontab里的文件都是绝对路径。）
如果要使用python脚本当前目录，可以用`os.path.abspath(os.path.dirname(__file__))`

`os.path.dirname(__file__)`  ：
它返回的是脚本执行的路径，所以如果就是在脚本当前路径下执行，返回的结果是空。

`os.path.abspath(file)`:
返回的是py脚本的当前绝对路径。注意返回结果是包括了filename的，因此一般和上面的命令一起只用，这样就真正的得到了它的绝对路径。

`os.chdir（）`可以切换目录
```python
import os
 
path1 = os.path.dirname(__file__)
path2 = os.path.abspath(path1)
 
print "path1: ", path1
print "path2: ", path2
执行：python ./base/test_os_path.py 
结果：
path1:  ./base
path2:  /home/chenliclchen/pycharmProject/register/base
 
执行：python test_os_path.py
结果：
path1: 
path2:  /home/chenliclchen/pycharmProject/register/base
```
如果把abspath的参数也换成‘\_\_file__’， 那两次的执行结果都是”path2:  /home/chenliclchen/pycharmProject/register/base/test_os_path.py“
