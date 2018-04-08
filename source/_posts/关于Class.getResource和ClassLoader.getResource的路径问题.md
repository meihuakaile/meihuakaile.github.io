---
title: 关于Class.getResource和ClassLoader.getResource的路径问题
tags: ['Class.getResource', 'ClassLoader.getResource', 'ClassLoader.getResourceAsStrea']
categories: [java, spring]
copyright: true
---
ca [ http://www.cnblogs.com/yejg1212/p/3270152.html
](http://www.cnblogs.com/yejg1212/p/3270152.html) （有详细例子，建议看）

[ http://blog.csdn.net/netbug_nb/article/details/46121037
](http://blog.csdn.net/netbug_nb/article/details/46121037) （有详细例子）

总结：

1.Class.getResource（“”）括号中最前面加不加/的效果不同，总结是有/就会取根目录下找，没有就在当前路径下找。

加/ ：是取得class根目录下的路径，即编译以后target/classes的路径，还有maven项目java资源文件和resources目录在同一层时，
那层的路径。

不加/： 就是当前类的路径，编译以后的在target下的该class文件的路径

2.ClassLoader.getResource（“”）括号中最前面不能加/

不加/  的效果和1中加了/的效果一样

加了/ 输出是null。

3.ClassLoader.getResourceAsStream（）和2一样。

[ ](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=1736784
08)

