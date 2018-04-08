---
title: 用图形画出caffe输出数据的python程序&&git基础命令
tags: ['caffe', '图片']
categories: ['cnn图片数据处理、显示']
copyright: true
---
caffe的训练过程输出的数据用图形显示出来。先上效果图：

![](https://img-blog.csdn.net/20160505220839809?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)

图形说明：

    
    
    x是迭代次数，y左是train loss；y右是test accuracy。绿色是左边的线，红色是右边的线。

提醒：slover的格式必须是标准格式，如，冒号后边要有空格。。。。好吧，是我懒得做处理了，反正，就酱~

下载：

    
    
      git clone https://github.com/meihuakaile/caffe_tool.git

ubuntu中caffe的输出在/tmp/中，以caffe*命名。

我的代码基本就是把日志里的一些需要的文本匹配了出来。

代码用python编写，简单易懂易修改。

使用方法：

    
    
    python draw_loss.py --log log地址

git基础命令：

1.cd mkdir  
2.pwd显示当前目录  
3.git init初始化当前目录成git可以管理的目录  
4.git add 文件名  把文件提交到缓存区（.git文件夹中）  
5.git commit -m ‘提交描述’ 把缓存区内容推到文件中  
6.git status 显示当前仓库状态  
7.git diff 文件名 查看这个文件修改了什么  
8.git log 显示历史记录  
9.git log --pretty=oneline 以行的形式显示历史记录

10.git reset --hard HEAD^  返回上一个版本  
git reset --hard HEAD~n 返回第n个版本（是返回，而不是撤销哦，而且再看status也看不到n之后的了）  
11.git reflog 显示连带返回版本的log  
12.git reset --hard 版本号 返回这个版本  
13.cat 文件名  显示文件的内容  
14.git checkout -- 文件名  可以在提交之前就撤销文件的修改  
15.rm 文件名  删除该文件  
16.git remote add origin 仓库（https://github.com/tugenhua0707/testgit.git）
把本地仓库和远程仓库连起来  
git push -u origin master  
由于远程库是空的，我们第一次推送master分支时，加上了 –u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的m
aster分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。  
git push origin master 之后可以直接推送  
17\. git clone -q https://github.com/meihuakaile/hello_world 从远程克隆仓库到本地

git学习遇到的问题：  

1.fatal: I don't handle protocol 'https'  
解决： git clone -q https://github.com/meihuakaile/hello_world（加上
-q，网上说是windows的git版本问题）  

