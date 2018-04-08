---
title: python的Image和skimage处理图片
tags: ['python', '图片处理']
categories: ['cnn图片数据处理、显示']
copyright: true
---
一、import  Image

    
    
    img = Image.open(path)#打开图片 
    
    
    img.getpixel((height, width))#得到(height, width)处的像素值（可能是一个list，3通道）
    
    
    img.convert("L")#转灰度图

![](https://img-blog.csdn.net/20160410160028105?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

    
    
    size = (64, 64)
    img.resize(size, Image.ANTIALIAS)#改变尺寸

![](https://img-blog.csdn.net/20160410160625341?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

    
    
     box = (10, 10, 100, 100)
     img.crop(box)#在img上的box处截图

![](https://img-blog.csdn.net/20160410160954837?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

    
    
     img_data = np.array(img)
     for i in xrange(300):
       x = random.randint(0, img_data.shape[0]-1)
       y = random.randint(0, img_data.shape[1]-1)
       img_data[x][y][0] = 255
     img = Image.fromarray(img_data)#加300个噪音,转来转去麻烦可以直接用skimage度图片就不用转了

  
![](https://img-blog.csdn.net/20160410162213947?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

    
    
     img.rotate(90)#图片旋转90

![](https://img-blog.csdn.net/20160410162545310?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

    
    
    img.transpose(Image.FLIP_LEFT_RIGHT)#图片镜像

![](https://img-blog.csdn.net/20160410162800584?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)

二、skimage打开的图片img_data：  

    
    
     from skimage import io,transform
     img_data = io.imread(img_path)
     transform.resize(img_data, (64, 64))#改变尺寸

![](https://img-blog.csdn.net/20160410172405402?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

    
    
    transform.rescale(img_data, 0.5)#缩小/放大图片

![](https://img-blog.csdn.net/20160410172708856?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  
  

  

