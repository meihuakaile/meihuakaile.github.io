---
title: python简单小记
tags: ['python']
categories: ['python']
copyright: true
---
# 集合
{} 字典、对象
[ ]列表、数组
()元组 
增加元素： list用append(obj)、insert(index, obj)，元组不可变，字典通过key直接添加，多次添加会覆盖上次数据
减少： list用pop([index])，元组不可变，字典pop(key)
是否存在：字典key in dict
长度：list用len(list)

\# 元组和列表变成数组 字典变成对象
json.dumps(obj) obj转json
json.loads(json) json转obj

# 引号
python有时会出现三个双引号/单引号， 原因是可以输出字符串的原有样式。
python中单引号的存在是“let‘s go”这样的内容可以直接打印出来。（外单内双/内双外单  都可以）

1.单引号里可以有双引号。内部内容（包括双引号）可以当做字符串。
2.双引号里可以有单引号。内部内容（包括单引号）可以当做字符串。
3.三个单引号/三个双引号。内部内容可以按照内部所有格式输出，结果不变。

eg ：
```python
>>> a = '''hello,'''
>>> a = '''hello,
...     lili'''
>>> print a
hello,
    lili
```

# None
python中的None是空对象的意思；可以将None赋给任意值，也可以将任意值给值是None的对象。

# enumerate
迭代 。 可同时得到下标和数据，有时比较方便：
```python
>>> l = [1, 3, 4, 6]
>>> for ind, item in enumerate(l):
...     print ind, item
...
0 1
1 3
2 4
3 6
```
enumerate还可以有第二个参数start_ind，表示从第几个下标开始。

# pass
和java里标注的 “do something” 功能一样。 先把位置站住，为之后再填代码。
也有人为了代码块里没有东西  防止不出错。

# mysql
安装：`sudo apt-get install python-mysqldb`
**_注意sql语句中的字符串一定要加上引号，如下面（2）/（3）的%s外面的引号。！！！！！否则无法识别成字符串_**

## sql语句
（1）`sql = """insert into user(email, name, password) VALUES ("""+str(email)+"," + str(name) + "," + str(password) + ")"`
（2） `sql = 'insert into user(email, name, password) VALUES ("%s", "%s", "%s")'
    cursor.execute(sql % (email, name, password))`
（3）`sql = "insert into user(email, name, password) VALUES ('%s', '%s', '%s')" % (email, name, password)`

## 数据库连接
```python
self._connecter = MySQLdb.Connect("localhost", "chenliclchen", "024", "test")
self._sql_cursor = self._connecter.cursor()
```
connect的参数是 主机名  数据库用户名  数据库用户密码  数据库名

## 简单操作
```python
try:
    self._sql_cursor.execute(sql)
    self._connecter.commit()
except:
    logger.exception('exception')
    self._connecter.rollback()
 
self._connecter.close()
```
增删改都可用上面的语法，查：
```python
try:
    self._sql_cursor.execute(sql)
    result = self._sql_cursor.fetchall()
except:
    logger.exception("exception")
    self._connecter.rollback()
 
self._connecter.close()
```
fetchone(): 该方法获取下一个查询结果集。结果集是一个对象
fetchall():接收全部的返回结果行.

# logging
python的logging默认是warn级别。

#### 终端配置
```python
logger = logging.getLogger("register")
console = logging.StreamHandler()
console.setLevel(logging.INFO)
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
console.setFormatter(formatter)
logger.addHandler(console)
```
logging.getLogger(name) name为空时默认是root。为name为register的logger配置之后，其他的代码也用register的logger时配置一样。
#### 打印到文件配置
```python
logging.basicConfig(level=logging.INFO,
format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
datefmt='%a, %d %b %Y %H:%M:%S',
filename='register.log')
```
#### logging.basicConfig参数
`filename`: 指定日志文件名
`filemode`: 和file函数意义相同，指定日志文件的打开模式，'w'或'a'
`format`: 指定输出的格式和内容，format可以输出很多有用信息，如上例所示:
`%(levelno)s` : 打印日志级别的数值
`%(levelname)s`: 打印日志级别名称
`%(pathname)s`: 打印当前执行程序的路径，其实就是sys.argv[0]
`%(filename)s`: 打印当前执行程序名
`%(funcName)s`: 打印日志的当前函数
`%(lineno)d`: 打印日志的当前行号
`%(asctime)s`: 打印日志的时间
`%(thread)d`: 打印线程ID
`%(threadName)s`: 打印线程名称
`%(process)d`: 打印进程ID
`%(message)s`: 打印日志信息
`datefmt`: 指定时间格式，同time.strftime()
`level`: 设置日志级别，默认为logging.WARNING
`stream`: 指定将日志的输出流，可以指定输出到sys.stderr,sys.stdout或者文件，默认输出到sys.stderr，当stream和filename同时指定时，stream被忽略

