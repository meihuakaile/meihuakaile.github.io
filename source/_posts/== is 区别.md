---
title: '== is 区别'
date: "2018/05/24"
tags: [python]
categories: [python]
copyright: true
---
== 比较的是数值，自定义对象由eq方法决定； is比较的是 地址。

### 基础数据
例1：
```python
>>> a = 400
>>> b = 400
>>> a == b
True
>>> a is b
False
```
```python
>>> c = 3
>>> d = 3
>>> c == d
True
>>> c is d
True
```
例1的结果推测， == 比较的是数值； is比较的是 地址。 
可是例2的结果仿佛推翻这个结论。但其实并没有。
造成例2结果的原因是python的垃圾回收。python对数值在【-5， 256】的数建立了一个对象池，所有在这个范围里的相同数指向的都是同一个对象。因此当数值为3时， is的结果也是true。

### 对自定义对象

```python
class Person(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age
 
p1 = Person("lili", 23)
p2 = Person("lili", 23)
print id(p1)
print id(p2)
print '== ', p1 == p2
print 'is', p1 is p2
输出：
140352009155664
140352008229072
==  False
is False
 

class Person(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def __eq__(self, other):
        return self.name == other.name and self.age == other.age
 
p1 = Person("lili", 23)
p2 = Person("lili", 23)
print id(p1)
print id(p2)
print '== ', p1 == p2
print 'is', p1 is p2
输出：
139796538532944
139796537606416
==  True
is False
```
结论： == 的结果由对象的__eq__方法的定义决定；is判断内存地址。