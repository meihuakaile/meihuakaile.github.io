---
title: faster rcnn源码解读总结
tags: ['faster rcnn源码理解']
categories: ['faster rcnn', 'faster cnn源码理解']
copyright: true
---
1\.  初始数据通过  imdb  类的操作放在它的属性  roidb  里。

2.roidb  只是一个字典，可以拿出来当做一个单独的字典，脱离  imdb  。

3.roi_data_layer  下的  layer  就是  input-data  。  Forward  中加载数据并控制一次一张图片

的数据进入网络。送到  rpn-data  中三组数据：

gt_boxes  ：大小（一张图片  xml  中  box  个数  , 5  ）；一张图中  box  的坐标以及类别

data  ： 大小（  1,3,  高  ,  宽）；一张图的数据

im_info  ： 大小（  1, 3  ）； （高  ,  宽  ,  下面提到的比例）

图片的大小与原图不同，每张图的高或宽被  rescale  成  600  ，另一边会按照相同的比例  rescale
（代码出处未找到，且不懂这样的原因？？？？？？）

4.`AnchorTargetLayer  就是  rpn-data.  计算  anchors,  以及  anchors  是否合理（大小，
overlap  ），并根据每个  anchor  与  gt_box  的重叠度判断  labels  ；  anchors
大小是卷积网络过来数据的高宽再乘  9  个（即，一个点有  9  个）  .  最后产生四组数据（设  k=len(anchors)  ）：

labels  ：大小（  k, 1  ）； 前景  =1  ，背景  =0  ， 否则  =-1

rpn_bbox_targets:  大小  (k, 4)

bbox_inside_weights:  大小（  k, 4  ）  ;  有前景  =1  ，否则为  0

bbox_outside_weights:  大小  (k, 4);  有前景或背景  =1/  （前景  \+  背景），否则为  0

