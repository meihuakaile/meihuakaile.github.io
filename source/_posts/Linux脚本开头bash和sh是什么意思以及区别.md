---
title: 'Linux脚本开头#!/bin/bash和#!/bin/sh是什么意思以及区别'
date: "2018/08/14"
tags: [linux]
categories: [linux]
---
转载自：https://www.cnblogs.com/EasonJim/p/6850319.html
### 意思
`#!/bin/sh`是指此脚本使用`/bin/sh`来解释执行，`#!`是特殊的表示符，其后面根的是此解释此脚本的shell的路径。
其实第一句的`#!`是对脚本的解释器程序路径，脚本的内容是由解释器解释的，我们可以用各种各样的解释器来写对应的脚本。
比如说`/bin/csh`脚本，`/bin/perl`脚本，`/bin/awk`脚本，`/bin/sed`脚本，甚至`/bin/echo`等等。
`#!/bin/bash`同理。

### 区别
![](1.png)
GNU/Linux操作系统中的/bin/sh本是bash (Bourne-Again Shell) 的符号链接，但鉴于bash过于复杂，有人把bash从NetBSD移植到Linux并更名为dash (Debian Almquist Shell)，并建议将/bin/sh指向它，以获得更快的脚本执行速度。Dash Shell 比Bash Shell小的多，符合POSIX标准。
Ubuntu继承了Debian，所以从Ubuntu 6.10开始默认是Dash Shell。
应该说，/bin/sh与/bin/bash虽然大体上没什么区别，但仍存在不同的标准。标记为#!/bin/sh的脚本不应使用任何POSIX没有规定的特性 (如let等命令, 但#!/bin/bash可以)。Debian曾经采用/bin/bash更改/bin/dash，目的使用更少的磁盘空间、提供较少的功能、获取更快的速度。但是后来经过shell脚本测试存在运行问题。因为原先在bash shell下可以运行的shell script (shell 脚本)，在/bin/sh下还是会出现一些意想不到的问题，不是100%的兼用。
上面可以这样理解，使用man sh命令和man bash命令去观察，可以发现sh本身就是dash，也就更好的说明集成Debian系统之后的更改。

