---
title: 'java'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
copyright: true
---
### 内存泄露、内存溢出
泄露场景：
- 长生命周期对象持有短生命周期对象。eg，静态持有局部变量
- hash相关对象，计算hash值的数值变化，导致找不到该对象
- 数据库链接、文件打开等没有关闭。

溢出场景：
- 堆内存溢出（outOfMemoryError：java heap space）
在jvm规范中，堆中的内存是用来生成对象实例和数组的。
如果细分，堆内存还可以分为年轻代和年老代，年轻代包括一个eden区和两个survivor区。
eden --》survivor --》OLD

- 方法区内存溢出（outOfMemoryError：permgem space）
在jvm规范中，方法区主要存放的是类信息、常量、静态变量等。
所以如果程序加载的类过多，或者使用反射、gclib等这种动态代理生成类的技术，就可能导致该区发生内存溢出。

- 线程栈溢出（java.lang.StackOverflowError）
线程栈时线程独有的一块内存结构，所以线程栈发生问题必定是某个线程运行时产生的错误。
一般线程栈溢出是由于递归太深或方法调用层级过多导致的。

来自：https://blog.csdn.net/shimiso/article/details/21830871

### 三种gc方式
- 计数-清除。会产生很多零碎空间
- 拷贝。浪费空间
- 计数-移动。第二种的改进。

### 同步/异步、阻塞/非阻塞
- 同步。就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。
- 异步。则是相反，*调用*在发出之后，这个调用就直接返回了，没有返回结果。*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用。
- 阻塞调用。是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
- 非阻塞调用。指在不能立刻得到结果之前，该调用不会阻塞当前线程。

所谓同步异步，只是对于被调用者而言。
所谓阻塞非阻塞，仅仅对于调用者而言。

来自：https://www.zhihu.com/question/19732473

### 三次握手、四次挥手、http协议、四层
### 其他
- 进程 > 线程
- hashcode ——> equal 
https://blog.csdn.net/u012767369/article/details/79362752
### 单例模式
https://www.cnblogs.com/zhaoyan001/p/6365064.html
### 排序、查找
### shuffle过程
### join实现
### 矩阵、topn
https://blog.csdn.net/mylittlered/article/details/43272013

ArrayList是实现了基于动态数组的数据结构，LinkedList是基于链表结构，双向链表（为什么？）。
ArrayList的当前容量足够大，add()操作的效率非常高的。只有当ArrayList对容量的需求超出当前数组大小时，才需要进行扩容。扩容的过程中，会进行大量的数组复制操作。而数组复制时，最终将调用System.arraycopy()方法，因此add()操作的效率还是相当高的。

1.对ArrayList和LinkedList而言，在列表末尾增加一个元素所花的开销都是固定的。对 ArrayList而言，主要是在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配；而对LinkedList而言，这个开销是 统一的，分配一个内部Entry对象。
2.在ArrayList集合中随机位置添加或者删除一个元素时，当前的列表所所有的元素都会被移动。而LinkedList集合中添加或者删除一个元素的开销是固定的。
3.LinkedList集合不支持 高效的随机随机访问（RandomAccess），因为可能产生二次项的行为。
4.ArrayList的空间浪费主要体现在在list列表的结尾预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗相当的空间

进行 查操作时用ArrayList，进行增删操作的时候最好用LinkedList。
LinkedList.get(int location):通过一个计数索引值来实现的。例如，当我们调用get(int location)时，首先会比较“location”和“双向链表长度的1/2”；若前者大，则从链表头开始往后查找，直到location位置；否则，从链表末尾开始先前查找，直到location位置。
header是双向链表的表头，它是双向链表节点所对应的类Entry的实例。Entry中包含成员变量： previous, next, element。其中，previous是该节点的上一个节点，next是该节点的下一个节点，element是该节点所包含的值。
　　size是双向链表中节点的个数。遍历时使用forEach/迭代比较快，for比较慢。

参考：https://www.cnblogs.com/sierrajuan/p/3639353.html
###
流式接口？