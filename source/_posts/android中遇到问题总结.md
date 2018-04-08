---
title: android中遇到问题总结
tags: ['android', 'eclipse', '调试']
categories: ['java', 'android', '问题解决&&安装']
copyright: true
---
1.Failure [INSTALL_FAILED_OLDER_SDK]  
错误：开始在eclipse中其他项目可以真机调试，可是只有其中一个不行，后来选择把apk考培的真机上运行报错“Failure
[INSTALL_FAILED_OLDER_SDK]”。  

解决：把AndroidMainfest文件的android:minSdkVersion="16"这句话注释掉就行了。

  

2\. Type 'JNICALL' could not be resolved  
解决：Project Properties -> C/C++ General -> Path and Symbols  
选择include标签，Add -> $Android_NDK_HOME/platforms/android-14/arch-arm/usr/include  
选中All languages.  
最后Apply -> OK  
（  $Android_NDK_HOME  是你下载的  Android_NDK地址  ）  
3.eclipse菜单栏新建android的模拟器图标不见了  
解决：不是不见了，右上角的快速切换到java环境就好了。  
  
4、logcat不显示数据  
解决：打开DDMS，点击右上角你的设备即可。  
  
5、opendir failed, Permission denied  solution  

问题：真机调试时在adb访问data，报错：opendir failed, Permission denied  solution

![](https://img-blog.csdn.net/20151103112750957?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

解决：

![](https://img-blog.csdn.net/20151103112943924?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

