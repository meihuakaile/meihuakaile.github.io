---
title: "faster rcnn源码解读（五）之layer（网络里的input-data）"
date: "2018/04/08"
tags: ['深度学习', 'faster rcnn源码理解']
categories: ['faster rcnn', 'faster cnn源码理解']
copyright: true
---
faster rcnn用python版本的  [ https://github.com/rbgirshick/py-faster-rcnn](https://github.com/rbgirshick/py-faster-rcnn)

layer源码地址： [ https://github.com/rbgirshick/py-faster-rcnn/blob/master/lib/roi_data_layer/layer.py ](https://github.com/rbgirshick/py-faster-rcnn/blob/master/lib/roi_data_layer/layer.py)

### 源码
```python
    # --------------------------------------------------------
    # Fast R-CNN
    # Copyright (c) 2015 Microsoft
    # Licensed under The MIT License [see LICENSE for details]
    # Written by Ross Girshick
    # --------------------------------------------------------
    
    """The data layer used during training to train a Fast R-CNN network.
    
    RoIDataLayer implements a Caffe Python layer.
    """
    
    import caffe
    from fast_rcnn.config import cfg
    from roi_data_layer.minibatch import get_minibatch
    import numpy as np
    import yaml
    from multiprocessing import Process, Queue
    
    class RoIDataLayer(caffe.Layer):
        """Fast R-CNN data layer used for training."""
    
        def _shuffle_roidb_inds(self):
            """Randomly permute the training roidb."""
            if cfg.TRAIN.ASPECT_GROUPING:
                widths = np.array([r['width'] for r in self._roidb])
                heights = np.array([r['height'] for r in self._roidb])
                horz = (widths >= heights)
                vert = np.logical_not(horz)
                horz_inds = np.where(horz)[0]
                vert_inds = np.where(vert)[0]
                inds = np.hstack((
                    np.random.permutation(horz_inds),
                    np.random.permutation(vert_inds)))
                inds = np.reshape(inds, (-1, 2))
                row_perm = np.random.permutation(np.arange(inds.shape[0]))
                inds = np.reshape(inds[row_perm, :], (-1,))
                self._perm = inds#把roidb的索引打乱，造成的shuffle，打乱的索引存储的地方
            else:
                self._perm = np.random.permutation(np.arange(len(self._roidb)))
            self._cur = 0
    
        def _get_next_minibatch_inds(self):
            """Return the roidb indices for the next minibatch."""
            if self._cur + cfg.TRAIN.IMS_PER_BATCH >= len(self._roidb):
                self._shuffle_roidb_inds()
    
            db_inds = self._perm[self._cur:self._cur + cfg.TRAIN.IMS_PER_BATCH]
            self._cur += cfg.TRAIN.IMS_PER_BATCH#相当于一个指向_perm的指针，每次取走图片后，他会跟着变化#cfg.TRAIN.IMS_PER_BATCH： （猜测，每次取图片的数量）
            return db_inds#本次取得图片的索引
    
        def _get_next_minibatch(self):#取得本次图片的索引，即db_inds
            """Return the blobs to be used for the next minibatch.
    
            If cfg.TRAIN.USE_PREFETCH is True, then blobs will be computed in a
            separate process and made available through self._blob_queue.
            """
            if cfg.TRAIN.USE_PREFETCH:
                return self._blob_queue.get()
            else:
                db_inds = self._get_next_minibatch_inds()
                minibatch_db = [self._roidb[i] for i in db_inds]#本次的roidb
                return get_minibatch(minibatch_db, self._num_classes)
    
        def set_roidb(self, roidb):
            """Set the roidb to be used by this layer during training."""
            self._roidb = roidb
            self._shuffle_roidb_inds()
            if cfg.TRAIN.USE_PREFETCH:
                self._blob_queue = Queue(10)
                self._prefetch_process = BlobFetcher(self._blob_queue,
                                                     self._roidb,
                                                     self._num_classes)
                self._prefetch_process.start()
                # Terminate the child process when the parent exists
                def cleanup():
                    print 'Terminating BlobFetcher'
                    self._prefetch_process.terminate()
                    self._prefetch_process.join()
                import atexit
                atexit.register(cleanup)
    
        def setup(self, bottom, top):
            """Setup the RoIDataLayer."""
    
            # parse the layer parameter string, which must be valid YAML
            layer_params = yaml.load(self.param_str_)
    
            self._num_classes = layer_params['num_classes']#网络里的类别数值21
    
            self._name_to_top_map = {}#{'gt_boxes': 2, 'data': 0, 'im_info': 1}字典的value值是top的对应索引
    
            # data blob: holds a batch of N images, each with 3 channels
            idx = 0
            top[idx].reshape(cfg.TRAIN.IMS_PER_BATCH, 3,
                max(cfg.TRAIN.SCALES), cfg.TRAIN.MAX_SIZE)
            self._name_to_top_map['data'] = idx
            idx += 1
    
            if cfg.TRAIN.HAS_RPN:
                top[idx].reshape(1, 3)
                self._name_to_top_map['im_info'] = idx
                idx += 1
    
                top[idx].reshape(1, 4)
                self._name_to_top_map['gt_boxes'] = idx
                idx += 1
            else: # not using RPN
                # rois blob: holds R regions of interest, each is a 5-tuple
                # (n, x1, y1, x2, y2) specifying an image batch index n and a
                # rectangle (x1, y1, x2, y2)
                top[idx].reshape(1, 5)
                self._name_to_top_map['rois'] = idx
                idx += 1
    
                # labels blob: R categorical labels in [0, ..., K] for K foreground
                # classes plus background
                top[idx].reshape(1)
                self._name_to_top_map['labels'] = idx
                idx += 1
    
                if cfg.TRAIN.BBOX_REG:
                    # bbox_targets blob: R bounding-box regression targets with 4
                    # targets per class
                    top[idx].reshape(1, self._num_classes * 4)
                    self._name_to_top_map['bbox_targets'] = idx
                    idx += 1
    
                    # bbox_inside_weights blob: At most 4 targets per roi are active;
                    # thisbinary vector sepcifies the subset of active targets
                    top[idx].reshape(1, self._num_classes * 4)
                    self._name_to_top_map['bbox_inside_weights'] = idx
                    idx += 1
    
                    top[idx].reshape(1, self._num_classes * 4)
                    self._name_to_top_map['bbox_outside_weights'] = idx
                    idx += 1
    
            print 'RoiDataLayer: name_to_top:', self._name_to_top_map
            assert len(top) == len(self._name_to_top_map)
    
        def forward(self, bottom, top):
            """Get blobs and copy them into this layer's top blob vector."""
            blobs = self._get_next_minibatch()
    
            for blob_name, blob in blobs.iteritems():
                top_ind = self._name_to_top_map[blob_name]
                # Reshape net's input blobs
                top[top_ind].reshape(*(blob.shape))
                # Copy data into net's input blobs
                top[top_ind].data[...] = blob.astype(np.float32, copy=False)
    
        def backward(self, top, propagate_down, bottom):
            """This layer does not propagate gradients."""
            pass
    
        def reshape(self, bottom, top):
            """Reshaping happens during the call to forward."""
            pass
    
    class BlobFetcher(Process):
        """Experimental class for prefetching blobs in a separate process."""
        def __init__(self, queue, roidb, num_classes):
            super(BlobFetcher, self).__init__()
            self._queue = queue
            self._roidb = roidb
            self._num_classes = num_classes
            self._perm = None
            self._cur = 0
            self._shuffle_roidb_inds()
            # fix the random seed for reproducibility
            np.random.seed(cfg.RNG_SEED)
    
        def _shuffle_roidb_inds(self):
            """Randomly permute the training roidb."""
            # TODO(rbg): remove duplicated code
            self._perm = np.random.permutation(np.arange(len(self._roidb)))
            self._cur = 0
    
        def _get_next_minibatch_inds(self):
            """Return the roidb indices for the next minibatch."""
            # TODO(rbg): remove duplicated code
            if self._cur + cfg.TRAIN.IMS_PER_BATCH >= len(self._roidb):
                self._shuffle_roidb_inds()
    
            db_inds = self._perm[self._cur:self._cur + cfg.TRAIN.IMS_PER_BATCH]
            self._cur += cfg.TRAIN.IMS_PER_BATCH
            return db_inds
    
        def run(self):
            print 'BlobFetcher started'
            while True:
                db_inds = self._get_next_minibatch_inds()
                minibatch_db = [self._roidb[i] for i in db_inds]
                blobs = get_minibatch(minibatch_db, self._num_classes)
                self._queue.put(blobs)
```
### 代码讲解
下面的roidb都只是一次batch的  

3.1 setup  在caffe.SGDSolver时调用；setup的top（list猜测是c++的vector）的每个项是caffe._caffe.Blob
（猜测，输出的Top shape就是上面的top,在setup中被shape；top[0],1 3 [600] 1000;top[1],1 3;top[2], 1 4)（疑问，在forward中blob的数据shape被重置，有时大小甚至会不定）

3.2 name_to_top: {'gt_boxes': 2, 'data': 0, 'im_info': 1}  字典的value值是top的对应索引

3.3 solver.step(1)  会调用layer的reshape、forward

3.4 self._perm： 把roidb的索引打乱，造成图片的shuffle，打乱的索引存储的地方

3.5 cfg.TRAIN.IMS_PER_BATCH：（猜测，每次取图片的数量）

3.6 self._cur： 相当于一个指向_perm的指针，每次取走图片后，他会跟着变化

3.7 db_inds： 本次取得图片的索引

3.8 def _get_next_minibatch_inds(self)： 取得本次图片的索引，即db_inds

3.9 minibatch_db： 本次的 roidb

3.10 _num_classes： 网络里的类别数值  21

3.11 forward（）： 得到blob并处理放进top

solver.step(1)->reshape->forward -> _get_next_minbatch->_get_next_minbatch_inds-> (前面在layers里,现在进入minibatch组建真正的blob)get_minibatch
  

