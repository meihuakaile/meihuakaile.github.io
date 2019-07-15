---
title: ignore-resource-not-found和ignore-unresolvable
date: "2018/04/08"
tags: ['ignore-unresolvable', 'ignore-resource-not-found']
categories: [spring]
copyright: true
---
.ignore-resource-not-found和ignore-unresolvable两个属性是类似的作用（网上说推荐配对使用，但很少看到配对使用的）

如果location中的文件指向了一个不存在的文件（在没有指定上面两个参数的情况下，spring也并不会报错），那么也极有可能意味着有属性无法解析（虽然存在其他属性文件中存在重名，但是这个是应该避免的，  所以当ignore-resource-not-found设为true时，ignore-unresolvable也必须设为true。）
                       \-------------------------------------来自网络
其实当ignore-unresolvable设为true时，ignore-resource-not-found的值true或false，并不影响异常的抛出。  
如果设置为ture，后属性值无法解析成功，将赋值为${属性名}

** ignore-resource-not-found： ** 如果属性文件找不到，是否忽略，默认false，即不忽略，找不到文件并不会抛出异常。    
** ignore-unresolvable： ** 是否忽略解析不到的属性，如果不忽略，找不到将抛出异常。但它设置为true的主要原因是： 

** 理解：ignore-unresolvable为true时，配置文件${}找不到对应占位符的值 不会报错，会直接赋值'${}'；如果设为false，会直接报错。 
设置它为true的主要原因，是一个xml中有多个配置文件时的情况： **
同个模块中如果出现多个context:property-placeholder ，location properties文件后，运行时出现 
_ **Could not resolve placeholder 'key' in string value${key1} ** _
原因是在加载第一个context:property-placeholder时会扫描所有的bean，而有的bean里面出现第二个 context:property-placeholder引入的properties的占位符${key2}，但此时还没有加载第二个property-placeholder，所以解析不了${key2}。

办法一，可以将通过模块的多个property-placeholder合并为一个，将初始化放在一起(不推荐)。
方法二，添加ignore-unresolvable="true"，这样可以在加载第一个property-placeholder时出现解析不了的占位符进行忽略掉。

参考：  [ http://blog.csdn.net/Rickesy/article/details/50791534
](http://blog.csdn.net/Rickesy/article/details/50791534) (比较多乱)  

