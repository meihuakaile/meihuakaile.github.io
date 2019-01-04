---
title: 'jackson'
date: "2017/10/19"
tags: [java]
categories: ['java']
---
参考：http://www.cnblogs.com/kakag/p/5054772.html
http://blog.csdn.net/java_huashan/article/details/46375857

依赖：
```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-core</artifactId>
	<version>2.9.2</version>
</dependency>
```
Jackson 主要有三部分组成，除了三个模块之间存在依赖，不依赖任何外部 jar 包。
三个模块及artifactId：jackson-core、jackson-annotations、jackson-databind。

提供了三种使用方式：

`Streaming API` : 其他两种方式都依赖于它而实现，如果要从底层细粒度控制 json 的解析生成，可以使用这种方式;
`Tree Model` : 通过基于内存的树形结构来描述 json 数据。json 结构树由 JsonNode 组成。不需要绑定任何类和实体，可以方便的对 JsonNode 来进行操作。
`Data Binding` : 最常用的方式，基于属性的 get 和 set方法以及注解来实现 JavaBean 和 json 的互转，底层实现还是 Streaming API.

我目前实际遇到的：
第一种不长使用。
第二种在返回的json很大层次很深但是你只需要某几层的某几个数据时比较适合使用。
第三种就是常规的使用，经常使用。可以把得到的json直接转成相应的类别。


json有很多的属性可以设置，具体可以看上面参考链接里的第一个链接。

比如下面第三种时例子里的`mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)`;   
jsonString和自定义类转换时，当自定义类中有比jsonString少的属性时，设置了上面的值就不会报错，可以继续自动转换；可是没有设置时就会报错。
自定义类属性比jsonString多时jackson本身就不会报错。


第一种不再细说，因为不长使用。
第二种简单例子：
```java
#Jackson提供一个树节点被称为"JsonNode",ObjectMapper提供方法来读json作为树的JsonNode根节点
JsonNode jsonNode = JsonUtils.getMapper().readTree(jsonString);

#查看节点的类型
jsonNode.getNodeType()
#是不是一个容器

jsonNode.isContainerNode()
#得到 节点 下属性名

Iterator<String> fieldNames = jsonNode.fieldNames();
while(fieldNames.hasNext()){
 log.info(fieldNames.next());
}
#asText的作用是有值返回值，无值返回空字符串 对值是基本类型时有用
jsonNode.asText()
#get() type是array时可以用来取第N个
node.get(0)
#得到node下的str节点
node.get(“str”)
#得到node下的str的节点，与get不同的是str不存在时返回的不是null而是missing node不妨碍后面的as...
node.path("str")
#findPath可以直接找到json中的某个节点
jsonNode.findPath("str")
```

第三种：
jsonString和对应类可直接转换。
```java
import com.facebook.presto.jdbc.internal.guava.base.Preconditions;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import javax.validation.constraints.NotNull;
import java.io.IOException;

@Slf4j
public final class JsonUtils {

 private static final ObjectMapper mapper = new ObjectMapper();

 private JsonUtils() {
 }

 static {
 	//设置忽略未声明的字段
 	mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
 }

 /**
 * 把json数据转成类
 *
 * @param json 需要转成对象的字符串
 * @param object 对象
 * @param <T> 泛型
 * @return 转成的对象
 */

 public static <T> T toObject(String json, Class<T> object) throws IOException {
 	Preconditions.checkArgument(StringUtils.isNotBlank(json), "toObject方法的参数异常", json);

 	try {
 		return mapper.readValue(json, object);
 	} catch (IOException e) {
 		log.error("{} json转成object时失败！", json);
 		throw e;
 	}
 }
}
```
**_json中出现很多\时，是因为多层序列化，有对象被多次序列化。一个对象A序列化后作为string给另一个对象B，B再序列化，之后A就会出现\，如果再多一层，引号前面会有三个\\\。
json反序列化时需要转化成对象或者map。_**