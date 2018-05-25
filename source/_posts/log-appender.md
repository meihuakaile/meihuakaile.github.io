---
title: 'log-appender'
date: "2017/05/25"
tags: ["appender"]
categories: [java]
copyright: true
---
`<appender>`是`<configuration>`的子节点。有name、class属性。

注意点：appender下的file节点和rollingPolicy下fileNamePattern的区别。前者是活动文件，即当前日志存储的文件；后者是归档文件，即每天压缩的文件之后起的名字（假设policy是每天都把日志压缩一下的策略）。

### ConsoleAppender
把日志输出到控制台。子节点：
（1）`<encoder>`对日志格式化。
（2）`<target>`字符串System.out或System.err,默认是System.out。（不太懂，没用过）

例：
```xml
 <!-- 控制台输出日志 -->
 <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
 	<layout class="ch.qos.logback.classic.PatternLayout">
 		<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
 	</layout>
 </appender>
```
### FileAppender
把日志输出到文件，子节点有：
（1）`<file>` 日志被写入的文件位置，可以相对也可以绝对。上层目录不存在会自动创建，没有默认值。
（2） ~~`<rollingPolicy>`~~
（3）`<encoder>` 将记录格式化。
（4）`<append>` true，追加到现有文件中；false，清空重新写到文件中。 默认true。
（5）`<prudent>` true，安全的写入到文件，即使其他的FileAppender也在写这个文件，效率低。 默认是false。

### RollingFileAppender
滚动记录日志。满足某一条件时，文件会被移动别的文件。子节点：
（1）`<file>` 日志被写入的文件位置，可以相对也可以绝对。上层目录不存在会自动创建，没有默认值。
（2）`<rollingPolicy>` 发生滚动时，RollingFileAppender的行为，重命名或者移动位置。
（3）`<encoder>` 将记录格式化。
（4）`<append>` true，追加到现有文件中；false，清空重新写到文件中。 默认true。
（5）`<prudent>` true，不支持FixedWindowRollingPolicy；支持TimeBasedRollingPolicy，但是有两个约束，其一，不支持不允许文件压缩，其二，不能设置file属性。（建议不要设为true）
（6）`<triggeringPolicy>` 告知 RollingFileAppender 合适激活滚动。（？？？？？？）

### 常用rollingPolicy
#### TimeBasedRollingPolicy
最常用的策略。用时间指定滚动策略。有以下节点：
（1）`<fileNamePattern>`  必要节点。可以是文件和%d的组合。%d可以包含时间格式，可以直接用%d。和RollingFileAppender下的file节点不同的是，此处是文件归档的名字，即当是当前策略时即是每天日志文件压缩时的名字；而file下的是日志实时记录的位置。
（2）`<maxHistory>` 可选节点，保留文档的最大数量，超出数量就删除。 注意，删除旧文档时，为了建立文档而创建的目录也会被删除。如果设置是每天一滚动，maxHistory设置是3，那样就只会保存最近3填的日志，其他的删除。

#### FixedWindowRollingPolicy
根据固定窗口算法重命名文件的滚动策略（？？？？？）。有以下子节点：
（1）`<minIndex>` 窗口索引最小值
（2）`<maxIndex>` 窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
（3）`<fileNamePattern >`  必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log.%i.log.zip

### triggeringPolicy
#### SizeBasedTriggeringPolicy
看当前活动文件的大小，如果超过指定大小会告知 RollingFileAppender 触发当前活动文件滚动。只有一个节点：
（1）`<maxFileSize>`:这是活动文件的大小，默认值是10MB.

### encoder
负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。
有一个`<pattern>`节点，用来设置日志的输入格式。使用“%”加“转换符”方式，如果要输出“%”，则必须用“\”对“\%”进行转义。
常用转换符，可参看 http://aub.iteye.com/blog/1103685  
```xml
<encoder>
    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
</encoder>
```
### 例子
保存最近30天日志
```xml
<!--滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件-->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${catalina.base}/logs/data-atpqmq.log</file>
    <!--当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${catalina.base}/logs/data-atpqmq.%d{yyyy-MM-dd}.log</fileNamePattern>
        <maxHistory>30</maxHistory>
    </rollingPolicy>

    <!--对记录事件进行格式化。-->
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>

    <!--如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。-->
    <append>true</append>
    <!--如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false-->
    <prudent>false</prudent>
</appender>
```
按照固定窗口模式生成日志文件，当文件大于20MB时，生成新的日志文件。窗口大小是1到3，当保存了3个归档文件后，将覆盖最早的日志。
```xml
<configuration>   
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">   
    <file>test.log</file>   
   
    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">   
      <fileNamePattern>tests.%i.log.zip</fileNamePattern>   
      <minIndex>1</minIndex>   
      <maxIndex>3</maxIndex>   
    </rollingPolicy>   
   
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">   
      <maxFileSize>20MB</maxFileSize>   
    </triggeringPolicy>   
    <encoder>   
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>   
    </encoder>   
  </appender>   
           
  <root level="DEBUG">   
    <appender-ref ref="FILE" />   
  </root>   
</configuration>  
```

参看：http://aub.iteye.com/blog/1103685