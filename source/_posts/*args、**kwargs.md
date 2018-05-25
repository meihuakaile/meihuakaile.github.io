---
title: "*args、**kwargs"
date: "2018/05/24"
tags: [args]
categories: [python基础]
copyright: true
---
python2.7

1、两者都是接受可变参数的。\*args是接收数值，\*\*kwargs是接收数值对儿。即前者接收的是元组，后者接收的字典。
2、在定义时顺序必须是  普通参数、\*args、\*\*kwargs。
3、args、kwargs只是约定俗成的名字，还可以起成其他名字。
```python
def test(**kwargs):
    print "test:", kwargs, type(kwargs)
 
def test1(*args):
    print "test1:", args, type(args)
 
def test2(a, b, *args):
    print "test2: ", args, a, b
 
def test3(*args, **kwargs):
    print "test1:", args, type(args), kwargs, type(kwargs)
 
test1(1, 3, 2)
test2(1, 2, 3)
test(a=1, b=2, c=3)
test3(1, 2, a=3, b=4)
 
输出：
test1: (1, 3, 2) <type 'tuple'>
test2:  (3,) 1 2
test: {'a': 1, 'c': 3, 'b': 2} <type 'dict'>
test1: (1, 2) <type 'tuple'> {'a': 3, 'b': 4} <type 'dict'>
```