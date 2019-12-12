---
layout: post
title: tomcat出现PermGen Space问题
published: true
author: xiemx
comments: true
date: 2016-03-29 05:03:32
tags: [ tomcat, webserver, http ]
categories:
    - tomcat
# permalink: '/2016/03/29/tomcat-permgen-space'
---
### 现象
tomcat服务器运行一段时间，总是会自动报异常：`java.lang.OutOfmemoryError:PermGen Space`的错误，导致项目无法正常运行。

出现这个错误是由于内存泄漏。PermGen Space指的是内存的永久保存区，该块内存主要是被JVM存放class和mete信息的，当class被加载loader的时候就会被存储到该内存区中，与存放类的实例的heap区不同，java中的垃圾回收器GC不会在主程序运行期对PermGen space进行清理，所以当我们的应用中有很多的class时，很可能就会出现PermGen space的错误。

### 解决方法
手动设置MaxPermSize的大小

修改 TOMCAT_HOME/bin/catalina.bat(Linux上为catalina.sh)文件，在echo "using CATALINA_BASE：$CATALINA_BASE"上面加入这一行内容：
```shell
set JAVA_OPTS=%JAVA_OPTS% -server -XX:PermSize=128m -XX:MaxPermSize=512m
```