---
title: 'map的key为自定义对象'
date: "2018/05/24"
tags: [map的key]
categories: ['java']
copyright: true
---
1.**_并不是所有对象都是可以做map的key的_**。
如，我们平常自己实现的类，如果不重写hashCode和equals方法，是没有办法做map的key。
在比较的过程中先比较hashCode再比较equals方法。
如果hashCode一样，equals也是true，数据会被冲掉（丢失）。
hashCode一般使用自己实现类的不可变属性计算而得，equals则由自己的实现定义计算而得。

详细举例：
譬如把一个自己定义的class Foo{...}作为key放到HashMap。实际上HashMap也是把数据存在一个数组里面，所以在put函数
里面，HashMap会调Foo.hashCode()算出作为这个元素在数组里面的下标，然后把key和value封装成一个对象放到数组。
等一下，万一2个对象算出来的hash code一样怎么办？会不会冲掉？先回答第2个问题，会不会冲掉就要看Foo.equals()了，
如果equals()也是true那就要冲掉了。万一是false,就是所谓的collision了。当2个元素hashCode一样但是equals为false的时候，那个HashMap里面的数组的这个元素就变成了链表。也就是hash code一样的元素在一个链表里面，链表的头在那个数组里面。get的时候，HashMap也先调key.hashCode()算出数组下标，然后看equals如果是true就是找到了，所以就涉及了equals。

2.hashMap可以做key。
map是一个接口，AbstractMap是一个实现了map接口的抽象类，这个抽象类实现了其他map通用的方法，如equals、hashCode、contains等。
而其他map类继承了AbstractMap，只需要实现部分方法。因此继承了AbstractMap的类都可以作为key。

