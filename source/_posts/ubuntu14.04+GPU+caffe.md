---
title: ubuntu14.04+GPU+caffe
date: "2018/04/08" 
tags: ['cuda', 'gpu', 'caffe']
categories: ['caffe安装&&问题&&解决']
copyright: true
---
  
非常建议ubuntu14.04的系统  
# 验证硬件支持GPU CUDA
执行下面的操作，然后验证硬件支持GPU CUDA，只要型号存在于[nvidia-cuda-gpus](https://developer.nvidia.com/cuda-gpus)，就没问题了  

    
    
     $ lspci | grep -i nvidia

（GPU买之前是专门查过的，这个地方不太重要）  
# 确定系统是否支持  

    
    
    $ uname -m && cat /etc/*release

重点是“x86_64”这一项，保证是x86架构，64bit系统  
# 确定有gcc  

    
    
        $ gcc --version

（网上说，没有的话就先安装吧，这个是必须的用来编译CUDA Toolkit，不过Ubuntu 14.04是默认有的）  
# 下载cuda和nvidia驱动（建议都下载.run格式的）  
nvidia驱动： [ http://www.geforce.cn/drivers ](http://www.geforce.cn/drivers)  
cuda下载： [ https://developer.nvidia.com/cuda-downloads
](https://developer.nvidia.com/cuda-downloads)  
cuda由于比较大而且网内访问较慢，需要时间有点多  
# 安装前准备，在tty中显示中文  
因为待会的安装可能会有很多问题，报错是中文的话会显示乱码，影响找错。  
## 5.1安装fbterm  

    
    
    sudo apt-get install fbterm

## 5.2编辑.fbtermrc文件

    
    
    sudo vi .fbtermrc

，加入：  

    
    
    font-size=16
    text-codings=utf8

  
# 安装nvidia显卡和cuda
 （建议把他们两个放在同一个目录里，待会方面找）  
## 6.1进入tty（ctrl+alt+f1）后输入如下指令进入中文tty  

    
    
    sudo fbterm

## 6.2退出GUI，

    
    
    sudo stop lightdm

## 6.3禁用Ubuntu系统自带显卡驱动，  

    
    
        sudo vim/etc/modprobe.d/nvidia-graphics-drivers.conf

在文件输入：

    
    
       blacklist nouveau

保存退出。  

    
    
      sudo vim /etc/default/grub

在文件末尾添加：

    
    
      rdblacklist=nouveau nouveau.modeset=0

保存退出。  
## 6.4其他操作：  

    
    
    sudo mv /boot/initramfs-$(uname -r).img/boot/initramfs-$(uname -r)-nouveau.img
    sudo dracut/boot/initramfs-$(uname -r).img $(uname -r)
    sudo update-initramfs –u

  
第一条就提示没有这个文件，后面的就忘了，但是最最后的安装成功，这个地方就忽略了  
## 6.5 安装驱动
cd进入nvidia驱动的目录，安装驱动：  

    
    
     ls -l 查看目录里的内容
      sudo sh ./nvidia驱动名字

  
之后安装  
## 6.6 安装cuda
（不要用sudo，我的用了之后没法跑samples，即用了之后检测不到cuda）。另外其实这个cuda里还有一个nvidia，因为上面已
经安装过nvidia了，安装cuda时就不要选它了，把软连接（问是否要建一个/usr/local/cuda的目录）选上，把samples选上（后面检测是否安
装上cuda）：  

    
    
    sh ./cuda名字

  
# 后续  
## 7.1回到图形界面  

    
    
     sudo start lightdm

  
## 7.2修改系统环境变量  

    
    
    sudo vim ~/.bashrc

  
在最后加上：

    
    
    export PATH=/usr/local/cuda-6.5/bin:$PATH
    export LD_LIBRARY_PATH=/usr/local/cuda-6.5/lib64:$LD_LIBRARY_PATH
    sudo ldconfig（如果这句话的作用还是没有用，重启）

  
## 7.3检测cuda是否安装成功，编译cuda-samples并执行  

    
    
    进入你刚刚选择cuda sample安装的目录后make（如果提示没有make命令，请安装cmake。sudo apt-get install cmake)
    编译完毕，切换release目录cd ./bin/x86_64/linux/release
    运行实例 ./deviceQuery

  
  
# caffe安装:  
## 安装依赖
进入这个网址按照它的指令，安装依赖，里面要求的cuda上面已经安装：  
[ http://caffe.berkeleyvision.org/install_apt.htm
](http://caffe.berkeleyvision.org/install_apt.html) l  
## 下载caffe：  
[ https://github.com/BVLC/Caffe/ ](https://github.com/BVLC/Caffe/)  
## 安装python模块
（不影响编译，可以略过，但是你最终还是要用到，最好安装）  
官网：

[ http://caffe.berkeleyvision.org/installation.html
](http://caffe.berkeleyvision.org/installation.html)

因为官网提供方法安装不可行，可以参考我的： [caffe用python时可能需要的模块安装](/2018/04/08/caffe用python时可能需要的模块安装)  

## 编译  

官网： [ http://caffe.berkeleyvision.org/installation.htm
](http://caffe.berkeleyvision.org/installation.html) l

看到那个大大的Compilation以及CMake Build没有，因为我们是GPU，而且也没有安装cudnn，所以可以直接执行它的命令，不用改动配置文件（如果是cpu，就还要把CPU_ONLY :=1的注释放开）  
## 其他的 
[ http://caffe.berkeleyvision.org/ ](http://caffe.berkeleyvision.org/)
官网有很多例子等着你去看  

