---
title: 'npm'
date: "2018/05/11"
tags: [npm]
categories: [开发常用工具]
copyright: true
---
# node和npm下载安装
下载： [https://nodejs.org/en/](https://nodejs.org/en/)
配置： 
`sudo ln /home/chenliclchen/install/node-v8.9.4-linux-x64/bin/node /usr/local/bin/node`
`sudo ln /home/chenliclchen/install/node-v8.9.4-linux-x64/bin/npm /usr/local/bin/npm`
加环境变量：
在.bashrc 中加入`/usr/local/bin`路径，并source .bashrc
# 把 /home/chenliclchen/install/node-v8.9.4-linux-x64/ 换成你的解压目录
# 使用nrm来切换npm源。
安装nrm：`npm install -g nrm`
列出可用的源，加*是正在使用的源：`nrm ls`
添加私有源：`nrm add`
通过nrm use指令来切换不同的源：`nrm use taobao `

# 常用命令
`npm install npm -g`          升级npm
`npm install <Module Name>`   安装模块，安装后放在工程目录下的 node_modules 目录中，在代码中只需要通过 require('Module Name') 的方式，无需指定第三方包路径。
`npm uninstall <Module Name>` 卸载模块
`npm update <Module Name>`    更新模块

# npm本地、全局安装区别 
npm 的包安装分为本地安装（local）、全局安装（global）两种，从敲的命令行来看，差别只是有没有-g。
本地安装：
（1） 将安装包放在 ./node_modules 下（运行 npm 命令时所在的目录），如果没有 node_modules 目录，会在当前执行 npm 命令的目录下生成 node_modules 目录。
（2）可以通过 require() 来引入本地安装的包。
全局安装：
（1）将安装包放在 /usr/local 下或者你 node 的安装目录。
（2）可以直接在命令行里使用。
`npm list -g` 查看全局安装的模块；
`npm list <Module Name>`查看某模块的信息