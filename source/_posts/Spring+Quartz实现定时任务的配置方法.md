---
title: Spring+Quartz实现定时任务的配置方法
date: "2018/04/08"
tags: ['Spring Quartz', '定时任务']
categories: [spring]
copyright: true
---
实现比较麻烦，建议看[另一篇](/2018/04/08/spring用Scheduled注解方式实现定时任务)实现使用注解的方式，更简洁。
普通类方法。例子主要功能，每分钟输出“everyMinute”，每天18点输出“hours”

###  1.增加依赖库
```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>4.3.10</version>
    </dependency>
     
    <dependency>
        <groupId>org.quartz-scheduler</groupId>
        <artifactId>quartz</artifactId>
        <version>2.2.2</version>
    </dependency>
```
###  2.普通类
输出类  
```java
    package com.qq.fresh.testqmq;
     
    import lombok.extern.slf4j.Slf4j;
     
    /**
     * TestQuartz
     *
     * @author chenliclchen
     * @date 17-11-2 下午5:19
     */
    @Slf4j
    public class TestQuartz {
     
     public void everyMinute(){
        log.info("everyMinute");
     }
     
     public void hours(){
        log.info("hours");
     }
    }
```
###  3.业务类的配置
```xml
    <!--业务类的配置-->
    <bean id="job" class="com.qq.fresh.testqmq.TestQuartz"/>
```
###  4.JobDetail的配置
```xml
    <!--JobDetail的配置-->
    <bean name="minute"
     class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
        <property name="targetObject" ref="job"></property>
        <property name="targetMethod" value="everyMinute"></property>
    </bean>
    <bean name="hours"
     class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
        <property name="targetObject" ref="job"></property>
        <property name="targetMethod" value="hours"></property>
    </bean>
```
###  5.Trigger的配置
定时的cron定义 主要是  秒 分 时 日 月 周 年(可选) 特殊字符主要代表什么，可参看下面的2/3链接.  
```xml
    <!--Trigger的配置-->
    <bean id="everyMinute" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="minute"></property>
        <property name="startDelay" value="0"></property>
        <property name="cronExpression" value="0 */1 * * * ?"></property>
    </bean>
    <bean id="anHours" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="hours"></property>
        <property name="cronExpression" value="0 0 18 * * ?"></property>
    </bean>
```
###  6.Scheduler的配置
```xml
    <!--Scheduler的配置-->
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref bean="everyMinute"/>
                <ref bean="anHours"/>
            </list>
        </property>
        <property name="quartzProperties">
            <props>
                <prop key="org.quartz.threadPool.threadCount">1</prop>
            </props>
        </property>
    </bean>
```
参考：  

1\. [ http://blog.csdn.net/javawebxy/article/details/50492409
](http://blog.csdn.net/javawebxy/article/details/50492409)

2\. [ http://www.cnblogs.com/happyday56/p/4164877.html
](http://www.cnblogs.com/happyday56/p/4164877.html)

3\. [ http://www.cnblogs.com/henuyuxiang/p/4152805.html
](http://www.cnblogs.com/henuyuxiang/p/4152805.html)

  
  

  

