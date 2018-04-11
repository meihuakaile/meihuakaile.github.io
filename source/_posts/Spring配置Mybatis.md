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

# spring配置mybatis
1、上面的sqlSession是下面代码中的sqlSessionTemplate的实例。
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
mysql.uname=chenliclchen
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
        <property name="basePackage" value="com.qunar.data.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
```
