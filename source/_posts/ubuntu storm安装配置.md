---
title: ubuntu storm安装配置
tags: ['ubuntu', 'storm安装配置']
categories: ['']
copyright: true
---
一、安装准备：

JDK、ssh、python。安装都比较简单，我之前做别的时候已经安装，此处不再详述。

ssh的安装和服务启动可以参见我的另一个讲 [ hadoop安装的文章
](http://blog.csdn.net/u010668907/article/details/49159031)
。而且我的storm集群和hadoop相同，都是配置在三台虚拟机上的，用户组及用户名和hadoop那个一样，都要求用户名是hadoop

二、安装zookeeper集群（三台虚拟机）

前：创建三个文件夹：  

    
    
    #zookeeper安放路径
    mkdir -p /opt/modules
    #zookeeper日志存放路径
    mkdir -p /var/log/zookeeper
    #zookeeper数据存放路径
    mkdir -p /tmp/zookeeper

  

1.下载zookeeper源码

官网下载zookeeper

    
    
    mv zookeeper-3.4.8.tar.gz /opt/modules
    tar zxvf zookeeper-3.4.8.tar.gz
    ln -s zookeeper-3.4.8 zookeeper

  

2.配置zookeeper属性文件

进入zookeeper根目录，之后

    
    
    cd conf
    cp zoo_sample.cfg zoo.cfg

之后vim打开zoo.cfg，把下面内容追加到里面：

    
    
    tickTime=2000
    clientPort=2181
    initLimit=5
    syncLimit=2
    server.1=ip1:2888:3888
    server.2=ip2:2888:3888
    server.3=ip3:2888:3888

!!!!!!ip1,ip2,ip3用你的ip地址替换。

server.A=B:C:D，A是一个数字。B是ip地址，C首选端口，D是防止对方挂掉的预防端口。  

如果只是一台是虚拟机，那就要分配不同的端口了。

zoo.cfg里datadir的默认值是/tmp/zookeeper，我们要在这个目录下创建myid文件。三个虚拟机里这个文件只有一行，分别是上面对应的A值

    
    
    #创建myid文件
    vim /tmp/zookeeper/myid
    #添加对应数字
    1

  
3、配置日志打印文件

把输出日志放到指定文件夹下，便于日志回收和查看。下面在zookeeper根目录下进行：

打开bin/zkEnv.sh ，把下面代码添加到脚本主体（新手注意，不要放在有!/的第一行）的开头部分：  

    
    
    ZOO_LOG_DIR=/var/log/zookeeper

  
4.修改三个主要的文件夹的用户：  

    
    
    sudo chown -R hadoop:hadoop /opt/modules/zookeeper*
    sudo chown -R hadoop:hadoop /var/log/zookeeper
    sudo chown -R hadoop:hadoop /tmp/zookeeper

5.启动zookeeper集群

三台虚拟机进入zookeeper的根目录，执行下面命令：  

    
    
    bin/zkServer.sh start

再执行：

    
    
    bin/zkCli.sh -server 127.0.0.1:2181

如果安装成功会有[zk:127.0.0.1:2181(CONNETED) 1]的字样（忘记截图），可以输入help，ls /等命令。

！！！！！datadir下的日志和快照不会自动清理，需要通过其他方式定期清理。

  

三、storm安装：

1.安装storm依赖库

ZeroMQ

JZMQ

1.1安装zeromq：  

    
    
    wget http://download.zeromq.org/zeromq-2.1.7.tar.gz
    tar zxvf zeromq-2.1.7.tar.gz
    cd zeromq-2.1.7
    ./configure
    make
    sudo make install

  
安装出错：

（1）.安装过程中会出现cannot link with -luuid, install uuid-dev  c++错误，执行如下命令：  
  

    
    
    sudo apt-get install g++ build-essential gawk zlib1g-dev uuid-dev

（2）.configure: error: Unable to find a working C++ compiler  

    
    
    sudo apt-get install g++

  
1.2安装jzmq：

    
    
    sudo apt-get install libtool autoconf
    git clone https://github.com/nathanmarz/jzmq.git
    cd jzmq
    ./autogen.sh
    ./configure
    make
    sudo make install

安装出错：

（1）安装 JZMQ出错 (1).make[1]: *  
没有规则可以创建“org/zeromq/ZMQ.class”需要的目标“classdist_noinst.stamp”。 停止  

    
    
    #修正方法，创建classdist_noinst.stamp文件
    touch src/classdist_noinst.stamp

（2）make[1]: *** 没有规则可以创建“all”需要的目标“org/zeromq/ZMQ$Context.class”。 停止。  
make[1]:正在离开目录 `/home/hadoop/Storm/jzmq/src'  
make: *** [all-recursive] 错误 1  

    
    
    cd src/
    javac -d . org/zeromq/*.java
    #手动编译

  
2.安装storm集群：

下载storm： [ 点击打开链接 ](http://storm.incubator.apache.org/)

配置storm.yaml文件，以下都是在storm根目录下。

打开conf/storm.yaml文件

    
    
    #1.去掉下面代码的#并修改
    storm.zookeeper.servers:
            - "ip1"
            - "ip2"
            - "ip3"
    #如果zookeeper没有用默认端口，还要改的storm.zookeeper.port
    #2.storm.local.dir用于存少量nimbus，supervisor进程的少量状态
    storm.local.dir: "/var/storm"
    #3.nimbus.host,storm集群nimbus的机器地址，各个supervisor需要知道哪个是nimbus
    nimbus.host: "ip1"
    

上面只是少量的属性配置，其他的属性都在defaults.yaml文件里。

  

3.启动storm集群

    
    
    mkdir /var/storm
    sudo chown -R hadoop:hadoop /var/storm
    sudo chown -R hadoop:hadoop 你的storm地址

nimbus：在主控节点上运行，即ip1。在ip1虚拟机上，启动ip1并放在后台执行：

    
    
    bin/storm nimbus </dev/null 2<&1 &

supervise:在Storm工作节点上运行，在ip2,ip3上启动并放在后台执行：

    
    
    bin/storm supervisor </dev/null 2<&1 &

UI:必须在ip1上，因为它会去找nimbus连接

    
    
    bin/storm ui </dev/null 2<&1 &

之后在浏览器打开http://ip1:8080,会出现下面界面：

  

4.停止storm

集群storm停止需要一个一个的杀死进程。  

  

  

