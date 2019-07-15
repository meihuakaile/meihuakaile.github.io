---
title: 'spring的线程安全'
date: "2018/04/20"
tags: [spring]
categories: [spring]
copyright: true
---
### 参数不安全

解决办法就是：有状态的变量本地化。spring框架也有这样的实现，spring的dao是单例的，也就是线程安全的，但是按说每个方法都有一个数据库连接Connection，Connection的值肯定不是线程安全，spring为了解决这个问题就是把Connection本地化了，就是下面例子里的方法，把Connection赋值给本地的Connection变量。
```java
@Service
public class TestCur{
	//private Map<String, String> map = Maps.newHashMap();
	public void deal(Map map1){
		//此处要对map1进行一些计算等操作。在多线程的情况下，又无法确保参数map1是安全的，考虑在此方法里先把map1深拷贝
		//（只是值拷贝，也不算是深拷贝）给方法里的本地变量，这样就不用担心其他线程在方法外面对参数map进行修改而影响本方法。
		//例如公司的qconfig，不能直接给类的成员变量（如，上面的map），只能先深拷贝给方法本地的map再赋值给类成员变量map。
	}
}
```
### spring中的线程安全
在spring中加了spring注解的都是单例，不在类里写可变状态的类成员变量时，就不用考虑线程不安全的问题。写了可变状态变量的就比如1中的map。
原因是：如果控制器是使用单例形式，且controller中有一个私有的变量a,所有请求到同一个controller时，使用的a变量是共用的，即若是某个请求中修改了这个变量a，则，在别的请求中能够读到这个修改的内容。
解决方法有两个：
（1）写一个treadlocal，把a本地化（1中提到的spring解决Connection就是用的这种办法，只不过spring需要管理的太多用了treadlocalmap）
（2）在spring的配置文件中为这个controller写上scope="prototype"，定义为非单例的：
`<bean id="userController" class="com.qunar.data.controller.UserController" scope="prototype"></bean>`  
但最好不要在想要线程安全的类里写类成员变量。
### 其他
另外数据库连接方面，只要使用spring 自己实现的dao，如JdbcTemplate，就不会发生数据库连接泄露的问题。但是如果手动调用Connection而忘记关闭就很可能会导致数据库连接泄露问题。
`DataSourceUtils.getConnection()`方法会首先查看当前是否存在事务管理上下文，如果存在就尝试从事务管理上下文拿连接，如果获取失败，直接从数据源中拿。在获取连接后，如果存在事务管理上下文则把连接绑定上去。

spring在servlet上扩展，都是线程安全。