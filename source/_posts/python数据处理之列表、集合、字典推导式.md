---
title: python数据处理之列表、集合、字典推导式
tags: ['python', '数据处理']
categories: ['cnn图片数据处理、显示']
copyright: true
---
1.列表：

_ [expr for item in collection if condition] _

举例：

    
    
    >>> result = []
    >>> [result.append(item) for item in fruit if len(item) > 5]
    [None, None]
    >>> result
    ['banana', 'orange']
    

效果与下面类似：  

    
    
    >>> result = []
    >>> fruit = ['apple', 'banana', 'orange']
    >>> for item in fruit:
    ...     if len(item)>5:
    ...         result.append(item)
    ... 
    >>> result
    ['banana', 'orange']
    

  

2.集合：  
_ (expr for item in collection if condition) _ 与列表只有外面括号的差别。

！！！多谢下面的指出，集合外面的括号应该是大括号{}，即{ _ expr for item in collection if condition _ }

  

3.字典：

_ {key : value for item in collectio if condition} _

例子：  

    
    
    >>> fruit = ['apple', 'banana', 'orange']
    >>> dictresult = {}
    >>> dictresult = {key: value for key, value in enumerate(fruit) if len(value) > 5}
    >>> dictresult
    {1: 'banana', 2: 'orange'}
    
    

相同效果：

    
    
    >>> fruit = ['apple', 'banana', 'orange']
    >>> dictresult = {}
    >>> for key, value in enumerate(fruit):
    ...     if len(value) > 5:
    ...         dictresult[key] = value
    ... 
    >>> dictresult
    {1: 'banana', 2: 'orange'}
    

  

