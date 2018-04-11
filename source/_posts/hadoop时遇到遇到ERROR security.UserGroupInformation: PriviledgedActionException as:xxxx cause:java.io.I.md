---
title: "hadoop时遇到遇到ERROR security.UserGroupInformation: PriviledgedActionException as:xxxx cause:java.io.I"
date: "2018/04/08"
tags: ['hadoop']
categories: ['hadoop']
copyright: true
---
hadoop 1.X版本在windows下的eclipse里操纵ubuntu远程跑hadoop项目遇到问题：
```
16:52:22 WARN util.NativeCodeLoader: Unable to load native-hadoop library for
your platform... using builtin-java classes where applicable

16:52:22 ERROR security.UserGroupInformation: PriviledgedActionException
as:chenli cause:java.io.IOException: Failed to set permissions of path: \tmp
\hadoop-chenli\mapred\staging\chenli1971412180\\.staging to 0700  
Exception in thread "main" java.io.IOException: Failed to set permissions of
path: \tmp\hadoop-chenli\mapred\staging\chenli1971412180\\.staging to 0700  
at org.apache.hadoop.fs.FileUtil.checkReturnValue(FileUtil.java:691)  
at org.apache.hadoop.fs.FileUtil.setPermission(FileUtil.java:664)  
at org.apache.hadoop.fs.RawLocalFileSystem.setPermission(RawLocalFileSystem.ja
va:514)  
at org.apache.hadoop.fs.RawLocalFileSystem.mkdirs(RawLocalFileSystem.java:349)  
at org.apache.hadoop.fs.FilterFileSystem.mkdirs(FilterFileSystem.java:193)  
at org.apache.hadoop.mapreduce.JobSubmissionFiles.getStagingDir(JobSubmissionF
iles.java:126)  
at org.apache.hadoop.mapred.JobClient$2.run(JobClient.java:942)  
at org.apache.hadoop.mapred.JobClient$2.run(JobClient.java:936)  
at java.security.AccessController.doPrivileged(Native Method)  
at javax.security.auth.Subject.doAs(Unknown Source)  
at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.j
ava:1190)  
at org.apache.hadoop.mapred.JobClient.submitJobInternal(JobClient.java:936)  
at org.apache.hadoop.mapreduce.Job.submit(Job.java:550)  
at org.apache.hadoop.mapreduce.Job.waitForCompletion(Job.java:580)  
at firsthadoop.org.eg.WordCount.main(WordCount.java:142)  
```
  

解决：
    hadoop-core-1.0.2.jar的org.apache.hadoop.fs.FileUtil文件中找到以下部分，注释掉checkReturnValue方法中的下面代码：
```java
      private static void checkReturnValue(boolean rv, File p,
        FsPermission permission) throws IOException {
        /*
        //win7 connect to linux hadoop
        if (!rv) {
          throw new IOException("Failed to set permissions of path: " + p +" to " +  String.format("o", permission.toShort()));
        }
        */
      }
```
  

或者下载： [ http://download.csdn.net/detail/u010668907/9413683
](http://download.csdn.net/detail/u010668907/9413683) 直接替换原本的hadoop-
core-1.0.2.jar

