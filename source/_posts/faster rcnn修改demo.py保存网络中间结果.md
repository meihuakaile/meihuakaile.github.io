---
title: faster rcnn修改demo.py保存网络中间结果
date: "2018/04/08"
tags: ['深度学习', 'faster rcnn中间层显示']
categories: ['faster rcnn', 'faster cnn源码理解']
copyright: true
---
faster rcnn用python版本 [ https://github.com/rbgirshick/py-faster-rcnn
](https://github.com/rbgirshick/py-faster-rcnn)  

以demo.py中默认网络VGG16.

原本demo.py地址 [ https://github.com/rbgirshick/py-faster-
rcnn/blob/master/tools/demo.py ](https://github.com/rbgirshick/py-faster-
rcnn/blob/master/tools/demo.py)

### 样例
图有点多，贴一个图的部分结果出来：

![](17.jpeg)  

上图是原图；
下面第一张是网络中命名为“conv1_1”的结果图；
第二张是命名为“rpn_cls_prob_reshape”的结果图；
第三张是“rpnoutput”的结果图

![](18.jpeg) ![](19.jpeg) ![](20.jpeg)

### 修改后的代码
看一下我修改后的代码：
```python
    #!/usr/bin/env python
    
    # --------------------------------------------------------
    # Faster R-CNN
    # Copyright (c) 2015 Microsoft
    # Licensed under The MIT License [see LICENSE for details]
    # Written by Ross Girshick
    # --------------------------------------------------------
    
    """
    Demo script showing detections in sample images.
    
    See README.md for installation instructions before running.
    """
    
    import _init_paths
    from fast_rcnn.config import cfg
    from fast_rcnn.test import im_detect
    from fast_rcnn.nms_wrapper import nms
    from utils.timer import Timer
    import matplotlib.pyplot as plt
    import numpy as np
    import scipy.io as sio
    import caffe, os, sys, cv2
    import argparse
    import math
    
    CLASSES = ('__background__',
               'aeroplane', 'bicycle', 'bird', 'boat',
               'bottle', 'bus', 'car', 'cat', 'chair',
               'cow', 'diningtable', 'dog', 'horse',
               'motorbike', 'person', 'pottedplant',
               'sheep', 'sofa', 'train', 'tvmonitor')
    
    NETS = {'vgg16': ('VGG16',
                      'VGG16_faster_rcnn_final.caffemodel'),
            'zf': ('ZF',
                      'ZF_faster_rcnn_final.caffemodel')}
    
    
    def vis_detections(im, class_name, dets, thresh=0.5):
        """Draw detected bounding boxes."""
        inds = np.where(dets[:, -1] >= thresh)[0]
        if len(inds) == 0:
            return
    
        im = im[:, :, (2, 1, 0)]
        fig, ax = plt.subplots(figsize=(12, 12))
        ax.imshow(im, aspect='equal')
        for i in inds:
            bbox = dets[i, :4]
            score = dets[i, -1]
    
            ax.add_patch(
                plt.Rectangle((bbox[0], bbox[1]),
                              bbox[2] - bbox[0],
                              bbox[3] - bbox[1], fill=False,
                              edgecolor='red', linewidth=3.5)
                )
            ax.text(bbox[0], bbox[1] - 2,
                    '{:s} {:.3f}'.format(class_name, score),
                    bbox=dict(facecolor='blue', alpha=0.5),
                    fontsize=14, color='white')
    
        ax.set_title(('{} detections with '
                      'p({} | box) >= {:.1f}').format(class_name, class_name,
                                                      thresh),
                      fontsize=14)
        plt.axis('off')
        plt.tight_layout()
        #plt.draw()
    def save_feature_picture(data, name, image_name=None, padsize = 1, padval = 1):
        data = data[0]
        #print "data.shape1: ", data.shape
        n = int(np.ceil(np.sqrt(data.shape[0])))
        padding = ((0, n ** 2 - data.shape[0]), (0, 0), (0, padsize)) + ((0, 0),) * (data.ndim - 3)
        #print "padding: ", padding
        data = np.pad(data, padding, mode='constant', constant_values=(padval, padval))
        #print "data.shape2: ", data.shape
        
        data = data.reshape((n, n) + data.shape[1:]).transpose((0, 2, 1, 3) + tuple(range(4, data.ndim + 1)))
        #print "data.shape3: ", data.shape, n
        data = data.reshape((n * data.shape[1], n * data.shape[3]) + data.shape[4:])
        #print "data.shape4: ", data.shape
        plt.figure()
        plt.imshow(data,cmap='gray')
        plt.axis('off')
        #plt.show()
        if image_name == None:
            img_path = './data/feature_picture/' 
        else:
            img_path = './data/feature_picture/' + image_name + "/"
            check_file(img_path)
        plt.savefig(img_path + name + ".jpg", dpi = 400, bbox_inches = "tight")
    def check_file(path):
        if not os.path.exists(path):
            os.mkdir(path)
    def demo(net, image_name):
        """Detect object classes in an image using pre-computed object proposals."""
    
        # Load the demo image
        im_file = os.path.join(cfg.DATA_DIR, 'demo', image_name)
        im = cv2.imread(im_file)
    
        # Detect all object classes and regress object bounds
        timer = Timer()
        timer.tic()
        scores, boxes = im_detect(net, im)
        for k, v in net.blobs.items():
            if k.find("conv")>-1 or k.find("pool")>-1 or k.find("rpn")>-1:
                save_feature_picture(v.data, k.replace("/", ""), image_name)#net.blobs["conv1_1"].data, "conv1_1") 
        timer.toc()
        print ('Detection took {:.3f}s for '
               '{:d} object proposals').format(timer.total_time, boxes.shape[0])
    
        # Visualize detections for each class
        CONF_THRESH = 0.8
        NMS_THRESH = 0.3
        for cls_ind, cls in enumerate(CLASSES[1:]):
            cls_ind += 1 # because we skipped background
            cls_boxes = boxes[:, 4*cls_ind:4*(cls_ind + 1)]
            cls_scores = scores[:, cls_ind]
            dets = np.hstack((cls_boxes,
                              cls_scores[:, np.newaxis])).astype(np.float32)
            keep = nms(dets, NMS_THRESH)
            dets = dets[keep, :]
            vis_detections(im, cls, dets, thresh=CONF_THRESH)
    
    def parse_args():
        """Parse input arguments."""
        parser = argparse.ArgumentParser(description='Faster R-CNN demo')
        parser.add_argument('--gpu', dest='gpu_id', help='GPU device id to use [0]',
                            default=0, type=int)
        parser.add_argument('--cpu', dest='cpu_mode',
                            help='Use CPU mode (overrides --gpu)',
                            action='store_true')
        parser.add_argument('--net', dest='demo_net', help='Network to use [vgg16]',
                            choices=NETS.keys(), default='vgg16')
    
        args = parser.parse_args()
    
        return args
    
    def print_param(net):
        for k, v in net.blobs.items():
    	print (k, v.data.shape)
        print ""
        for k, v in net.params.items():
    	print (k, v[0].data.shape)  
    
    if __name__ == '__main__':
        cfg.TEST.HAS_RPN = True  # Use RPN for proposals
    
        args = parse_args()
    
        prototxt = os.path.join(cfg.MODELS_DIR, NETS[args.demo_net][0],
                                'faster_rcnn_alt_opt', 'faster_rcnn_test.pt')
        #print "prototxt: ", prototxt
        caffemodel = os.path.join(cfg.DATA_DIR, 'faster_rcnn_models',
                                  NETS[args.demo_net][1])
    
        if not os.path.isfile(caffemodel):
            raise IOError(('{:s} not found.\nDid you run ./data/script/'
                           'fetch_faster_rcnn_models.sh?').format(caffemodel))
    
        if args.cpu_mode:
            caffe.set_mode_cpu()
        else:
            caffe.set_mode_gpu()
            caffe.set_device(args.gpu_id)
            cfg.GPU_ID = args.gpu_id
        net = caffe.Net(prototxt, caffemodel, caffe.TEST)
        
        #print_param(net)
    
        print '\n\nLoaded network {:s}'.format(caffemodel)
    
        # Warmup on a dummy image
        im = 128 * np.ones((300, 500, 3), dtype=np.uint8)
        for i in xrange(2):
            _, _= im_detect(net, im)
    
        im_names = ['000456.jpg', '000542.jpg', '001150.jpg',
                    '001763.jpg', '004545.jpg']
        for im_name in im_names:
            print '~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~'
            print 'Demo for data/demo/{}'.format(im_name)
            demo(net, im_name)
    
        #plt.show()
```
### 代码讲解
1.在data下手动创建“feature_picture”文件夹就可以替换原来的demo使用了。

2.上面代码主要添加方法是：save_feature_picture，它会对网络测试的某些阶段的数据处理然后保存。

3.某些阶段是因为：if k.find("conv")>-1 or k.find("pool")>-1 or k.find("rpn")>-1这行代码（110行），保证网络层name有这三个词的才会被保存，因为其他层无法用图片保存，如全连接（参数已经是二维的了）等层。

4.放开174行print_param(net)的注释，就可以看到网络参数的输出。

5.执行的最终结果 是在data/feature_picture产生以图片名字为文件夹名字的文件夹，文件夹下有以网络每层name为名字的图片。

6.另外部分网络的层name中有非法字符不能作为图片名字，我在代码的111行只是把‘字符/’剔除掉了，所以建议网络名字不要又其他字符。  

### 图片下载和代码下载

    git clone https://github.com/meihuakaile/faster-rcnn.git
