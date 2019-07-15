---
title: Spring配置Mybatis
date: 2018-02-01 16:52:13
tags: [database, mybatis]
categories: [spring]
copyright: true
---
　　mybatis的使用有多种办法。如果使用了spring，可以使用最简单的办法，在spring的配置文件中引入mapper.xml和mybatis.xml（本文也只讲解了这种方法）。
# 使用spring+mybatis依赖包
```xml
<mybatisspring.version>1.3.1</mybatisspring.version>
<mybatis.version>3.4.4</mybatis.version>
<mysql.version>5.1.30</mysql.version>

<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>${mybatisspring.version}</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>${mybatis.version}</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql.version}</version>
</dependency>
```
# mybatis.xml
　　首先，一个mybatis.xml文件做部分配置，如[MyBatis.xml配置](/2018/03/29/Mybatis-xml配置/)
# mapper.xml
　　之后还需要mapper.xml，可能需要多个，一般和dao层类对应。
mapper.xml中需要注意的是namespace，目前见到比较常用的两种办法：
* 在外面写一个接口类，namespace写成这个接口类的名字，接口类的方法写成mapper里的sql id，在service层直接通过接口名+方法名(namespace+sql id)使用。<strong>有多个参数时，用@Param("name") 标记多个参数，name是mapper中使用这个参数时的名字。</strong>
* 第一种方法比较局限，dao层是一个接口，主要实现在mapper中，如果想在dao层中加别的代码（一般也不建议加别的逻辑）就无法实现。

第二种方法是得到spring配置中的sqlSession，通过它的selectOne等方法执行mapper，这时候mapper的namespace可以是自己取得名字，selectOne的第一个参数是namespace+sql id，第二个参数是sql语句需要的参数。
但是某人说在网上看的不建议使用第二种方法，原因他也忘记了，可能是不方便。如果使用方法1，在idea上装一些插件可以通过dao接口的接口方法点到mapper里，而且ctrl+F6修改接口名时mapper里也会自动修改，说起来确实方便了很多。

# spring配置mybatis
1、上面的sqlSession是下面代码中的sqlSession的实例。
2、下面代码中的
```xml
<property name="configLocation" value="classpath:mybatis.xml"/>
<property name="mapperLocations" value="classpath*:mapper/*.xml"/>
```
配置了第一二步的mybatis.xml和mapper.xml文件。

jdbc.properties：
``` 
driverClassName=com.mysql.jdbc.Driver
mysql.url=jdbc:mysql://localhost:3306/test?autoReconnect=true&connectTimeout=10000&socketTimeout=60000&useUnicode=true&characterEncoding=UTF-8
mysql.uname=cl
mysql.passWord=1234
```
spring配置：
```xml
<context:property-placeholder location="classpath:properties/jdbc.properties" ignore-unresolvable="true"/>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
    init-method="init" destroy-method="close">
        <property name="driverClassName" value="${driverClassName}"/>
        <property name="url" value="${mysql.url}"/>
        <property name="username" value="${mysql.uname}"/>
        <property name="password" value="${mysql.passWord}"/>
        <property name="maxActive" value="30"/>
        <property name="initialSize" value="10"/>
        <property name="minIdle" value="10"/>
        <property name="maxWait" value="200"/>

        <property name="timeBetweenEvictionRunsMillis" value="60000"/>
        <property name="minEvictableIdleTimeMillis" value="300000"/>

        <property name="validationQuery" value="SELECT 'x'" />
        <property name="testWhileIdle" value="true"/>
        <property name="testOnBorrow" value="true"/>
        <property name="testOnReturn" value="false"/>
        <property name="poolPreparedStatements" value="true"/>
        <property name="maxPoolPreparedStatementPerConnectionSize" value="20"/>

    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis.xml"/>
        <property name="mapperLocations" value="classpath*:mapper/*.xml"/>
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.data.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
```
# 进阶
SqlSessionFactoryBean是一个工厂bean，它的作用就是解析配置（数据源、别名等）。
有些项目有很多的数据源，然后可能会把所有的mapper定义在同一个文件（配置mybatis的文件里用mappers参数定义，就是上面的mybatis.xml文件）里，mapperLocations不用。
然后这种情况spring怎么知道你的sqlSession是为谁服务的呢？那就看MapperScannerConfigurer的配置参数markerInterface使用接口过滤/annotationClass使用注解过滤了。

MapperScannerConfigurer：
`basePackage`：扫描器开始扫描的基础包名，支持嵌套扫描；
`sqlSessionTemplateBeanName`：前文提到的模板bean的名称；
`markerInterface`：基于接口的过滤器，实现了该接口的dao才会被扫描器扫描，与basePackage是与的作用。
`annotationClass`：基于注解的过滤器，配置了该注解的dao才会被扫描器扫描，与basePackage是与的作用。

## markerInterface接口过滤：
```xml
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis.xml"/>
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.data.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <property name="markerInterface" value="com.data.dao.UserDao" />  
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
```
`com.data.dao.UserDao`是一个接口，实现这个接口，且在com.data.dao下 的类会使用这个sqlSession

## annotationClass注解过滤：
```xml
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis.xml"/>
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.data.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <property name="annotationClass" value="com.data.dao.UserDao" />  
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
```
`com.data.dao.UserDao`是一个自定义注解，被这个注解标注，且在com.data.dao下 的类会用这个sqlSession。
自定义注解例子：
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Component
public @interface UserDao {
    /**
     * The value may indicate a suggestion for a logical component name,
     * to be turned into a Spring bean in case of an autodetected component.
     * @return the suggested component name, if any
     */
    String value() default "";
}
```
进阶部分参考：https://blog.csdn.net/hupanfeng/article/details/21454847