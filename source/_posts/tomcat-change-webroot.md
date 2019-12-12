---
layout: post
title: Tomcat修改网站根目录
published: true
author: xiemx
comments: true
date: 2016-08-18 04:08:40
tags: [ tomcat, webserver, http ]
categories:
    - tomcat
# permalink: '/2016/08/18/tomcat-change-webroot'
---
1.tomcat原来的默认根目录是`http://localhost:8080`，如果想修改访问的根目录，可以这样：

找到tomcat的server.xml（在conf目录下），找到:
```xml
<Host name="localhost" appBase="webapps"
       unpackWARs="true" autoDeploy="true"
       xmlValidation="false" xmlNamespaceAware="false"></Host>
```
在</Host>前插入:
```xml
<Context path="" docBase="/home/tomcat/" debug="0"/>
```
其中/home/tomcat/就是我想设置的网站根目录，然后重启tomcat。再次访问`http://localhost:8080`时，就是直接访问`/home/tomcat/`目录下的文件了。

2.tomcat的web.xml（在conf目录下），在该文件中找到
```xml
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
```
这是tomcat默认的3个文件，当你输入指定路径后，tomcat会自动查找这3个页面。如果你想让tomcat自动找到自己的页面，比如main.jsp。可以修改上面信息为：
```xml
    <welcome-file-list>
        <welcome-file>main.jsp</welcome-file>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
```