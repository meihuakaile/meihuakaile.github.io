---
title: 'azkaban'
date: "2018/10/19"
tags: [hadoop]
categories: ['hadoop']
---
## 出错
1、`JCE报错`
```bash
testV1_1 FAILED
    java.lang.RuntimeException: java.lang.RuntimeException: org.jasypt.exceptions.EncryptionOperationNotPossibleException: Encryption raised an exception. A possible cause is you are using strong encryption algorithms and you have not installed the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files in this Java Virtual Machine
        at azkaban.crypto.Crypto.decrypt(Crypto.java:76)
        at azkaban.crypto.DecryptionTest.testV1_1(DecryptionTest.java:35)

        Caused by:
        java.lang.RuntimeException: org.jasypt.exceptions.EncryptionOperationNotPossibleException: Encryption raised an exception. A possible cause is you are using strong encryption algorithms and you have not installed the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files in this Java Virtual Machine
            at azkaban.crypto.CryptoV1_1.decrypt(CryptoV1_1.java:57)
            at azkaban.crypto.Crypto.decrypt(Crypto.java:74)
            ... 1 more

            Caused by:
            org.jasypt.exceptions.EncryptionOperationNotPossibleException: Encryption raised an exception. A possible cause is you are using strong encryption algorithms and you have not installed the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files in this Java Virtual Machine
                at org.jasypt.encryption.pbe.StandardPBEByteEncryptor.handleInvalidKeyException(StandardPBEByteEncryptor.java:1073)
                at org.jasypt.encryption.pbe.StandardPBEByteEncryptor.decrypt(StandardPBEByteEncryptor.java:1050)
                at org.jasypt.encryption.pbe.StandardPBEStringEncryptor.decrypt(StandardPBEStringEncryptor.java:725)
                at azkaban.crypto.CryptoV1_1.decrypt(CryptoV1_1.java:55)
                ... 2 more

azkaban.crypto.EncryptionTest > testEncryption FAILED
    org.jasypt.exceptions.EncryptionOperationNotPossibleException: Encryption raised an exception. A possible cause is you are using strong encryption algorithms and you have not installed the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files in this Java Virtual Machine
        at org.jasypt.encryption.pbe.StandardPBEByteEncryptor.handleInvalidKeyException(StandardPBEByteEncryptor.java:1073)
        at org.jasypt.encryption.pbe.StandardPBEByteEncryptor.encrypt(StandardPBEByteEncryptor.java:924)
        at org.jasypt.encryption.pbe.StandardPBEStringEncryptor.encrypt(StandardPBEStringEncryptor.java:642)
        at azkaban.crypto.CryptoV1_1.encrypt(CryptoV1_1.java:42)
        at azkaban.crypto.Crypto.encrypt(Crypto.java:58)
        at azkaban.crypto.EncryptionTest.testEncryption(EncryptionTest.java:28)
        
5 tests completed, 2 failed  
> Task :azkaban-common:compileJava 
注: 某些输入文件使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
注: 某些输入文件使用了未经检查或不安全的操作。
注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。


FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':az-crypto:test'.
> There were failing tests. See the report at: file:///opt/azkaban/az-crypto/build/reports/tests/test/index.html

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org      
```
解决：
(下载JCE,使用的JDK8，包含了JCE所需要的jre8)[https://www.oracle.com/technetwork/cn/java/javase/downloads/jce8-download-2133166-zhs.html]
解压后放置到jdk所在目录下（使用`which java`找java目录）的：
`cp UnlimitedJCEPolicyJDK8/* /usr/local/jdk1.8.0_74/jre/lib/security`

参考：https://juejin.im/entry/5bbd92e9e51d4539701e85ab

2、`keytool 错误: java.io.FileNotFoundException: keystore (权限不够)`
执行`keytool -keystore keystore -alias jetty -genkey -keyalg RSA`出错。
解决：`-keystore keystore`这个参数的原因，当前目录没有写权限。只需把换成有权限的目录，再把文件拷到当前目录即可。

参考：http://www.cnblogs.com/mengshu-lbq/archive/2012/05/16/2503564.html

3、`azkaban.executor.ExecutorManagerException: No active executors found`
解决：
数据库 `INSERT INTO executors(host,port, active) VALUES("executor_ip", 12321, true)`
配置文件
```bash
executor.port=12321
azkaban.use.multiple.executors=true
azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus
azkaban.executorselector.comparator.NumberOfAssignedFlowComparator=1
azkaban.executorselector.comparator.Memory=1
azkaban.executorselector.comparator.LastDispatched=1
azkaban.executorselector.comparator.CpuUsage=1
```

参考：https://zzyongx.github.io/blogs/azkaban.html

## 安装
配置文件 
注：先配置好服务器节点上的时区 
1、先生成时区配置文件Asia/Shanghai，用交互式命令 tzselect 即可 
2、拷贝该时区文件，覆盖系统本地时区配置 
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

### 安装
`git clone https://github.com/azkaban/azkaban.git`
### 编译
进入到Azkaban的根目录下面，执行`./gradlew build`，进行编译，如出错请参考上面
编译好的文件都放在build/distributions/目录下
执行`cp –r azkaban-*/build/distributions/*.tar.gz /opt/` 拷贝编译好的tar.gz包

参考：https://blog.csdn.net/ProfoundOx/article/details/78888183（比较靠谱的安装）
https://blog.csdn.net/weixin_35852328/article/details/79327996

## redash安装
http://172.18.0.1:5000/
参考：https://blog.csdn.net/diantun00/article/details/80968604