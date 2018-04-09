---
title: python图片处理Image和skimage的不同
date: "2018/04/08"
tags: ['skimage', '图片处理']
categories: ['cnn图片数据处理、显示']
copyright: true
---
做cnn的难免要做大量的图片处理。由于接手项目时间不长，且是新项目，前段时间写代码都很赶，现在稍微总结（恩，总结是个好习惯）。

首先安装python-Image和python-skimage、python-matplotlib。

#### 简单例子
```python
    import Image as img
    import os
    from matplotlib import pyplot as plot
    from skimage import io,transform
    import argparse
    
    def show_data(data):
        fig = plot.figure()
        ax = fig.add_subplot(121)
        ax.imshow(data, cmap='gray')
        ax2 = fig.add_subplot(122)
        ax2.imshow(data)
        plot.show()
    if __name__ == "__main__":
        parse = argparse.ArgumentParser()
        parse.add_argument('--picpath', help = "the picture' path")
        args = parse.parse_args()
        img_file1 = img.open(args.picpath)#Image读图片
        one_pixel = img_file1.getpixel((0,0))[0]
        print "picture's first pixe: ",one_pixel  
        print "the picture's size: ", img_file1.size#Image读出来的size是高宽
        show_data(img_file1)
        img_file2 = io.imread(args.picpath)#skimage读图片
        show_data(img_file2)
        print "picture's first pixel: ", img_file2[0][0][0]
        print "the picture's shape: ", img_file2.shape#skimage读出来的shape是高，宽， 通道
```
调用及输出：

![](/images/1.png)  

Image读出来的是PIL什么的类型，
skimage.io读出来的数据是numpy的格式。
想看Image和skimage读出来图片的区别，可以直接输出它们读图片以后的返回结果。  

#### Image和skimage读图片
```python
    img_file1 = img.open(args.picpath)
    img_file2 = io.imread(args.picpath)
```
#### 读图片后数据的大小
```python
    print "the picture's size: ", img_file1.size
    print "the picture's shape: ", img_file2.shape
```
从输出可以看出
img读图片的大小是图片的（height,width）；
skimage的是(height,width, channel)
[这也是为什么caffe在单独测试时要要在代码中设置：transformer.set_transpose  ('data',  (  2, 0, 1))，因为caffe可以处理的图片的数据格式是(channel,height,width)，所以要转换数据啊]
#### 得到像素
```python
    one_pixel = img_file1.getpixel((0,0))[0]
    img_file2[0][0][0]
```
img读出来的图片获得某点像素用getpixel((h,w))可以直接返回这个点三个通道的像素值
skimage读出来的图片直接img_file2[0][0][0]获得某点某通道像素值，但是一定记住它的格式，并不是你想的(channel,height,width)

  

关于matplotlib简单的画图请关注下篇～  

  

