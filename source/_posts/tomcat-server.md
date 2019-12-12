---
layout: post
title: Tomcat搭建web服务器
published: true
author: xiemx
comments: true
date: 2016-01-24 10:01:58
tags: [ tomcat, webserver, http]
categories:
    - tomcat
# permalink: '/2016/01/24/tomcat%e6%90%ad%e5%bb%baweb%e6%9c%8d%e5%8a%a1%e5%99%a8'
---
tomcat是一款处理jsp页面的web套件，可以处理html页面和jsp页面。由于java语言的跨平台性，软件只要解压后即可运行，但运行java需要在系统中安装jdk的java虚拟机。tomcat的软件包可以去tomcat官网http://tomcat.apache.org获得，jdk的软件包需要去oracle官网下载`http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html`。

另外在部署tomcat虚拟机时我们一般会创建tomcat用户和组并将tomcat程序放在家目录下，家目录的权限为700，该目录只有tomcat用户有完整权限，其他用户无任何权限，可增加安全性。另一种部署软件的方法同我们平时安装源码软件相同，将软件放在/usr/lib/local目录下。这里就将软件部署在home/tomcat/目录下。

### tomcat目录结构

```shell
  [root@serverc tomcat]# tar xf apache-tomcat-8.0.24.tar.gz  -C /home/tomcat/
  [root@serverc tomcat]# cd /home/tomcat/apache-tomcat-8.0.24/
  [root@serverc apache-tomcat-8.0.24]# ls -l
  drwxr-xr-x 2 root root  4096 Dec 11 14:22 bin  #存放命令
  drwxr-xr-x 2 root root  4096 Jul  2 04:23 conf  #存放配置文件
  drwxr-xr-x 2 root root  4096 Dec 11 14:22 lib   #存放库文件，java的库文件后缀为jar
  -rw-r--r-- 1 root root 57011 Jul  2 04:23 LICENSE
  drwxr-xr-x 2 root root     6 Jul  2 04:20 logs   #存放相关日志
  -rw-r--r-- 1 root root  1444 Jul  2 04:23 NOTICE
  -rw-r--r-- 1 root root  6741 Jul  2 04:23 RELEASE-NOTES
  -rw-r--r-- 1 root root 16204 Jul  2 04:23 RUNNING.txt
  drwxr-xr-x 2 root root    29 Dec 11 14:22 temp   #存放临时文件
  drwxr-xr-x 7 root root    76 Jul  2 04:21 webapps  #默认网站网页文件根目录
  drwxr-xr-x 2 root root     6 Jul  2 04:20 work   #工作目录
```

### 启动tomcat
```shell
bin目录下有控制tomcat服务的启动关闭脚本，在启动jdk时我们声明下jdk的安装位置:
export  JAVA\_HOME="/usr/java/jdk1.7.0\_79/" 
声明tomcat程序中命令和库文件所在位置
export CATALINA\_HOME="/home/tomcat/apache-tomcat-8.0.24/"
声明tomcat程序中配置文件、网站根目录等所在位置
export CATALINA\_BASE="/home/tomcat/apache-tomcat-8.0.24/"
启动tomcat服务的脚本名称为startup.sh。执行该脚本可启动tomcat服务。该服务默认监听tcp/8080端口。关闭脚本为shutdown.sh。

  [root@serverc ~]# cd /home/tomcat/apache-tomcat-8.0.24/bin/
  [root@serverc bin]# ./startup.sh
  [root@serverc bin]# ps -ef | grep java
  [root@serverc bin]# netstat -ltunp | grep 8080
  tcp6       0      0 :::8080                 :::*                    LISTEN      1102/java

```
测试tomcat服务器

在浏览器中输入地址访问8080，http://192.168.17.10:8080 来测试tomcat服务器是否正常

![tomcat](/images/tomcat-300x84.png)