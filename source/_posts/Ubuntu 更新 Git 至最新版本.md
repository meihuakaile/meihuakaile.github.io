---
title: 'Ubuntu更新Git至最新版本'
date: "2018/05/11"
tags: [git]
categories: [开发常用工具]
---
linux给git升级
直接`sudo apt-get install git`
上面的命令没有反应，这是因为 Ubuntu 自带的源中，Git 版本就是这么低，能怎么办。 
所以需要加入一个源，带有最新 Git 版本的源，步骤如下： 
1，添加源：
`sudo add-apt-repository ppa:git-core/ppa`
2，更新安装列表：
`sudo apt-get update`
3，升级 Git：
`sudo apt-get install git`
一气呵成，完成 Git 升级。

最后，可以执行`git --version`验证是否升级到最新版本。


转载自：
作者：诸葛_瓜皮 
来源：CSDN 
原文：https://blog.csdn.net/Ezreal_King/article/details/79999131 
