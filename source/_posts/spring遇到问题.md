---
title: spring遇到问题
date: "2018/09/05"
tags: ['spring']
categories: ['spring问题']
copyright: true
---
# javax.management.RuntimeMBeanException: java.lang.UnsupportedOperationException: Usage threshold is not supported
使用spring，启动tomcat时报错：`javax.management.RuntimeMBeanException: java.lang.UnsupportedOperationException: Usage threshold is not supported`
```xml
javax.management.RuntimeMBeanException: java.lang.UnsupportedOperationException: Usage threshold is not supported
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.rethrow(DefaultMBeanServerInterceptor.java:839) ~[na:1.8.0_144]
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.rethrowMaybeMBeanException(DefaultMBeanServerInterceptor.java:852) ~[na:1.8.0_144]
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.getAttribute(DefaultMBeanServerInterceptor.java:651) ~[na:1.8.0_144]
        at com.sun.jmx.mbeanserver.JmxMBeanServer.getAttribute(JmxMBeanServer.java:678) ~[na:1.8.0_144]
        at qunar.servlet.server.TomcatServer.extractContext(TomcatServer.java:40) [common-core-8.3.2.jar:na]
        at qunar.servlet.server.ServerWrapper.<init>(ServerWrapper.java:26) [common-core-8.3.2.jar:na]
        at qunar.servlet.server.TomcatServer.<init>(TomcatServer.java:14) [common-core-8.3.2.jar:na]
        at qunar.ServletWatcher.portOf(ServletWatcher.java:240) [common-core-8.3.2.jar:na]
        at qunar.ServletWatcher.fixPort(ServletWatcher.java:232) [common-core-8.3.2.jar:na]
        at qunar.ServletWatcher.init(ServletWatcher.java:91) [common-core-8.3.2.jar:na]
        at qunar.ServletWatcher.contextInitialized(ServletWatcher.java:58) [common-core-8.3.2.jar:na]
        at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:5016) [tomcat-embed-core-7.0.59.jar:7.0.59]
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5524) [tomcat-embed-core-7.0.59.jar:7.0.59]
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150) [tomcat-embed-core-7.0.59.jar:7.0.59]
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1575) [tomcat-embed-core-7.0.59.jar:7.0.59]
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1565) [tomcat-embed-core-7.0.59.jar:7.0.59]
        at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_144]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_144]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_144]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_144]
Caused by: java.lang.UnsupportedOperationException: Usage threshold is not supported
        at sun.management.MemoryPoolImpl.getUsageThreshold(MemoryPoolImpl.java:106) ~[na:1.8.0_144]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_144]
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_144]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_144]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_144]
        at sun.reflect.misc.Trampoline.invoke(MethodUtil.java:71) ~[na:1.8.0_144]
        at sun.reflect.GeneratedMethodAccessor40.invoke(Unknown Source) ~[na:na]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_144]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_144]
        at sun.reflect.misc.MethodUtil.invoke(MethodUtil.java:275) ~[na:1.8.0_144]
        at com.sun.jmx.mbeanserver.ConvertingMethod.invokeWithOpenReturn(ConvertingMethod.java:193) ~[na:1.8.0_144]
        at com.sun.jmx.mbeanserver.ConvertingMethod.invokeWithOpenReturn(ConvertingMethod.java:175) ~[na:1.8.0_144]
        at com.sun.jmx.mbeanserver.MXBeanIntrospector.invokeM2(MXBeanIntrospector.java:117) ~[na:1.8.0_144]
        at com.sun.jmx.mbeanserver.MXBeanIntrospector.invokeM2(MXBeanIntrospector.java:54) ~[na:1.8.0_144]
        at com.sun.jmx.mbeanserver.MBeanIntrospector.invokeM(MBeanIntrospector.java:237) ~[na:1.8.0_144]
        at com.sun.jmx.mbeanserver.PerInterface.getAttribute(PerInterface.java:83) ~[na:1.8.0_144]
        at com.sun.jmx.mbeanserver.MBeanSupport.getAttribute(MBeanSupport.java:206) ~[na:1.8.0_144]
        at javax.management.StandardMBean.getAttribute(StandardMBean.java:372) ~[na:1.8.0_144]
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.getAttribute(DefaultMBeanServerInterceptor.java:647) ~[na:1.8.0_144]
        ... 17 common frames omitted
```
原因：jmxmp的端口被占了

参考：https://stackoverflow.com/questions/12283207/unable-to-connect-with-jmxmp-in-tomcat/12337393#12337393?s=d4f50b7dc0164ed298b008d97968de5b

# Error instantiating class com.qunar.market.model. with invalid types () or values (). Cause: java.lang.NoSuchMethodException: com.qunar.market.model.MaterialDeliverModel
原因：构造函数被重载过，但是没有空的构造函数。
参考：https://blog.csdn.net/qq_25821067/article/details/54811165