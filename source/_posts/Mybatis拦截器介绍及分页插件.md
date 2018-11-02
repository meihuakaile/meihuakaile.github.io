---
title: 'Mybatis拦截器介绍及分页插件'
date: "2018/05/31"
tags: [mybatis]
categories: ['mybatis']
copyright: true
---
Mybatis提供了一个Interceptor接口，通过实现该接口就可以定义我们自己的拦截器。该接口中一共定义有三个方法，intercept、plugin和setProperties。plugin方法是拦截器用于封装目标对象的，通过该方法我们可以返回目标对象本身，也可以返回一个它的代理。当返回的是代理的时候我们可以对其中的方法进行拦截来调用intercept方法，当然也可以调用其他方法，这点将在后文讲解。setProperties方法是用于在Mybatis配置文件中指定一些属性的。

对于实现自己的Interceptor而言有两个很重要的注解，一个是@Intercepts，其值是一个@Signature数组。
@Intercepts用于表明当前的对象是一个Interceptor，而@Signature则表明要拦截的接口、方法以及对应的参数类型。

Plugin的wrap方法，它根据当前的Interceptor上面的注解定义哪些接口需要拦截，然后判断当前目标对象是否有实现对应需要拦截的接口，如果没有则返回目标对象本身，如果有则返回一个代理对象。

插件PageHelper通过实现Interceptor接口实现了物理分页。
使用方法：
1、mvn
```xml
<dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper</artifactId>
      <version>3.7.3</version>
</dependency>
```
2、在mybatis的配置文件中写上
```xml
<plugins>
        <!--<plugin interceptor="com.data.intercepts.MyIntercept"></plugin>-->
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <property name="dialect" value="mysql"/>
        </plugin>
</plugins>
```
参数`dialect`的值代表连接数据库类型。
3、直接使用
```java
Page page = PageHelper.startPage(1, 1, true);
select ...
```
PageHelper必须在执行sql前调用，只对第一个执行的sql有效。
`PageHelper.startPage()`参数：
`pageNum` 页码，与`RowBound`的`offset`不同，本参数使用**_1_**做为起始值。
`pageSize` 每页显示数量
`count` 是否进行count查询，为true时同时得到数据库中此条sql的总数。
`reasonable` 分页合理化,null时用默认配置
`pageSizeZero` 当设置为true的时候，如果pagesize设置为0（或RowBounds的limit=0），就不执行分页，返回全部结果
通过看源码可以看到，物理分页还是通过limit实现，
count操作是通过`select count(1) from origin_sql`实现，使用时想要得到总条目数/总页数等使用`Page`的方法。
所以从效率上来说和直接limit应该是一样的，只是代码更简洁，扩展性更好了。
还可以拦截RowBound()，但是使用起来非常不方便，只能有limit效果，其他都要搭配Page才能使用，还不如直接用Page。

出错：
`java.lang.NoSuchMethodError: org.apache.ibatis.reflection.MetaObject.forObject(Ljava/lang/Object;Lorg/apache/ibatis/reflection/factory/ObjectFactory;`
原因：mybatis和PageHelper插件的版本不匹配或者PageHelper3.6.3版本的问题，修改PageHelper版本即可。