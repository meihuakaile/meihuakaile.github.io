---
title: faster rcnn源码解读（四）之数据类型imdb.py和pascal_voc.py（主要是imdb和roidb数据类型的解说）
date: "2018/04/08"
tags: ['深度学习', 'faster rcnn源码理解']
categories: ['faster rcnn', 'faster cnn源码理解']
copyright: true
---
faster用python版本的  [ https://github.com/rbgirshick/py-faster-rcnn](https://github.com/rbgirshick/py-faster-rcnn)

imdb.py源码地址： [ https://github.com/rbgirshick/py-faster-rcnn/blob/master/lib/datasets/imdb.py ](https://github.com/rbgirshick/py-faster-rcnn/blob/master/lib/datasets/imdb.py)

### imdb源码
```python
    # --------------------------------------------------------
    # Fast R-CNN
    # Copyright (c) 2015 Microsoft
    # Licensed under The MIT License [see LICENSE for details]
    # Written by Ross Girshick
    # --------------------------------------------------------
    
    import os
    import os.path as osp
    import PIL
    from utils.cython_bbox import bbox_overlaps
    import numpy as np
    import scipy.sparse
    from fast_rcnn.config import cfg
    
    class imdb(object):
        """Image database."""
    
        def __init__(self, name):
            self._name = name
            self._num_classes = 0#类别的长度
            self._classes = []#类别定义
            self._image_index = []#a list of image name（read from eg：root/data + /VOCdevkit2007/VOC2007/ImageSets/Main/{image_set}.txt）
            self._obj_proposer = 'selective_search'
            self._roidb = None#gt_roidb（cfg.TRAIN.PROPOSAL_METHOD=gt导致了此操作）
            self._roidb_handler = self.default_roidb
            # Use this dict for storing dataset specific config options
            self.config = {}
    
        @property
        def name(self):
            return self._name
    
        @property
        def num_classes(self):
            return len(self._classes)
    
        @property
        def classes(self):
            return self._classes
    
        @property
        def image_index(self):
            return self._image_index
    
        @property
        def roidb_handler(self):
            return self._roidb_handler
    
        @roidb_handler.setter
        def roidb_handler(self, val):
            self._roidb_handler = val
    
        def set_proposal_method(self, method):
            method = eval('self.' + method + '_roidb')
            self.roidb_handler = method
    
        @property
        def roidb(self):
            # A roidb is a list of dictionaries, each with the following keys:
            #   boxes
            #   gt_overlaps
            #   gt_classes
            #   flipped
            if self._roidb is not None:
                return self._roidb
            self._roidb = self.roidb_handler()
            return self._roidb
    
        @property
        def cache_path(self):
            cache_path = osp.abspath(osp.join(cfg.DATA_DIR, 'cache'))
            if not os.path.exists(cache_path):
                os.makedirs(cache_path)
            return cache_path
    
        @property
        def num_images(self):
          return len(self.image_index)
    
        def image_path_at(self, i):
            raise NotImplementedError
    
        def default_roidb(self):
            raise NotImplementedError
    
        def evaluate_detections(self, all_boxes, output_dir=None):
            """
            all_boxes is a list of length number-of-classes.
            Each list element is a list of length number-of-images.
            Each of those list elements is either an empty list []
            or a numpy array of detection.
    
            all_boxes[class][image] = [] or np.array of shape #dets x 5
            """
            raise NotImplementedError
    
        def _get_widths(self):
          return [PIL.Image.open(self.image_path_at(i)).size[0]
                  for i in xrange(self.num_images)]
    
        def append_flipped_images(self):
            num_images = self.num_images
            widths = self._get_widths()
            for i in xrange(num_images):
                boxes = self.roidb[i]['boxes'].copy()
                oldx1 = boxes[:, 0].copy()
                oldx2 = boxes[:, 2].copy()
                boxes[:, 0] = widths[i] - oldx2 - 1
                boxes[:, 2] = widths[i] - oldx1 - 1
                assert (boxes[:, 2] >= boxes[:, 0]).all()
                entry = {'boxes' : boxes,
                         'gt_overlaps' : self.roidb[i]['gt_overlaps'],
                         'gt_classes' : self.roidb[i]['gt_classes'],
                         'flipped' : True}
                self.roidb.append(entry)
            self._image_index = self._image_index * 2
    
        def evaluate_recall(self, candidate_boxes=None, thresholds=None,
                            area='all', limit=None):
            """Evaluate detection proposal recall metrics.
    
            Returns:
                results: dictionary of results with keys
                    'ar': average recall
                    'recalls': vector recalls at each IoU overlap threshold
                    'thresholds': vector of IoU overlap thresholds
                    'gt_overlaps': vector of all ground-truth overlaps
            """
            # Record max overlap value for each gt box
            # Return vector of overlap values
            areas = { 'all': 0, 'small': 1, 'medium': 2, 'large': 3,
                      '96-128': 4, '128-256': 5, '256-512': 6, '512-inf': 7}
            area_ranges = [ [0**2, 1e5**2],    # all
                            [0**2, 32**2],     # small
                            [32**2, 96**2],    # medium
                            [96**2, 1e5**2],   # large
                            [96**2, 128**2],   # 96-128
                            [128**2, 256**2],  # 128-256
                            [256**2, 512**2],  # 256-512
                            [512**2, 1e5**2],  # 512-inf
                          ]
            assert areas.has_key(area), 'unknown area range: {}'.format(area)
            area_range = area_ranges[areas[area]]
            gt_overlaps = np.zeros(0)
            num_pos = 0
            for i in xrange(self.num_images):
                # Checking for max_overlaps == 1 avoids including crowd annotations
                # (...pretty hacking :/)
                max_gt_overlaps = self.roidb[i]['gt_overlaps'].toarray().max(axis=1)
                gt_inds = np.where((self.roidb[i]['gt_classes'] > 0) &
                                   (max_gt_overlaps == 1))[0]
                gt_boxes = self.roidb[i]['boxes'][gt_inds, :]
                gt_areas = self.roidb[i]['seg_areas'][gt_inds]
                valid_gt_inds = np.where((gt_areas >= area_range[0]) &
                                         (gt_areas <= area_range[1]))[0]
                gt_boxes = gt_boxes[valid_gt_inds, :]
                num_pos += len(valid_gt_inds)
    
                if candidate_boxes is None:
                    # If candidate_boxes is not supplied, the default is to use the
                    # non-ground-truth boxes from this roidb
                    non_gt_inds = np.where(self.roidb[i]['gt_classes'] == 0)[0]
                    boxes = self.roidb[i]['boxes'][non_gt_inds, :]
                else:
                    boxes = candidate_boxes[i]
                if boxes.shape[0] == 0:
                    continue
                if limit is not None and boxes.shape[0] > limit:
                    boxes = boxes[:limit, :]
    
                overlaps = bbox_overlaps(boxes.astype(np.float),
                                         gt_boxes.astype(np.float))
    
                _gt_overlaps = np.zeros((gt_boxes.shape[0]))
                for j in xrange(gt_boxes.shape[0]):
                    # find which proposal box maximally covers each gt box
                    argmax_overlaps = overlaps.argmax(axis=0)
                    # and get the iou amount of coverage for each gt box
                    max_overlaps = overlaps.max(axis=0)
                    # find which gt box is 'best' covered (i.e. 'best' = most iou)
                    gt_ind = max_overlaps.argmax()
                    gt_ovr = max_overlaps.max()
                    assert(gt_ovr >= 0)
                    # find the proposal box that covers the best covered gt box
                    box_ind = argmax_overlaps[gt_ind]
                    # record the iou coverage of this gt box
                    _gt_overlaps[j] = overlaps[box_ind, gt_ind]
                    assert(_gt_overlaps[j] == gt_ovr)
                    # mark the proposal box and the gt box as used
                    overlaps[box_ind, :] = -1
                    overlaps[:, gt_ind] = -1
                # append recorded iou coverage level
                gt_overlaps = np.hstack((gt_overlaps, _gt_overlaps))
    
            gt_overlaps = np.sort(gt_overlaps)
            if thresholds is None:
                step = 0.05
                thresholds = np.arange(0.5, 0.95 + 1e-5, step)
            recalls = np.zeros_like(thresholds)
            # compute recall for each iou threshold
            for i, t in enumerate(thresholds):
                recalls[i] = (gt_overlaps >= t).sum() / float(num_pos)
            # ar = 2 * np.trapz(recalls, thresholds)
            ar = recalls.mean()
            return {'ar': ar, 'recalls': recalls, 'thresholds': thresholds,
                    'gt_overlaps': gt_overlaps}
    
        def create_roidb_from_box_list(self, box_list, gt_roidb):
            assert len(box_list) == self.num_images, \
                    'Number of boxes must match number of ground-truth images'
            roidb = []
            for i in xrange(self.num_images):
                boxes = box_list[i]
                num_boxes = boxes.shape[0]
                overlaps = np.zeros((num_boxes, self.num_classes), dtype=np.float32)
    
                if gt_roidb is not None and gt_roidb[i]['boxes'].size > 0:
                    gt_boxes = gt_roidb[i]['boxes']
                    gt_classes = gt_roidb[i]['gt_classes']
                    gt_overlaps = bbox_overlaps(boxes.astype(np.float),
                                                gt_boxes.astype(np.float))
                    argmaxes = gt_overlaps.argmax(axis=1)
                    maxes = gt_overlaps.max(axis=1)
                    I = np.where(maxes > 0)[0]
                    overlaps[I, gt_classes[argmaxes[I]]] = maxes[I]
    
                overlaps = scipy.sparse.csr_matrix(overlaps)
                roidb.append({
                    'boxes' : boxes,
                    'gt_classes' : np.zeros((num_boxes,), dtype=np.int32),
                    'gt_overlaps' : overlaps,
                    'flipped' : False,
                    'seg_areas' : np.zeros((num_boxes,), dtype=np.float32),
                })
            return roidb
    
        @staticmethod
        def merge_roidbs(a, b):
            assert len(a) == len(b)
            for i in xrange(len(a)):
                a[i]['boxes'] = np.vstack((a[i]['boxes'], b[i]['boxes']))
                a[i]['gt_classes'] = np.hstack((a[i]['gt_classes'],
                                                b[i]['gt_classes']))
                a[i]['gt_overlaps'] = scipy.sparse.vstack([a[i]['gt_overlaps'],
                                                           b[i]['gt_overlaps']])
                a[i]['seg_areas'] = np.hstack((a[i]['seg_areas'],
                                               b[i]['seg_areas']))
            return a
    
        def competition_mode(self, on):
            """Turn competition mode on or off."""
            pass
```
### 代码讲解
get_imdb->factory->pascal_voc->(继承)imdb

#### factory

year = ['2007', '2012']
split = ['train', 'val', 'trainval', 'test']

#### imdb

image_set: split
devkit_path: config.DATA_DIR(root/data/) + VOCdevkit + year
data_path: devkit_path + '/' + 'VOC' + year
image_index: a list read image name from 如，root/data + /VOCdevkit2007/VOC2007/ImageSets/Main/{image_set}.txt

roidb: gt_roidb得到（cfg.TRAIN.PROPOSAL_METHOD=gt导致了此操作）
classes： 类别定义
num_classes： 类别的长度
class_to_ind：  {类别名：类别索引}字典

num_images(): image_index'length，数据库中图片个数
image_path_at（index）： 得到第index图片的地址， data_path \+ '/' +'JPEGImages' + image_index[index] + image_ext(.jpg)

在train_faster_rcnn_alt_opt.py的imdb.set_proposal_method之后一旦用imdb.roidb都会用gt_roidb读取xml中的内容中得到部分信息
xml的地址： data_path + '/' + 'Annotations' + '/' + index + '.xml'
(root/data/) + VOCdevkit + year  + '/' + 'VOC' + year + '/' + 'Annotations' +'/' + index + '.xml'

#### roidb
get_training_roidb： 对得到的roi做是否反转（参见roidb的flipped，为了扩充数据库）和到roidb.py的prepare_roidb中计算得到roidb的其他数据
一张图有一个roidb  ，每个roidb是一个字典
boxes: four rows. the proposal. left-up, right-down
gt_overlaps: len（box）× 类别数（即，每个box对应的类别。初始化时，从xml读出来的类别对应类别值是1.0，被压缩保存）
gt_classes: 每个box的类别索引
flipped: true,代表图片被水平反转，改变了boxes里第一、三列的值（所有原图都这样的操作，imdb.image_index × 2）(cfg.TRAIN.USE_FLIPPED会导致此操作的发生，见train.py 116行)
seg_areas： box的面积

#### prepare_roidb中得到
（下面的值在roidb.py的prepare_roidb中得到）
image： image_path_at（index），此roi的图片地址
width：此图片的宽
height：高
max_classes: box的类别=labels（gt_overlaps行最大值索引）
max_overlaps: （gt_overlaps行最大值）（max_overlaps=0，max_classes=0，即都是背景，否则不正确）
output_dir： ROOT_DIR + 'output' + EXP_DIR('faster_rcnn_alt_opt') + imdb.name("voc_2007_trainval" or "voc_2007_test")

