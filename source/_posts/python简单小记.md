---
title: python简单小记
tags: ['python']
categories: ['python基础']
copyright: true
---
我的环境windows，editplus，python-2.7.6。  

1.带参数输出：

    
    
    list = {'zhang','wang','li','zhao'}
    for s in list:
    	print('my first name is {0}'.format(s))
    
    

  
输出结果：

\---------- Python ----------  
my first name is zhao  
my first name is zhang  
my first name is wang  
my first name is li  
  
输出完成 (耗时 0 秒) - 正常终止

  

2.把上面那个例子list的值写成汉字，会报错：

SyntaxError: Non-ASCII character '\xe5' in file jichu.py on line 1, but no
encoding declared; see http://www.python.org/peps/pep-0263.html for details

因为python默认没有汉字，这里要改变编码，在文件开头写上：#coding=utf-8修改结果如下：

    
    
    #coding=utf-8
    list = {'张','王','李','赵'}
    for s in list:
    	print('my first name is {0}'.format(s))
    	

  
输出结果：

\---------- Python ----------  
my first name is 李  
my first name is 赵  
my first name is 张  
my first name is 王  
  
输出完成 (耗时 0 秒) - 正常终止

3.python中的类：

    
    
    class Hello:
    	def __init__ (self,name):
    		self._name = name
    	def say(self):
    		print('hello {0}'.format(self._name))
    	
    class Hai:
    	def __init__ (self,name):
    		hello._init(self,name)
    	def sayHai (self):
    		print('hai {0}'.format(self.name))
    
    h = Hello("zhang san")
    h.say()

  
输出：

\---------- Python ----------  
hello zhang san  
  
输出完成 (耗时 0 秒) - 正常终止

4.引入python文件

法1：

    
    
    import jichu1
    
    h = jichu1.Hello('lisi')
    h.say()
    

  
法2：

    
    
    from jichu1 import Hello
    
    h = Hello('lisi')
    
    h.say()

  
区别显而易见。

只是一些很基础的知识。  

