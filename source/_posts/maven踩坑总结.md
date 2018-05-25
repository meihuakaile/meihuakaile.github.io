---
title: 'maven踩坑总结'
date: "2018/04/20"
tags: [maven]
categories: [开发常用工具]
copyright: true
---
1.关于mvn的使用总结。
（1）把每个子模块中要使用的依赖都放在父pom的dependencyManagement中，所有version放在properties中。
（2）把几乎每个子模块都需要的依赖包放在父pom的dependencies中，不需要version。如，junit，slf4j等
（3）每个子模块单独需要的放在该子模块的pom中，不要version。
（4）spring中的一些包是互相包含的，但是在项目中要把所有的spring包都写上，写在dependencyManagement中。

（4）的原因：如果父pom中有spring，我们的项目中也加了spring的包，这个时候如果我们只引用了部分的spring包，然后这个部分的spring包包括了其他的spring包，这个其他的spring包如果在dependencyManagement没有指定版本就可能会导致spring包版本不一致。
如context自动导入aop，而且dependencyManagement没有指定aop版本，项目会去上一层dependencyManagement（即父pom）中找aop版本，这个时候就可能导致aop包的版本冲突（父pom的aop版本和引入的context包含的aop版本版本不一样）。

（5）某些情景中（比如公司强制）强制不让引入commons-logging包,但是spring-core中默认会导入commons-logging包。
SpringJUnit4ClassRunner中使用了commons-logging中的类，如果直接用exclusions，会抛出异常。
解决办法，加入依赖包：
```xml
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>jcl-over-slf4j</artifactId>
	<version>${jcl.version}</version>
</dependency>
```
这个依赖包是一个桥接工具，可以把原本的log重定向到slf4j，其他类似的包还有log4j-over-slf4j等。
如果同时使用了jcl-over-slf4j和slf4j-jcl的话,会导致jcl代理给slf4j,slf4j又绑定到jcl,就形成了一个死循环,抛出StackOverFlow异常。