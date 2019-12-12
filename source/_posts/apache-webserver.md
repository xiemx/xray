---
layout: post
title: Apache搭建web服务器
published: true
author: xiemx
comments: true
date: 2016-01-17 10:01:16
tags: [ apache, webserver ]
categories:
    - apache
---
linux下搭建web服务器的套件很多，基本使用的还是apache、nginx、tomcat。这三种服务器都有各自的优缺点，在不同的场景下应根据实际需求选用。linux下apache搭建web服务器配置起来较为简单，只需在系统中安装httpd软件包即可。但要实现复杂环境下的应用，则需要配置更多的功能。

1. 安装软件包
```shell
yum install httpd -y

[root@localhost ~]# rpm -ql httpd | grep 'httpd\.conf'
/etc/httpd/conf/httpd.conf    apache服务的主配置文件

[root@localhost ~]# grep -v '^#\|^$\|#' /etc/httpd/conf/httpd.conf
Listen 80     监听端口
Include conf.d/*.conf    装载附加配置文件
User apache
Group apache  运行用户和组
DocumentRoot "/var/www/html"    默认主目录
DirectoryIndex index.html index.html.var  默认首页
```

2. 在主目录/var/www/html下创建一个测试页

```shell
[root@localhost ~]# echo "test page" > /var/www/html/index.html
[root@localhost ~]# /etc/init.d/httpd start
Starting httpd:                        [OK]
```
3. 使用浏览器测试访问

[![apachetest](/images/apachetest-300x89.png)](/images/apachetest-300x89.png)