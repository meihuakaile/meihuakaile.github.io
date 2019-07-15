---
title: spring aop的所遇到的问题
date: "2018/04/08"
tags: ['SPRING AOP的所遇到的问题']
categories: [spring]
copyright: true
---
报错：
```
Post-processing of the FactoryBean's object failed; nested exception is org.springframework.aop.framework.AopConfigException: Could not generate CGLIB subclass of class [class com.sun.proxy.$Proxy68]: Common causes of this problem include using a final class or a non-visible class; nested exception is java.lang.IllegalArgumentException: Cannot subclass final class class com.sun.proxy.$Proxy68: java.lang.IllegalArgumentException: Cannot subclass final class class com.sun.proxy.$Proxy68
```
分析原因
代理了final修饰的类。可是哪里来的final类？ 原来，DAO层使用的是mybatis，可以只写接口不用写实现类。而我们项目中就是没有写实现类。
只在使用mybatis时，dao只有接口的方案时出了错，如果dao层仍然使用sqlSession的select方法不会出错。

详细：http://sparkgis.com/java/2017/08/%E8%AE%B0%E4%B8%80%E6%AC%A1spring-aop%E7%9A%84%E6%89%80%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98/