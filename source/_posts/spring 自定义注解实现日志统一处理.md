---
title: spring 自定义注解实现日志统一处理
date: "2018/04/08"
tags: ['spring 自定义注解', '日志统一处理']
categories: [spring]
copyright: true
---
需求：
通过注解的方式 统一处理controller和service的日志（实现上可能不太严谨，主要是实现流程）

原理：
先自定义注解。用aop切面拦截方法的使用，看是否有对应的自定义的注解，如果有，在切面中进行日志的统一打印，可以获取到加了注解方法的类名、方法名、参数。

如果想每个方法传进来不同信息，可以在自定义注解里写上参数，这样在使用时就可以带进来不同信息。例如，spring自带的注解@Resource（name=“”）。

#####  1.controller、service自定义注解
```java
    package com.fresh.annotation;
     
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    import java.lang.annotation.Target;
     
    /**
     * ControllerLog
     *
     * @author chenliclchen
     * @date 17-10-12 上午11:35
     */
    @Target({ElementType.METHOD, ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    public @interface ControllerLog {
        String description() default "";
    }
    
    
    package com.fresh.annotation;
     
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    import java.lang.annotation.Target;
     
    /**
     * ServiceLog
     *
     * @author chenliclchen
     * @date 17-10-12 上午11:34
     */
    @Target({ElementType.METHOD, ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    public @interface ServiceLog {
        String description() default "";
    }
```
可以通过需求加多个如同description的注解。  

#####  2.写aop进行切面
```java
    package com.fresh.aop;
     
    import com.fresh.annotation.ControllerLog;
    import com.fresh.annotation.ServiceLog;
    import lombok.extern.slf4j.Slf4j;
    import org.apache.commons.lang3.StringUtils;
    import org.apache.commons.lang3.text.StrBuilder;
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.*;
    import org.springframework.stereotype.Component;
    import java.lang.reflect.Method;
    import java.util.Arrays;
     
    /**
     * LogAop
     *
     * @author chenliclchen
     * @date 17-10-12 上午11:40
     */
    @Slf4j
    @Component
    @Aspect
    public class LogAop {
     
     @Pointcut("@annotation(com.fresh.annotation.ServiceLog)")
     public void serviceAspect(){
        log.info("service 日志");
     }
     
     @Pointcut("@annotation(com.fresh.annotation.ControllerLog)") //
     public void controllerAspect(){
        log.info("controller 日志");
     }
     
     @Before("controllerAspect() || serviceAspect()")
     public void doBefore(JoinPoint joinPoint){
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        String description = getDescription(joinPoint);
        Object[] args = joinPoint.getArgs();
        String logResult = new StringBuilder().append(description).append("操作: ")
        .append(className).append("类的").append(methodName).append("()方法被请求，参数为： ")
        .append(Arrays.toString(args)).toString();
     
        log.info(logResult);
     }
     
    // @AfterThrowing(pointcut = "controllerAspect() || serviceAspect()", throwing = "e")
    // public void doAfterThrowing(JoinPoint joinPoint, Throwable e){
    //      log.error("统一处理异常日志 {}", e);
    // }
     
     private String getDescription(JoinPoint joinPoint){
        Class<?> className = joinPoint.getTarget().getClass();
        //得到类的所有方法
        Method[] methods = className.getMethods();
        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        //在类所有方法中找到对应的被拦截到的方法，通过方法的名字和参数的个数。
        for(Method method: methods){
            if(method.getName().equals(methodName)){
                Class<?>[] parameterTypes = method.getParameterTypes();
                //参数个数是否相等。
                if(parameterTypes.length == args.length){
                    ControllerLog controllerLog = method.getAnnotation(ControllerLog.class);
                    if(controllerLog != null){
                        return controllerLog.description();
                    }else{
                        ServiceLog serviceLog = method.getAnnotation(ServiceLog.class);
                        return serviceLog != null? serviceLog.description(): "";
                    }
                }
            }
        }
        return "";
     }
    }
```
只写了before，如果有需求还可以写上after等。  

#####  3.xml配置
```xml
    <aop:aspectj-autoproxy proxy-target-class="true"/>
```
**_需要注意的是，如果项目的spring-mvc.xml和applicationContext.xml是两个文件，而你的切面是同时面向controller和service的，两个配置文件里都需要加上这行配置。**_

#####  4.使用
```java
    package com.fresh.controller;
     
    import com.fresh.annotation.ControllerLog;
    import com.fresh.annotation.LoginRequired;
    import com.fresh.service.TestService;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestMapping;
     
    import javax.annotation.Resource;
     
    /**
     * TestController
     *
     * @author chenliclchen
     * @date 17-10-11 下午9:05
     */
    //@LoginRequired
    @Controller
    @RequestMapping("/test")
    public class TestController {
     
     @Resource
     TestService testService;
     
     @RequestMapping("/one/{name}")
     @ControllerLog(description="测试用aop统一处理日志")
     public String test(@PathVariable String name){
        testService.printHello(name);
        return "test";
     }
    }
    
    
    package com.fresh.service;
     
    /**
     * TestService
     *
     * @author chenliclchen
     * @date 17-10-12 下午6:32
     */
    public interface TestService {
     void printHello( String name);
    }
     
    package com.fresh.service.impl;
     
    import com.fresh.annotation.ServiceLog;
    import com.fresh.service.TestService;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.stereotype.Service;
     
    /**
     * TestServiceImpl
     *
     * @author chenliclchen
     * @date 17-10-12 下午6:33
     */
    @Slf4j
    @Service
    public class TestServiceImpl implements TestService {
     
     @Override
     @ServiceLog(description = "测试aop统一处理service日志输出")
     public void printHello(String name) {
        log.info("hello {}", name);
     }
    }
```
这样，当浏览器输入：  [ http://localhost:8080/test/one/who
](http://localhost:8080/test/one/who) 时，会有如下输出日志：  
```
    INFO com.fresh.aop.LogAop - 测试用aop统一处理日志操作: com.fresh.controller.TestController类的test()方法被请求，参数为： [who]
    INFO com.fresh.aop.LogAop - 测试aop统一处理service日志输出操作: com.fresh.service.impl.TestServiceImpl类的printHello()方法被请求，参数为： [who]
```