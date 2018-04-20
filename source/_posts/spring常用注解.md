---
title: 'spring常用注解'
date: "2018/04/20"
tags: [spring]
categories: [spring]
copyright: true
---

## 接受前端数据类
### @PathVariabl
获取路径中传递参数 ，eg：
```java
@RequestMapping(value = "/{id}/{str}") 
public ModelAndView helloWorld(@PathVariable String id,  @PathVariable String str) { }
```
### @ModelAttribute
获取POST请求的FORM表单数据 。（实际是，不用@ModelAttribute也可以接收到数据）eg：
JSP
```jsp 
 <form method="post" action="hao.do"> 
 a: <input id="a" type="text"   name="a"/> 
 b: <input id="b" type="text"   name="b"/> 
 <input type="submit" value="Submit" /> 
 </form> 
```
JAVA pojo
```java 
public class Pojo{ 
     private String a; 
     private int b; 
```
JAVA controller
```java 
@RequestMapping(method = RequestMethod.POST) 
     public String processSubmit(@ModelAttribute("pojo") Pojo pojo) { 
         return "helloWorld"; 
     } 
```
### @RequestParam
get请求。绑定请求参数a到变量a 。
解惑：为何不用这个注解也可以接收到参数，那还加这个注解有什么用。
例1： 
```java
@RequestMapping(value = "/requestParam", method = RequestMethod.GET) 
 public String setupForm( String a, ModelMap model) {}
```
例2：
```java
 @RequestMapping(value = "/requestParam", method = RequestMethod.GET) 
 public String setupForm(@RequestParam String a, ModelMap model) {}
 ```
例3：
```java
 @RequestMapping(value = "/requestParam", method = RequestMethod.GET) 
 public String setupForm(@RequestParam{required=false} String a, ModelMap model) {}
 ```
例4：
```java
 @RequestMapping(value = "/requestParam", method = RequestMethod.GET) 
 public String setupForm(@RequestParam{defaultValue="0"} String a, ModelMap model) {}
 ```
例5：
```java
 @RequestMapping(value = "/requestParam", method = RequestMethod.GET) 
 public String setupForm(@RequestParam{value="id"} String a, ModelMap model) {}
 ```
例1和例2都能接受到参数，但是对与例2来说url "/requestParam"的后面一定要有参数，没有会报错；但是例1参数可有可无。
但是例2也可以通过设置required=false来指定不一定要参数(就是例3)，就可例1就不行了.
通过设置defaultValue="0" 可以在没有参数时指定默认值。
通过设置value="id" 给参数换成其他名字。

当请求参数a不存在时会有异常发生,可以通过设置属性required=false解决, 
例如: @RequestParam(value="a", required=false)
```java
 @RequestMapping(value = "/requestParam", method = RequestMethod.GET) 
 public String setupForm(@RequestParam(value="a", required=false) String a, ModelMap model) {}
 ```
### @RequestMapping
处理请求地址映射的注解。
三类属性：
（1）value、method。前者是url，后者是请求类型post/get/put/delete等。
（2）produces、consumes。前者是只接受请求的规定返回值（accept）是某值的请求，如application/json。后者是只接受请求值是某种类型的值的请求。
（3）params、header。只接受参数/头包含某内容的请求。
eg：
```java
@ResponseBody
@RequestMapping(value = "/portrait/tags",produces="application/json;charset=UTF-8")
 public String getTagList(String callback)
```
参考：https://www.cnblogs.com/qq78292959/p/3760560.html

## spring参数类
### @Value
假设有一个test.properties，内容：`testname=li`
方法1:
在spring的配置文件中配置：
```xml
<bean id="testProperties" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <list>
            <value>classpath*:/test.properties</value>
			<!--添加多个properties配置文件-->
        </list>
    </property>
    <property name="ignoreUnresolvablePlaceholders" value="true"/>
</bean>
```
或者
```xml
<context:property-placeholder location="classpath*:test.properties"  ignore-unresolvable="true"/>
```
使用，注意类的成员变量name的注解${}里的值就是properties里的key：
```java
@Slf4j
@Component
public class TestProper {

    @Value("${testname}")
    public String name;

    public void testProperties(){
        log.info("name:{}", name);
    }
}
```
方法2
在spring中配置：
```xml
<bean id="properFactory" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
    <property name="locations">
        <list>
            <value>classpath*:/test.properties</value>
        </list>
    </property>
</bean>
```
使用，注意类的成员变量name的注解  properFactory是上面spring配置的id  testname是properties配置文件里值的key：
```java
@Slf4j
@Component
public class TestProper {

    @Value("#{properFactory['testname']}")
    public String name;

    public void testProperties(){
        log.info("name:{}", name);
    }
}
```