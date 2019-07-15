---
title: faster rcnn源码解读总结
date: "2018/04/08"
tags: ['深度学习', 'faster rcnn源码理解']
categories: ['faster rcnn', 'faster cnn源码理解']
copyright: true
---
1\. 初始数据通过imdb类的操作放在它的属性roidb里。

2\. roidb只是一个字典，可以拿出来当做一个单独的字典，脱离imdb  。

3\. roi_data_layer下的layer就是input-data。Forward中加载数据并控制一次一张图片的数据进入网络。送到rpn-data中三组数据：
* gt_boxes：大小（一张图片的xml中box个数, 5）；一张图中box的坐标以及类别
* data：大小（1, 3, 高, 宽）；一张图的数据
* im_info：大小（1, 3）；（高, 宽, 下面提到的比例）

图片的大小与原图不同，每张图的高或宽被 rescale 成 600，另一边会按照相同的比例 rescale（代码出处未找到，且不懂这样的原因？？？？？？）

4\. AnchorTargetLayer 就是rpn-data.计算 anchors,以及anchors是否合理（大小，overlap），并根据每个anchor与gt_box的重叠度判断labels；
anchors大小是卷积网络过来数据的高宽再乘9个（即，一个点有9个）.
最后产生四组数据（设k=len(anchors)）：
* labels：大小（k, 1）； 前景=1，背景=0，否则=-1
* rpn_bbox_targets:  大小(k, 4)
* bbox_inside_weights:  大小（k, 4）; 有前景=1，否则为0
* bbox_outside_weights:  大小 (k, 4); 有前景或背景=1/（前景+背景），否则为0

5\. 区别 
* cfg.TRIAIN.IMS_PER_BATCH: 是训练proposals时的batch size, 在mini中被默认为1不能修改。
* cfg.TRAIN.BATCH_SIZE: 是后面基于proposal训练的时候的batch size
* cfg.TRAIN.RPN_BATCHSIZE: 是控制第一步训练的结果proposals的个数，在AnchorTargetLayer中被定义为256（如果想修改proposal个数可以修改）。