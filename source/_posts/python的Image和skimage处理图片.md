---
title: python的Image和skimage处理图片
date: "2018/09/08"
tags: ['python', '图片处理']
categories: ['cnn图片数据处理、显示']
copyright: true
---
前提
```python
import  Image
```
###  Image
#### 转灰度
    
    img = Image.open(path)#打开图片 
    img.getpixel((height, width))#得到(height, width)处的像素值（可能是一个list，3通道）
    img.convert("L")#转灰度图

![](2.png)
#### 改变尺寸
    
    size = (64, 64)
    img.resize(size, Image.ANTIALIAS)#改变尺寸

![](3.png)  
#### 截图
    
     box = (10, 10, 100, 100)
     img.crop(box)#在img上的box处截图

![](4.png)  
#### 加噪音
    
     img_data = np.array(img)
     for i in xrange(300):
       x = random.randint(0, img_data.shape[0]-1)
       y = random.randint(0, img_data.shape[1]-1)
       img_data[x][y][0] = 255
     img = Image.fromarray(img_data)#加300个噪音,转来转去麻烦可以直接用skimage度图片就不用转了

  
![](5.png)  
#### 图片旋转
    
     img.rotate(90)#图片旋转90

![](6.png)  
#### 图片镜像
    
    img.transpose(Image.FLIP_LEFT_RIGHT)#图片镜像

![](7.png)

### skimage
#### 改变尺寸  
    
     from skimage import io,transform
     img_data = io.imread(img_path)
     transform.resize(img_data, (64, 64))#改变尺寸

![](8.png)  
#### 缩小/放大图片  
    
    transform.rescale(img_data, 0.5)#缩小/放大图片

![](9.png)  
  

  

