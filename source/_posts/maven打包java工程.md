---
title: 'maven打包java工程'
date: "2018/06/21"
tags: []
categories: [spring]
copyright: true
---
(1)maven package直接打包
(2)maven-assembly-plugin
(3)maven-shade-plugin
这是了解到的三种打包。
第一种是直接打包，没有用插件，问题是不会把项目的依赖包打包，作为执行包可能会出错。
后面两个是插件。网上说(2)有bug，多个依赖包可能会依赖不同版本的同一个包，这时会把这个包的某一个版本打包，然后就会出错。
插件(3)会把这个包的所有版本都打包。（2）的goal有single、help；（3）的goal的goal有shade、help
```xml
<plugins>
        <!--tomcat插件，使用mvn:tomcat7 clean run-->
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        <configuration>
          <port>8080</port>
          <path>/</path>
          <uriEncoding>UTF-8</uriEncoding>
        </configuration>
      </plugin>
      
      <!--编译插件 maven 2 默认使用jdk1.3来编译，maven 3默认使用jdk1.5来编译-->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <!-- since 2.0 -->
        <version>3.7.0</version>
        <configuration>
            <!-- use the Java 8 language features -->
            <source>1.8</source>
            <!-- want the compiled classes to be compatible with JVM 1.8 -->
            <target>1.8</target>
            <!-- The -encoding argument for the Java compiler. -->
            <encoding>UTF8</encoding>
        </configuration>
      </plugin>
      
      <!--maven-shade-plugin插件-->
      <plugin>  
      <!--都是org.apache.maven.plugins， 可以省略-->
        <groupId>org.apache.maven.plugins</groupId>  
        <artifactId>maven-shade-plugin</artifactId>  
        <version>3.1.1</version>
        <executions>
            <execution>
               <!--将shade插件的shade:shade加入到Maven的package阶段-->
               <!--可以通过命令mvn clean package 也执行shade:shade阶段-->
               <phase>package</phase>
               <goals>
                    <goal>shade</goal>
               </goals>
               <!--直接在代码里指定主函数，使用run jar时，不再需要指定主函数-->
               <!--例如，hadoop的run jar执行时只需要 hadoop jar jar包 参数-->
               <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>
                                        com.mr.wordcount.WordCountJob
                                    </mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
            </execution>
        </executions>  
      </plugin>
      
      <!--maven-assembly-plugin插件-->
      <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <executions>
            <execution>
                <phase>package</phase>
                <goals>
                    <goal>single</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <descriptorRefs>
                <descriptorRef>jar-with-dependencies</descriptorRef>
            </descriptorRefs>
            <archive>
                <index>true</index>
                <manifest>
                    <mainClass>com.mr.wordcount.WordCountJob</mainClass>
                </manifest>
            </archive>
        </configuration>
      </plugin>
      
      <plugin> 
         <groupId>org.apache.maven.plugins</groupId> 
         <artifactId>maven-resources-plugin</artifactId> 
         <configuration> 
            <encoding>UTF-8</encoding> 
            <!-- 过滤后缀为pem、pfx、p12的证书文件 --> 
            <nonFilteredFileExtensions> 
               <nonFilteredFileExtension>pem</nonFilteredFileExtension> 
               <nonFilteredFileExtension>pfx</nonFilteredFileExtension> 
               <nonFilteredFileExtension>p12</nonFilteredFileExtension>
            </nonFilteredFileExtensions> 
         </configuration> 
      </plugin>
</plugins>
```
maven-compiler-plugin用来编译Java代码，maven-resources-plugin则用来处理资源文件。
默认的主资源文件目录是src/main/resources，很多用户会需要添加额外的资源文件目录，这个时候就可以通过配置maven-resources-plugin来实现。此外，资源文件过滤也是Maven的一大特性，你可以在资源文件中使用${propertyName}形式的Maven属性，然后配置maven-resources-plugin开启对资源文件的过滤，之后就可以针对不同环境通过命令行或者Profile传入属性的值，以实现更为灵活的构建。


总结自：http://chenzhou123520.iteye.com/blog/1706242
链接：https://www.jianshu.com/p/ce01bb1615a8

在项目下pom.xml的project节点下创建了开发环境和线上环境的profile
```xml
<profiles>
        <profile>
            <id>dev</id>
            <properties>
                <env>dev</env>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>prd</id>
            <properties>
                <env>prd</env>
            </properties>
        </profile>
    </profiles>
```
其中id代表这个环境的唯一标识.
properties下我们我们自己自定义了标签env，内容分别是dev和prd。
activeByDefault=true代表如果不指定某个固定id的profile，那么就使用这个环境.
参考：https://www.cnblogs.com/nfcm/p/7550772.html