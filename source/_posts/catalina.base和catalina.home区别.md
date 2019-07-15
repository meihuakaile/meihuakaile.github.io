---
title: 'catalina.base和catalina.home区别'
date: "2017/05/25"
tags: ["catalina.base","catalina.home"]
categories: [java]
copyright: true
---
参考：http://desert3.iteye.com/blog/1356006

tomcat的目录：
`bin` (运行脚本） 
`conf` (配置文件） 
`lib` (核心库文件） 
`logs` (日志目录) 
`temp` (临时目录) 
`webapps` (自动装载的应用程序的目录） 
`work` (JVM临时文件目录[java.io.tmpdir]) 

上述目录中lib和bin是tomcat独有的，其他的目录有多少个实例就必须有多少个这样的文件夹。因此引出了题目catalina.base和catalina.home的区别。
当你想要使用多个tomcat实例（运行多个项目），又不想多次安装tomcat时。会把除了lib和bin目录外的其他目录复制多份，每个项目一份。
**_而这时原来的有bin和lib的目录就是catalina.home；每个项目独有的其他几个目录所在层就是catalina.base。_**

**_总的说，CATALINA_HOME 是Tomcat 的安装目录，CATALINA_BASE 是Tomcat 的工作目录。_**
不知道时，可以在代码里通过String property = System.getProperty("catalina.base"); 得知它们的默认值（例如，你用mvn的tomcat插件执行项目时）。

