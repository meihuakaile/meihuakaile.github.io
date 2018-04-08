---
title: fast-rcnn训练自己数据集以及demo代码解读和总结（面向fast-rcnn初学者）
tags: ['深度学习', 'fast-rcnn']
categories: ['faster rcnn', 'fast rcnn']
copyright: true
---
首先推荐 [ 文章， ](http://www.cnblogs.com/louyihang-loves-baiyan/p/4885659.html)
里面有讲如何安装fast-rcnn，以及编译。

或者我直接把fast-rcnn的地址写出来：https://github.com/rbgirshick/fast-rcnn

  

一.最后的demo.py（地址：https://github.com/rbgirshick/fast-
rcnn/blob/master/tools/demo.py）的代码解读：

  

1.获取参数类型得到训练的类型，找到它的porototxt和model。  
2.net=caffe.Net(c1,c2,c3)得到网络c1是porototxt，c2是model，c3固定。  
3.进入demo（网络，图片名字，检测类别)方法  
3.1soi加载“图片_boxes.mat”,这是用selective search做的预处理文件。  
3.2cv2加载图片。  
3.3fast_rcnn.test.im_detect(c1,c2,c3),c1是net网络，c2是加载后的图片，c3是加载后的预处理文件。但是fast_r
cnn.test不知道是什么。返回的是“分数，boxes”【】  
4.画出结果  
4.1.由程序可知，CLASSES里是根据voc写的，因此顺序不可改变。由类别名称得到索引。  
4.2.把这个索引对应的scores里的分数定义一个阀值，把大于阀值的索引返回（where的语法仍然不对）  
4.3.得到相应boxes的值，并把scores的值和boxes的值水平合并  
4.4.送入画图函数  
5.画图。  

  

总结：  

Fast RCNN中，提取OP的过程和训练过程仍然是分离的。  
在训练过程中，需要用OP的方法先把图像OP提取好，再送入Fast RCNN中训练，  
在检测过程中也是如此需要先把相应的测试图像的OP提取出来送入检测。

阀值很重要，太大会导致有些图识别得到object。太小又会识别太多（不必要的），哈哈，你试试就知道啦。

  

二、直接用它的代码训练自己的数据（简单格式的）：

  

数据准备：

由代码可以看出图片放在了fast-rcnn主目录下的/data/demo下，图片后缀是'jpg';

另外我们还需要一个提取出来的op，即一个后缀名是mat的文件，它和图片放在同一个目录下。这个我们可以用网上的开源代码，selective search。 [
地址 ](https://github.com/sergeyk/selective_search_ijcv_with_python)
使用方法网站上有，如果使用中出错请参考我的另一片文章。

but，其实呢，我最初给的那个博主的可以识别出来很多车的情况，我这是没有的。大概忘了，貌似也就三四个的样子。

估计你还想知道怎么训练自己的model，那个还需要自己先动手表出真正的结果，因为我当时也就试试，没有真正的要用，所以就没用那个 经历去做了。

  

名词解释：ROI : Region of Interest  

