---
title: python图片处理Image和skimage的不同
tags: ['python', '图像处理']
categories: ['cnn图片数据处理、显示']
copyright: true
---
做cnn的难免要做大量的图片处理。由于接手项目时间不长，且是新项目，前段时间写代码都很赶，现在稍微总结（恩，总结是个好习惯）。

1,首先安装python-Image和python-skimage、python-matplotlib。

简单代码：

    
    
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
    

调用及输出：

![](https://img-blog.csdn.net/20160327205827457?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

其实Image读出来的是PIL什么的类型，而skimage.io读出来的数据是numpy格式的。如果想直接看Image和skimage读出来图片的区别，可以
直接输出它们读图片以后的返回结果。  

2.Image和skimage读图片：

    
    
    img_file1 = img.open(args.picpath)
    img_file2 = io.imread(args.picpath)

3.读图片后数据的大小：

    
    
    print "the picture's size: ", img_file1.size
    print "the picture's shape: ", img_file2.shape

4.得到像素：

    
    
    one_pixel = img_file1.getpixel((0,0))[0]
    img_file2[0][0][0]

  
分析：  

1.从3的输出可以看出img读图片的大小是图片的（height,width）；

skimage的是(height,width, channel)[这也是为什么caffe在单独测试时要要在代码中设置：  transformer  .
set_transpose  (  'data'  ,  (  2  ,  0  ,  1  ))
，因为caffe可以处理的图片的数据格式是(channel,height,width)，所以要转换数据啊]

2.img读出来的图片获得某点像素用getpixel((h,w))可以直接返回这个点三个通道的像素值

skimage读出来的图片可以直接img_file2[0][0][0]获得，但是一定记住它的格式，并不是你想的(channel,height,width)

  

关于matplotlib简单的画图请关注下篇～  

  

