---
title: numpy常用方法
date: "2018/04/08"
tags: ['numpy', '数据分析']
categories: ['cnn图片数据处理、显示']
copyright: true
top: 4
---
  
一般参数有axis时，<strong>0是列； 1是行<strong>
假设 ：  
```python
import scipy.io as scio
import operator
import numpy as np
```
#####  求平均值 mean
```python
    array1.mean(axis=0) == numpy.mean(array1, axis=0)
    
    Examples
    --------
    >>> a = np.array([[1, 2], [3, 4]])
    >>> np.mean(a)
    2.5
    >>> np.mean(a, axis=0)
    array([ 2., 3.])
    >>> np.mean(a, axis=1)
    array([ 1.5, 3.5])
```
axis=0 对 列求平均值。  
  

#####  求方差 std
```python
array1.std(axis=0) == numpy.std(array1, axis=0)
```
axis的意义同1，求方差。  

##### numpy scipy pandas 区别

NumPy 是基础的 数学 计算库，包括 基本的四则运行，方程式 计算，微积分 什么的，还有很多其他数学方面的计算，我也不是很清楚  
SciPy 是在NumPy基础上，封装了一层，没有那么纯数学，提供方法直接计算结果  
Pandas 是上层做数据分析用的，主要是做表格数据呈现  
如果不是纯数学专业还是从 Pandas 入手比较好。  

#####  读.mat文件 loadmat
```python
scio.loadmat(train_data_file) 
# return is a dict
```
#####  求几次方 **
```python
diff ** 2 
# 2次方，2换成0.5就是开方 diff 可数字和数组
```
#####  数组求和 sum
```python
diff.sum(axis=1)
# 求数组的和， axis的值同上 0是列； 1是行； diff 数组
```
#####  排序下标 argsort
```python
diff.argsort(axis=1)
 # diff数组从小到大数据的下标 axis同上，默认是1
examples：
>>> a = np.array([[1, 2], [3, 4]])
>>> a.argsort()
array([[0, 1],
[0, 1]])
# 输出数组行从小到大排序的原始数据坐标；
# 如，index=0行的1,2 从小到大排序还是 1 2 ，1原来数组的下标是0; 2原来数组的下标是1 ； 因此返回数据的第一行是0 1
```
#####  dict get
```python
class_count.get(label, 0) 
# class_count 是一个 dict 。 从class_count找key = label对应的value，找到就返回value，找不到就返回0。
```
#####  dict排序 sorted
```python
sorted_class_count = sorted(class_count.iteritems(), key=operator.itemgetter(1), reverse=True）
# 对字典的value排序，返回一个list，list里包含很多元组，这些元组就是之前dict的key-value对儿。
# class_count 是一个 dict 。 reverse=True致使value是从大到小的顺序。
# sorted_class_count[][]
```
#####  统计次数 bincount
```python
>>> a = np.array([1, 2, 3, 2])
>>> np.bincount(a)
array([0, 1, 2, 1])
# 对数组中数据统计次数；返回数组下标是原始数组的值，返回数组的数据值是次数。例如上面数组a值1出现的次数是1次，因此返回数组index=1的值为1，index=2的值为2
# 输入数组使用中发现的约束条件：一维数组、 必须是int整形。（个人理解，更适用于数组数据比较集中且从0/1开始的数）
```
#####  np.random.uniform（start， end[， size]）

生成一个数值<strong>均匀分布</strong>在start，end间的长度大小是size的数组（ndarray类型）。 注意是>= start && < end ，前闭后开。 size默认是1 

#####  np的flatten() vs ravel()

两者都是把多维矩阵铺平，以行为主。区别是flatten返回的是原矩阵的拷贝；ravel是返回的是原矩阵的一种变换视图，如果对返回值修改原矩阵也会跟着变化。  
```python
>>> a
array([[1, 2, 3],
[3, 2, 2],
[3, 4, 5],
[2, 3, 2]])
>>> b = a.flatten()
>>> b[0] = 0
>>> a #原矩阵没有变化
array([[1, 2, 3],
[3, 2, 2],
[3, 4, 5],
[2, 3, 2]])
>>> c = a.ravel()
>>> c[0] = 0
>>> a #原矩阵变化
array([[0, 2, 3],
[3, 2, 2],
[3, 4, 5],
[2, 3, 2]])
```
##### np.mgrid 生成网格
np.mgrid[start: end: size/seperate]原本是生成一个[start, end)的表格，分割大小是seperate；第三个参数后面有j时表示生成一个[start, end]的表格，表格大小是size：
```python
>>> np.mgrid[-10:10:5j] # 有‘j’。生成大小是5的分布在[-10, 10]的表格
array([-10., -5., 0., 5., 10.])
>>> np.mgrid[-10:10:5] # 用5分割[-10, 10)， 生成一个表格。
array([-10, -5, 0, 5])
>>> arr1, arr2 = np.mgrid[-10:10:5, -10:10:5j] # arr1，arr2铺平 再stack ，可以很好的作为二位坐标数据。
>>> arr1
array([[-10., -10., -10., -10., -10.],
[ -5., -5., -5., -5., -5.],
[ 0., 0., 0., 0., 0.],
[ 5., 5., 5., 5., 5.]])
>>> arr2
array([[-10., -5., 0., 5., 10.],
[-10., -5., 0., 5., 10.],
[-10., -5., 0., 5., 10.],
[-10., -5., 0., 5., 10.]])
```
#####  np.amin/amax == np.min/max

#####  矩阵拼接np.vstack(tuple) np.hstack(tuple) np.concatenate(tuple) np.stack()

tuple是一个arrays，就是由多个矩阵组成。  
hstack(tuple) 是把多个矩阵以行拼接，等同于np.concatenate(tuple, axis=1)  
vstack(tuple) 是把多个矩阵以列拼接，等同于np.concatenate(tuple, axis=0)  
```python
>>> a = np.array([[1,2,3], [2,3,4]])
>>> b = np.array([[2,3,9], [2,6,4]])
>>> np.hstack((a, b))
array([[1, 2, 3, 2, 3, 9],
[2, 3, 4, 2, 6, 4]])
>>> np.concatenate((a,b), axis = 1)
array([[1, 2, 3, 2, 3, 9],
[2, 3, 4, 2, 6, 4]])
    
>>> np.vstack((a, b))
array([[1, 2, 3],
[2, 3, 4],
[2, 3, 9],
[2, 6, 4]])
>>> np.concatenate((a,b), axis = 0)
array([[1, 2, 3],
[2, 3, 4],
[2, 3, 9],
[2, 6, 4]])
    
    
>>> np.stack((a, b), axis=0)
array([[[1, 2, 3],
[2, 3, 4]],
    
[[2, 3, 9],
[2, 6, 4]]])
>>> np.stack((a, b), axis=1)
array([[[1, 2, 3],
[2, 3, 9]],
    
[[2, 3, 4],
[2, 6, 4]]])
```
##### np.transpose(元组) 和 T

transpose适用于多维数组，它依赖与参数 元组 ，元组依赖与 shape。  
T用于一/二维数组。  

##### :, 代表一个维度的切片
eg：
```python
img = io.imread(img_path)
data = np.zeros((3, img.shape[0], img.shape[1]), dtype=np.uint8)
data[0] = img[:, :, 0]
data[1] = img[:, :, 1]
data[2] = img[:, :, 2]
```