---
title: spring 自定义注解实现登陆拦截
date: "2018/04/08"
tags: ['spring 自定义注解', '登陆验证']
categories: [spring]
copyright: true
---
需求：  
自定义一个注解，使得controller层的类或者方法在写上这个注解后，会有登陆验证。  
  
实现原理：  
（1）先写一个自定义注解，注解上可以通过注释确定是类/方法可以加此注解。  
（2）之后，写一个拦截器，拦截器内可以通过handler得到被拦截器拦截的类或者方法，之后可以通过这个类/方法得知它是否有之前写的注解，如果有，就需要登陆校
验。  
（3）之后要把这个拦截器配置到spring-mvc的配置文件中，需要让spring知道有哪些请求需要被拦截器拦截，一般是“/**”，是所有请求。  
（4）在controller的类或者方法上加上步骤一中的的自定义注解就可以轻易实现是否需要登陆校验了。  
  
例子：  
#####  1.自定义注解
```java
    package com.qunar.fresh.annotation;
    
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    import java.lang.annotation.Target;
    
    /**
    * LoginRequired
    *
    * @author chenliclchen
    * @date 17-10-11 下午8:30
    */
    @Target({ElementType.METHOD, ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    public @interface LoginRequired {
    
    }
```
@Target({ElementType.METHOD, ElementType.TYPE}) 表示类和方法都可以加此注解 
Retention(保留)注解说明,这种类型的注解会被保留到那个阶段. 有三个值:
1.RetentionPolicy.SOURCE —— 这种类型的Annotations只在源代码级别保留,编译时就会被忽略
2.RetentionPolicy.CLASS —— 这种类型的Annotations编译时被保留,在class文件中存在,但JVM将会忽略
3.RetentionPolicy.RUNTIME —— 这种类型的Annotations将被JVM保留,所以他们能在运行时被JVM或其他使用反射机制的代码所读取和使用.
还可以再写上@Documented注解：
 Documented 注解表明这个注解应该被 javadoc工具记录. 默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理, 所以注解类型信息也会被包括在生成的文档中. 

其他相关注解：https://www.jb51.net/article/55371.htm
#####  2.写一个拦截器
```java
    package com.qunar.fresh.interceptor;
    
    import com.qunar.fresh.annotation.LoginRequired;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.stereotype.Component;
    import org.springframework.web.method.HandlerMethod;
    import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.lang.reflect.Method;
    
    /**
     * AuthInterceptor
     *
     * @author chenliclchen
     * @date 17-10-11 下午8:38
     */
    @Slf4j
    @Component
    public class AuthInterceptor extends HandlerInterceptorAdapter{
    
     @Override
     public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    	// return super.preHandle(request, response, handler);
     	if(handler instanceof HandlerMethod){
     		HandlerMethod myHandlerMethod = (HandlerMethod) handler;
     		Object bean = myHandlerMethod.getBean();
     		Method method = myHandlerMethod.getMethod();
     		LoginRequired classAnnotation = bean.getClass().getAnnotation(LoginRequired.class);
     		LoginRequired methodAnnotation = method.getAnnotation(LoginRequired.class);
     		if(classAnnotation != null || methodAnnotation != null){
     			boolean loginState = isLogin(request, response);
     			if(loginState){
     				return true;
     			}
     			if(isAjax(request)){
     				//ajax 请求需要返回json
     				log.info("ajax 请求，没有登陆");
     			}else{
     				log.info("普通请求，没有登陆");
     			}
     			return false;
     		}
     	}
     	return true;
     }
    
     private boolean isLogin(HttpServletRequest request, HttpServletResponse response){
     	return false;
     }
     private boolean isAjax(HttpServletRequest request){
     	String requestType = request.getHeader("X-Requested-With");
     	if(requestType != null && requestType.equals("XMLHttpRequest")){
     		return true;
     	}else{
     		return false;
     	}
     }
    }
```
在isLogin方法中写上是否登陆的验证。  
  
原理是先通过myHandlerMethod.getBean()得到bean，
再通过bean.getClass().getAnnotation(LoginRequired.class)的返回值确定是否有LoginRequired（即步骤1）中的自定义注解。  
通过myHandlerMethod.getMethod()也可以得到method。
通过method.getAnnotation(LoginRequired.class)得到此方法是否有自定义注解。  

也可以通过下面的方法得到是否有LoginRequired 自定义注解。  
```java
    HandlerMethod handlerMethod = (HandlerMethod) handler;
    if (!handlerMethod.getBeanType().isAnnotationPresent(LoginRequired.class)
     && !handlerMethod.getMethod().isAnnotationPresent(LoginRequired.class)) {
     	return false;
    }
    return true;
```
#####  3.配置springmvc文件
```xml
    <mvc:interceptors>
    	<mvc:interceptor>
     		<mvc:mapping path="/**"/>
     		<ref bean="authInterceptor"/>
     	</mvc:interceptor>
    </mvc:interceptors>
```
#####  4.使用例子
```java
    package com.qunar.fresh.controller;
    
    import com.qunar.fresh.annotation.LoginRequired;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;
    
    /**
     * TestController
     *
     * @author chenliclchen
     * @date 17-10-11 下午9:05
     */
    @LoginRequired
    @Controller
    @RequestMapping("/test")
    public class TestController {
    
     @RequestMapping("/one")
     public String test(){
     	return "test";
     }
    }
```
这样，访问以/test开头的所有url都会有是否登陆的验证。

