---
title: 'mapper部分语法 CDATA foreach resultMap typeHandlers'
date: "2018/04/19"
tags: ['CDATA', 'foreach', 'resultMap', typeHandlers, 'mybatis']
categories: ['mybatis']
copyright: true
---
### CDATA
总结自：http://blog.csdn.net/glory1234work2115/article/details/51695540
在 XML 元素中，"<" 和 "&" 是非法的。
"<" 会产生错误，因为解析器会把该字符解释为新元素的开始。
"&" 也会产生错误，因为解析器会把该字符解释为字符实体的开始。

一般要写成：

| Item     | Value     | Qty   |
| :-------:| :--------:| :---: |
| &amp;lt;     | <         | 小于     |
| &amp;gt;     | >         | 大于    |
| &amp;amp;    | &         | 和号   |
| &amp;apos;   | '         | 省略号   |
| &amp;quot;   | "         | 引号   |
注释：严格地讲，在 XML 中仅有字符 "<"和"&" 是非法的。省略号、引号和大于号是合法的，但是把它们替换为实体引用是个好的习惯。
但是用了“<![CDATA[”之后就可以直接用'<'、‘>’等了。
CDATA 部分中的所有内容都会被解析器忽略。
CDATA 部分由 "<![CDATA[" 开始，由 "]]>" 结束。

关于 CDATA 部分的注释：
CDATA 部分不能包含字符串 "]]>"。也不允许嵌套的 CDATA 部分。
标记 CDATA 部分结尾的 "]]>" 不能包含空格或折行。
### foreach
来自：http://blog.csdn.net/jason5186/article/details/40896043

| 属性     | 描述     |
| :-: | :------------------------------------ |
| item | 循环体中的具体对象。支持属性的点路径访问，如item.age,item.info.details。具体说明：在list和数组中是其中的对象，在map中是value。该参数为必选。|
| collection | 要做foreach的对象，作为入参时，List<?>对象默认用list代替作为键，数组对象有array代替作为键，Map对象用map代替作为键。</br>当然在作为入参时可以使用@Param("keyName")来设置键，设置keyName后，list,array,map将会失效。 除了入参这种情况外，还有一种作为参数对象的某个字段的时候。举个例子：</br>如果User有属性List ids。入参是User对象，那么这个collection = "ids"</br>如果User有属性Ids ids;其中Ids是个对象，Ids有个属性List id;入参是User对象，那么collection = "ids.id"</br>上面只是举例，具体collection等于什么，就看你想对那个元素做循环。该参数为必选。|
| separator | 元素之间的分隔符，例如在in()的时候，separator=","会自动在元素中间用“,“隔开，避免手动输入逗号导致sql错误，如in(1,2,)这样。该参数可选。  |
| open| foreach代码的开始符号，一般是(和close=")"合用。常用在in(),values()时。该参数可选。  |
| close| foreach代码的关闭符号，一般是)和open="("合用。常用在in(),values()时。该参数可选。  |
| index | 在list和数组中,index是元素的序号，在map中，index是元素的key，该参数可选。|
 
简单例子：
```xml
<insert id="tmpInsertCount" parameterType="list">
          INSERT INTO `flight_qmq_order_count_tmp`(
          countTime,
          attribute,
          value
          )VALUES
         <foreach collection="list" item="item" separator=","> 
			(#{item.countTime}, #{item.attribute}, #{item.count}) 
		</foreach>
</insert>
```
### resultType  or   resultMap
mybatis推荐自定义类使用resultMap，而不用resultType，原因：
（1）resultMap适合返回值是自定义实体类的情况。需要在mapper中自定义resultMap，把数据库中的列和model对应起来。
（2）resultType适合使用返回值的数据类型是非自定义。如果想用自定义类型就必须命名规范，需要先设置mapUnderscoreToCamelCase为true，之后必须和数据库字段的下划线对应成驼峰。
（3）将来如果Model中的成员变量名字变了，resultMap只需要到Mapper中改对应resultMap；而resultType需要到数据库里改表字段的名字，如果其他项目用了这个数据库，还要改其他项目的东西。把问题的影响范围缩小的考虑，应该使用resultMap。
（4）而且查询时列名想用别名时就需要resultMap。

resultMap 需要自己定义，将数据库中的字段和model对应，简单例子：
```xml
<resultMap id="hotelInfoResultMap" type="HotelInfoModel">
    <id column="id" property="id"/>
    <result column="hotel_name" property="hotelName"/>
    <result column="city" property="city"/>
    <result column="price" property="price"/>
    <result column="level" property="level"/>
</resultMap>
```
resultMap的属性id在写上去了时作为resultMap的值；属性type是model的值。
column是数据库字段名，property是model字段名。

### typeHandlers
如果有自己的类型想默认转成数据库的类型，可以配置<typeHandlers>，它需要自己写一个类，数据库的读写都会默认扫描这个类，把对应的数据转换。
例如，一种应用场景：公司存小数用的money，存到数据库时想默认转成float等，就可以配一个typeHandlers。
```xml
<plugins>
    <plugin interceptor="XXX"/>
    <plugin interceptor="XXX"/>
</plugins>
```