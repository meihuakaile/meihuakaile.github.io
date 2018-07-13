---
title: 'maven常用'
date: "2018/04/20"
tags: [maven]
categories: [开发常用工具]
copyright: true
---
mvn --version == mvn -v
mvn archetype:generate 自动生成mvn项目

**_使用archetype插件生成一个webapp项目：_**
mvn archetype:generate -DgroupId=com.qunar.fresh -DartifactId=mytestapp -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false

`mvn clean` **_清除项目_**
`mvn compile` **_编译代码_**
`mvn test-compile` **_只编译测试代码_**
`mvn test` **_运行单元测试代码_**（编译源代码、测试代码；运行测试代码）
`mvn package` 打包成jar/war(根据pom文件packaging定)文件，在target文件下
`mvn install` **_部署到本地仓库_**
`mvn deploy` 在远程仓库中安装jar（一般不用）

`mvn clean install -Dmaven.test.skip=true` **_单元测试逻辑不编译，不执行，直接跳过_**
`mvn clean install -DskipTests` **_单元测试编译，只跳过测试过程_**

`mvn help:effective-pom` 查看依赖的父pom
`mvn dependency:tree` **_查看pom依赖树，解决pom依赖问题_**

`mvn dependency:analyze`  **_分析pom依赖_**
`mvn clean package -Pdev -Dmaven.test.skip=true` **_过滤掉test指定环境打包_**

`mvn clean tomcat7:run -Pdev` **_使用tomcat指定环境启动项目_**

**scrop 规定jar包的作用域**
1.`compile`（默认） 在编译、测试、运行时均需要使用此依赖，表示编译打包之后这个jar包会放在编译后的文件里；
2.`provided` 编译时有用，但是打包后不存在。对于编译和测试有效，但在运行时无效。常见的是servlet，servlet的实现在容器里（tomcat/jetty），这个在运行时会由你的 tomcat 、jboss 、jetty等容器来提供，但在编译和测试阶段你仍是需要这个 jar 包的；
3.`runtime` 运行时使用，写代码时不能使用。常见为日志、mysql driver
4.`test` 正常不能用，只用在用test时才能用，如junit。
5.`import` 内部组件，公共pom，pom中定义了很多jar版本号

**解决依赖冲突**
mvn自带解决依赖冲突：1、依赖最近者优先。2、路径相同，先声明的优先。
但是最好能自己指定：
1、`dependencyManegement` 解决包依赖冲突（首选办法），指定优选包版本
2、`exclusion` 解决包依赖冲突（加在不用的那个dependency里）。有一些公司规定不让使用某些包，如日志输出上不能用commons-logging

生命周期
clean Pre-clean（准备） Clean（移除） Post-clean 会以此执行上面的命令
default 构建的所有步骤 23个阶段 process-resources 资源文件的准备
site 站点（）Pre-site site Post-site site-deploy 项目的描述和使用指南

groupid 项目的唯一标示
ati。。。 项目组唯一id
version snapshot还在开发，快照；Release 稳定版本
**_finalname build中，，默认ROOT（最终的打包名字，，，）_**
**_profile 选择环境_**

**本地仓库** 默认就在.m2/repository/ 设置在setting.xml ！！！！
中央仓库 mvn/lib/maven-model-builder-3.jar,org/apache/macen/model/pom-4.0.0.xml

type 依赖类型（默认jar包）
scope 依赖范围
optional 依赖是否可选
exclusions 依赖冲突解决

<modules>
<packaging>pom</ 使用一个命令进行多个模块的聚合

groupid 部门名称 com.fresh.项目组名称
arti... 项目名
mvn package -Plocal profile用local
mvn clean package -Pdev
mvn clean package 工程打包
mvn enforcer:enforcer 检查重复类
mvn dependency:tree -Dverbose 具体看重复类的位置

https://stackoverflow.com/questions/24827194/maven-enforcer-plugin-missing-or-invalid-rules
