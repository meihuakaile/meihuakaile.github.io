---
title: Mybatis.xml配置
date: 2018-03-29 15:49:38
tags: [mybatis]
categories: ['mybatis']
copyright: true
---


### mapUnderscoreToCamelCase
```xml
<setting name="mapUnderscoreToCamelCase" value="true"/>
```
设为true时，代码把数据库中的带“_”的字段名和java model中的驼峰相对应。（其实这样会有很多的不好的地方）

### useGeneratedKeys
强制允许jdbc支持生成key。
```xml
<setting name="useGeneratedKeys" value="true"/>
```
Allows JDBC support for generated keys. A compatible driver is required.
This setting forces generated keys to be used if set to true,
as some drivers deny compatibility but still work

### typeAliases
xml里有<strong>parameterType</strong>  如果不在mybatis.xml中指定：
```xml
<typeAliases>
    <package name="com.data.model"/>
</typeAliases>
```
parameterType就要把实体类的包路径完全写下来。写了上面的package后会自动去这个包下面找，只需要写类名不需要包路径。
总结自：http://blog.csdn.net/lelewenzibin/article/details/42713585
### 最简单的xml配置例子
```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="useGeneratedKeys" value="true"/>
    </settings>
    <typeAliases>
        <package name="com.atp.model"/>
    </typeAliases>
</configuration>
```
