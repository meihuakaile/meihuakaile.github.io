---
title: python+caffe做图片处理可能会用到的方法
date: "2018/04/08"
tags: ['python', '图片处理']
categories: ['cnn图片数据处理、显示']
copyright: true
---
另外参考我的 [ 这篇 ](/2018/04/08/python图片处理Image和skimage的不同) 和 [ 这篇
](/2018/04/08/python的Image和skimage处理图片) ，了解更多的图片处理。
### 图片和numpy数据的转化
```python
# 可以实现图片和numpy数据的转化。
numpy.array(image)和Image.fromarray(np_data)  
```
### 随机
```python
    #start到end间随机数。start=<num<=end
    random.randint(start, end)
    #list以行随机打乱，用于存入数据库时的txt根据（即，常常看到的train.txt）
    random.shuffle(list)
    #numpy；产生一个size大小的随机numpy，数值在start和end之间
    numpy.random.randint(start, end, size=(height,width))
```
### 矩阵相乘
```python
    numpy.dot(numpyA, numpyB) #两个矩阵相乘
```
不断更新中，总是忘，目前就能记起这些了。  

