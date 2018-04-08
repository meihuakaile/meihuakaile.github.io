---
title: 初级cnn研究辅助：python的matplotlib显示图片
tags: ['图片', 'cnn']
categories: ['cnn图片数据处理、显示']
copyright: true
---
一、简单例子：

    
    
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

图片的其他处理，可以查看我的前几篇文章。

二、简单说一下

    
    
    ax = fig.add_subplot(121)

里的121.第一个“1”代表图片只有一行；第一个“2”代表有两列；第二个“1”代表第一张图片在1行2列的矩阵中的位置。

如果是一个2*2的矩阵，第三个数字的排序是：

1   2

3   4

即，以行为主

当然还会出现这样的需求：

    
    
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

效果：

![](https://img-blog.csdn.net/20160410200912716?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)

三、需求：在图中框出你想要的区域：

    
    
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

效果：

![](https://img-blog.csdn.net/20160410202101455?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)

四、或者你想要画点：

    
    
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

效果：

![](https://img-blog.csdn.net/20160410202859070?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)

（额，我怎么把我男神搞成这个样子了。。。。）  