#### logging带参数输出
（1）`logger.info("%(sql)s %(email)s %(name)s %(password)s", {'sql': sql, 'email': email, 'name': name, 'password': password})
         logger.debug(msg [ ,*args [, **kwargs]])
        logger.info("%s %s %s %s", *(sql, email, name, password))`
（2）`logger.log(lvl, msg[ , *args[ , **kwargs]] )` 级别作为参数
（3）`logger.exception('exception')` 会自动把异常写到logger的日志中去。

# with
#### 优雅关闭文件
在打开文件时，如果出现异常，文件可能会忘记关闭。如，
```python
file_object = open(html_file)
body_data = file_object.read()
```
一般的做法是用try包起来，然后在finally里关闭，因为finally是无论如何都会执行的：
```python
try:
    try:
        file_object = open(html_file)
        body_data = file_object.read()
    except IOError:
        log.write('no data read\n')
finally
    file_object.close()
```
但是这样的做法不够优雅。因此考虑用with：
```python
with open(html_file) as file_object:
    body_data = file_object.read()
```
这样就不用考虑忘记关闭文件了。如果with语句或语句块中发生异常，会调用默认的异常处理器处理，但文件还是会正常关闭。

#### 支持本协议的对象
with语句仅仅能对支持上下文管理协议的对象使用。支持本协议的对象有：

file
decimal.Context
thread.LockType
threading.Lock
threading.RLock
threading.Condition
threading.Semaphore
threading.BoundedSemaphore
#### 原理，执行过程
（1）、当with语句执行时，便执行上下文表达式(一般为某个方法，例如上面的open方法)来获得一个上下文管理器对象，上下文管理器的职责是提供一个上下文对象，用于在with语句块中处理细节。
（2）、一旦获得了上下文对象，就会调用它的\_\_enter\_\_()方法，将完成with语句块执行前的所有准备工作，如果with语句后面跟了as语句，则用__enter__()方法的返回值来赋值。
（3）、当with语句块结束时，无论是正常结束，还是由于异常，都会调用上下文对象的\_\_exit\_\_()方法，\_\_exit\_\_()方法有3个参数，如果with语句正常结束，三个参数全部都是 None；如果发生异常，三个参数的值分别等于调用sys.exc\_info()函数返回的三个值：类型（异常类）、值（异常实例）和跟踪记录（traceback），相应的跟踪记录对象。
（4）、因为上下文管理器主要作用于共享资源，\_\_enter\_\_()和\_\_exit\_\_()方法基本是完成的是分配和释放资源的低层次工作，比如：数据库连接、锁分配、信号量加/减、状态管理、文件打开/关闭、异常处理等。

#### 自己定义with
```python
class A(object):
    def __enter__(self):
        print('__enter__() called')
        return self
 
    def print_hello(self):
        print("hello world!")
 
    def __exit__(self, e_t, e_v, t_b):
        print('__exit__() called')
 
# 首先会执行__enter__方法
with A() as a:  # a为__enter__的返回对象
    a.print_hello()
    print('got instance')
    # 结束会执行__exit__方法

执行输出：
__enter__() called
hello world!
got instance
__exit__() called
```
参考：https://www.cnblogs.com/skiler/p/6958344.html

# \_\_repr\_\_和\_\_str\_\_
```python
>>> class Test(object):
...     def __init__(self, value='hello, world'):
...         self.data = value
...
>>>
>>> t = Test()
>>> t
<__main__.Test object at 0x7f136896e250>
>>> print t
<__main__.Test object at 0x7f136896e250>
```
\# 看到了么？上面打印类对象并不是很友好，显示的是对象的内存地址
\# 下面我们重构下该类的__repr__以及__str__，看看它们俩有啥区别
```python
# 重构__repr__
>>> class TestRepr(Test):
...     def __repr__(self):
...         return "repr"
...
>>>
>>> tr = TestRepr()
>>> tr
repr
>>> print tr
repr
```
\# 重构__repr__方法后，不管直接输出对象还是通过print打印的信息都按我们__repr__方法中定义的格式进行显示了
```python
# 重构__str__
>>> class TestStr(Test):
...     def __str__(self):
...         return "str"
...
>>>
>>> ts = TestStr()
>>> ts
<__main__.TestStr object at 0x7f136896e590>
>>> print ts
str
```
\# 你会发现，直接输出对象ts时并没有按我们__str__方法中定义的格式进行输出，而用print输出的信息却改变了
```python
>>> class TestStr(Test):
...     def __str__(self):
...         return "str"
...     def __repr__(self):
...         return "repr"
...
>>> ts = TestStr()
>>> ts
repr
>>> print ts
str
#  __repr__用于交互模式下提示回应；print输出会先找str
```
明显的区别是，在终端里repr可以使响应输出和print输出的结果一样；str在响应输出仍然是类的地址，print可以按照__str__的定义输出。
**_print情况下，先找str函数，没有实现再使用repr函数；终端交互响应时，会先找repr函数，没有再找str函数。_**

