---
title: python+caffe做图片处理可能会用到的方法
tags: ['python', '图片']
categories: ['cnn图片数据处理、显示']
copyright: true
---
1、numpy.array(image)和Image.fromarray(np_data)可以实现图片和numpy数据的转化。  

另外参考我的 [ 这篇 ](http://blog.csdn.net/u010668907/article/details/50993687) 和 [ 这篇
](http://blog.csdn.net/u010668907/article/details/51113235) ，了解更多的图片处理。

  

2、

    
    
    random.randint(start, end)#start到end间随机数。start=<num<=end
    random.shuffle(list)#list以行随机打乱，用于存入数据库时的txt根据（即，常常看到的train.txt）。
    numpy.random.randint(start, end, size=(height,width))#numpy；产生一个size大小的随机numpy，数值在start和end之间。
    

  
  
3、

    
    
    numpy.dot(numpyA, numpyB) #两个矩阵相乘

  

不断更新中，总是忘，目前就能记起这些了。  

