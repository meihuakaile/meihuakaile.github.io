---
title: caffe 数据库LMDB的读写
date: "2018/04/08"
tags: ['caffe', 'lmdb']
categories: ['cnn图片数据处理、显示']
copyright: true
top: 1
---
读写的图片都是灰度图，rgb图类似  

### 读数据库（图片的channel是2，其实是两张图片）

Datum是caffe里定义的一种存数据的结构。所以使用它时必须在开头import caffe。

它的属性有：
<ul>
<li>channels：图片的通道。如彩色图用3，灰度图用1.但是也许你想把它定义中其他数字，让它每个通道都是一个单张的图，这个例子就是2，每个通道是一张灰度图。</li>
<li>height：图片（即data）的高</li>
<li>width：图片（即data）的宽</li>
<li>data：图片的数据（像素值）</li>
<li>label：图片的label。如caffe的mnist里label是0~9的数字。</li>
</ul>

```python 
    import sys
    sys.path.insert(0,"../../python")#为了import caffe
    import numpy as np
    import lmdb
    import caffe
    import argparse
    from matplotlib import pyplot
    
    if __name__ == '__main__':
        parse = argparse.ArgumentParser()
        parse.add_argument('--lmdbpath')
        args = parse.parse_args()
     
        env = lmdb.open(args.lmdbpath, readonly=True)
        with env.begin() as txn:
            cursor = txn.cursor()
            for key, value in cursor:
                #print(key,len(value))#value是string类型
                print 'key: ',key
                   datum = caffe.proto.caffe_pb2.Datum()#datum类型
                   datum.ParseFromString(value)#转成datum
                   flat_x = np.fromstring(datum.data, dtype=np.uint8)#转成numpy类型
                   x = flat_x.reshape(datum.channels, datum.height, datum.width)#reshape大小
                   y = datum.label#图片的label
                fig = pyplot.figure()#把两张图片显示出来
                ax = fig.add_subplot(121)
                ax.imshow(x[0], cmap='gray')
                ax = fig.add_subplot(122)
                ax.imshow(x[1], cmap='gray')
                pyplot.show()
```
### 写数据库（例子中把两张图片作为一张图的两个channel）

caffe，以及faster rcnn写lmdb时都习惯把图片的名字写到txt文件中，通过txt去加载图片。思路大概如此。

我这个例子txt存的是：图片1名字   图片2名字    label

例如mnist的例子可以是：图片名字   label（代表这张图是0~9的哪个数字）
```python 
    import numpy as np
    import lmdb
    import Image as img
    from skimage import io
    import sys,os
    sys.path.insert(0, '../../python')
    import caffe
    import argparse
    import random
    
    def load_txt(txt, shuffle):
        if txt == None:
            print "txtpath!!!"
            exit(0)
        if not os.path.exists(txt):
             print "the txt is't exists"
             exit(0)
        flag = 0
        file_content = []
        txt_file = open(txt, 'r')
        for line in open(txt, 'r'):
            line = txt_file.readline()
            list = line.split()
            file_content.append([list[0], list[1], list[2]])
            flag += 1 
        if not shuffle == None: #为了打乱数据顺序
            random.shuffle(file_content)
        return file_content
    
    def add_argu(parse):
        parse.add_argument('--txt')
        parse.add_argument('--lmdb')
        parse.add_argument('--shuffle')#为了打乱数据顺序
        parse.add_argument('--picpath')
        return parse.parse_args() 
    
    if __name__ == '__main__':
        parse = argparse.ArgumentParser()
        args = add_argu(parse)
    
        content = []  
        content = load_txt(args.txt, args.shuffle)#加载图片名字和label
        print 'total: ', len(content)
        env = lmdb.Environment(args.lmdb, map_size=int(1e12))
        
        with env.begin(write=True) as txn:
            # txn is a Transaction object
            for i in range(len(content)):
                datum = caffe.proto.caffe_pb2.Datum()
                pic_path1 = args.picpath + '/' +  content[i][0]
                pic_path2 = args.picpath + '/' + content[i][1]
                label = int(content[i][2])
                img_file1 = io.imread(pic_path1)
                img_file2 = io.imread(pic_path2)
                datum.channels = 2#channels
                datum.height = img_file1.shape[0]#height
                datum.width = img_file1.shape[1]#width
                data = np.zeros((2,  img_file1.shape[0],  img_file2.shape[1]), dtype=np.uint8)#初始化data
                data[0] = img_file1
                data[1] = img_file2
                datum.data = data.tostring() #data
                datum.label = int(label)#label
                str_id = "%08d" %(i) + "_" + content[i][0] #'{:08}'.format(i) #顺序+图片名字作为key
                
                # The encode is only essential in Python 3
                txn.put(str_id.encode('ascii'), datum.SerializeToString())
```
注意：数据库的读是按照key的字典序读的，而不是按照写的顺序，所以写数据库时key必须重新写。
如果把图片名字作为key，读出来的图片仍是按照图片名字的字典序（不是写的顺序），因此之前的对图片名字打乱后再存入txt的操作就失去了意义。str_id是key  

