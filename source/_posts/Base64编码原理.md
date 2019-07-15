---
title: 'Base64编码原理'
date: "2018/05/24"
tags: [base64]
categories: ['java']
copyright: true
---
Base64的编码原理就是在有时我们的一些特殊字符无法进行传输，经常可以正确传输都是0~9,a~z,A~Z，+,/，它们正好可以用power（2,6）表示，即6位就可以了。
**_所以就是3 × 8 = 4 × 6的运算_**，我们原本的文本都是一个字节（8位）实现的，Base64就是把3个字节表示的内容转成用6位的0~9,a~z,A~Z，+,/表示。

总结自：http://blog.csdn.net/xiekuntarena/article/details/53521656
