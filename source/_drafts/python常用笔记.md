---
title: python常用笔记
date: "2018/04/24"
tags: ['python']
categories: ['python']
copyright: true
---
# 浮点数保留位数
三种办法：
round(num, 2)
float('%.2f' % a)
Decimal('5.201').quantize(Decimal('0.00'))
```python

>>> a = 5.21
>>> float('%.1f' % a)
5.2
>>> round(a, 1)
5.2
>>> from decimal import Decimal
>>> Decimal(a).quantize(Decimal('.0'))
Decimal('5.2')
```
# append/extend
这两个方法功能类似,但是在处理多个列表时,这两个方法的处理结果是完全不同的。
append是把第二个list整个做为第一个list的参数；
extend是把第二个list的参数值作为第一个list的参数。
```python
>>> l = [1, 2, 3]
>>> s = [4, 5]
>>> l.append(s)
>>> l
[1, 2, 3, [4, 5]]
>>> l.extend(s)
>>> l
[1, 2, 3, [4, 5], 4, 5]
```
# list.count(item)
计算item在list中出现的次数
# range/xrange
两者的用法一样，`range([start,] stop[, step])`，根据start与stop指定的范围以及step设定的步长，生成一个序列。
但是range是直接返回一个list，xrange是返回一个生成器。两者一般都用在循环里，如果数据很大，xrange每次访问返回一个数值节省内存。
# log
`import math
math.log(x[, base])`