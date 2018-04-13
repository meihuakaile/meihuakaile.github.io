---
title: 初级cnn研究辅助：python的matplotlib显示图片
date: "2018/04/08"
tags: ['python', '图片处理']
categories: ['cnn图片数据处理、显示']
copyright: true
top: 5
---
### 简单例子
```python
    # -*- coding=UTF-8 -*-
    import Image
    from matplotlib import pyplot as plt
    if __name__ == "__main__":
        img = Image.open("./Alex.jpg")
        img_gray = img.convert("L")
        fig = plt.figure()
        ax = fig.add_subplot(121)
        ax.imshow(img)
        ax = fig.add_subplot(122)
        ax.imshow(img_gray, cmap="gray")#以灰度图显示图片
        ax.set_title("hei,i'am the title")#给图片加titile
        #plt.axis("off")#不显示刻度
        plt.show()#显示刚才所画的所有操作
```
图片的其他处理，可以查看我的前几篇文章。

### add_subplot图片位置
add_subplot的参数由三个数字组成mnq。代表画布里有m行，n列位置，当前图片将要放在q位置。
q的计算方式以行为主：如果是四张图且显示是一个2*2的矩阵，q的排序是：
```
1   2
3   4
```
以上面代码为例，
    
    ax = fig.add_subplot(121)

里的121.第一个“1”代表画布只有一行；第一个“2”代表有两列；第二个“1”代表图片将放在1行2列的矩阵中的位置。



当然还会出现这样的需求，左边一张图右边两张图：
```python
    # -*- coding=UTF-8 -*-
    import Image
    from matplotlib import pyplot as plt
    if __name__ == "__main__":
        img = Image.open("./Alex.jpg")
        img_gray = img.convert("L")
        fig = plt.figure()
        ax = fig.add_subplot(121)
        ax.imshow(img)
        ax.set_title("hei,i'am the first")
    
        ax = fig.add_subplot(222)
        ax.imshow(img_gray, cmap="gray")#以灰度图显示图片
        ax.set_title("hei,i'am the second")#给图片加titile
    
        ax = fig.add_subplot(224)
        ax.imshow(img_gray, cmap="gray")#以灰度图显示图片
        ax.set_title("hei,i'am the third")#给图片加titile
        #plt.axis("off")#不显示刻度
        plt.show()#显示刚才所画的所有操作
```
效果：

![](10.png)

### 框出部分区域
```python
    # -*- coding=UTF-8 -*-
    import Image
    from matplotlib import pyplot as plt
    if __name__ == "__main__":
        img = Image.open("./Alex.jpg")
        img_gray = img.convert("L")
        fig = plt.figure()
        ax = fig.add_subplot(121)
        ax.imshow(img)
        ax.set_title("hei,i'am the first")
        pointx = [20, 120, 120, 20, 20]
        pointy = [20, 20, 120, 120, 20]
        ax.plot(pointx, pointy, 'r')#画一个矩形，黑色；'r'红色
```
效果：

![](11.png)

### 画点
```python
    # -*- coding=UTF-8 -*-
    import Image
    from matplotlib import pyplot as plt
    if __name__ == "__main__":
        img = Image.open("./Alex.jpg")
        img_gray = img.convert("L")
        fig = plt.figure()
        ax = fig.add_subplot(121)
        ax.imshow(img)
        ax.set_title("hei,i'am the first")
        pointx = [20, 120, 120, 20, 20]
        pointy = [20, 20, 120, 120, 20]
        ax.plot(pointx, pointy, 'r')#画一个矩形，黑色；'r'红色
        ax.scatter(65, 70)#画点
        ax.scatter(90, 70)#画点
        plt.axis("off")#不显示刻度
```
效果：

![](12.png)

（额，我怎么把我男神搞成这个样子了。。。。）  