# datetime
`def strftime(format, p_tuple=None)/strftime(format）`  time转string
`datetime.strptime(date_string, format)` string转time
`timedelta()`  时间计算

```python
# -*- coding:utf-8 -*-
# __author__='chenliclchen'

import time
from datetime import datetime, timedelta

# 输入当前时间，正数部分是秒，小数部分是毫秒。（java是正数部分是毫秒，python×1000 = java）
print time.time()

# time.struct_time(tm_year=2017, tm_mon=10, tm_mday=17, tm_hour=10, tm_min=22, tm_sec=54, tm_wday=1, tm_yday=290, tm_isdst=0)
# 年 月 日 小时 分钟 秒 一周第几天（0-6） 一年第几天 夏时令
nowtime = time.localtime(time.time())
print nowtime

# Tue Oct 17 10:34:13 2017
retime = time.asctime(nowtime)
print retime

# 时间转字符串
# 2017-10-17 10:38:09
time_format1 = "%Y-%m-%d %H:%M:%S"
print time.strftime(time_format1, nowtime)

# time.strftime(format)
# Tue Oct 17 10:40:56 2017
time_format2 = "%a %b %d %H:%M:%S %Y"
a = time.strftime(time_format2, nowtime)
print a

# 将字符串转成时间戳
# time.struct_time(tm_year=2017, tm_mon=10, tm_mday=17, tm_hour=10, tm_min=43, tm_sec=5, tm_wday=1, tm_yday=290, tm_isdst=-1)
print time.strptime(a, time_format2)

# 1508208272.0(1970~2038)
print time.mktime(time.strptime(a, time_format2))

# datetime.now(): 2017-10-17 10:50:05.383724 2017
print "datetime.now(): ", datetime.now(), datetime.now().year
# datetime.strftime(format)
# datetime format 2017-10-17 10:57:44 <type 'str'>
to_string = datetime.strftime(datetime.now(), time_format1)
to_string = datetime.now().strftime(time_format1)
print "datetime to string: ", to_string, type(to_string)
# datetime.strptime(date_string, format)
# datetime totime: 2017-10-17 10:57:44 <type 'datetime.datetime'>
to_date = datetime.strptime("2017-10-17 10:57:44", time_format1)
print "datetime_string to datetime: ", to_date, type(to_date)

tmp = datetime.strptime("2017-10-16", "%Y-%m-%d")
print "tmp, tmp.isoweekday(): ", tmp, tmp.isoweekday()
last_time = tmp + timedelta(days=-1)
print "last_time", last_time
print datetime.strftime(last_time, "%Y-%m-%d")
```
https://docs.python.org/3/library/datetime.html
# unicode-escape、string-escape
unicode-escape编码集，它是将unicode内存编码值直接存储。
如果输出中出现‘\u’，如‘\u9152\u5e97’ 这样的就需要先用它进行解码，在用utf-8编码。如：
```python
l = ['12', u'\u9152\u5e97\u53d1\u5e03\u6545\u969c']
print str(l).decode('unicode-escape').encode('utf-8')
```
下面的两种方法效果一样：
```python
ll = u'\u9152\u5e97\u53d1\u5e03\u6545\u969c'
print ll.encode('utf-8')
```
```python
ll = '\u9152\u5e97\u53d1\u5e03\u6545\u969c'
print ll.decode('unicode-escape').encode('utf-8')
```
string-escape编码集，可以对字节流用string-escape进行编码。
目前理解的是出现‘\x’时，如'\xe9\x85\x92\xe5\xba\x97’这样的就需要用此数据集解码就可以直接显示。如：
`print '\xe9\x85\x92\xe5\xba\x97'.decode('string-escape')`
