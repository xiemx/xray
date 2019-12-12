---
layout: post
title: Tomcat创建虚拟主机
published: true
author: xiemx
comments: true
date: 2016-01-30 04:01:28
tags: [ tomcat, http, webserver ]
categories:
    - tomcat
# permalink: '/2016/01/30/tomcat-virtual-host'
---
1.修改tomcat服务的主配置文件。该文件位于tomcat主程序下conf目录中，文件名称为server.xml

```shell
  [root@serverc ~]# cd /home/tomcat/apache-tomcat-8.0.24/conf/ 
  [root@serverc conf]# ls -l server.xml 
  -rw------- 1 tomcat root 6458 Jul  2 04:23 server.xml
```

2.编辑该文件，添加虚拟主机。host字段为tomcat服务的虚拟主机配置字段。配置两个虚拟主机，`www.tomcat1.com`和`www.tomcat2.com`，两台虚拟主机有自己的`appbase`（即网页文件根目录，下图中使用的是相对路径表示虚拟主机网页根目录，该路径是相对于tomcat服务主程序所在目录来说，该路径现不存在，后续再创建）和相关日志前缀。注`name`字段为虚拟主机名，`appBase`为根目录，`uppackwars`为自动解压是否开启，`autoDeploy`为自动部署是否开启

```xml
<Host name="www.tomcat1.com"  appBase="tomcat1.com"
unpackWARs="true" autoDeploy="true">
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
prefix="localhost_access_log" suffix=".txt"
pattern="%h %l %u %t "%r" %s %b" />
</Host>
<Host name="www.tomcat2.com"  appBase="tomcat2.com"
unpackWARs="true" autoDeploy="true">
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
prefix="localhost_access_log" suffix=".txt"
pattern="%h %l %u %t "%r" %s %b" />
</Host>
```

3.进入tomcat服务主程序所在目录，创建上述步骤中虚拟主机所指定的appBase，分别进入每一个虚拟主机的appBase目录下创建ROOT目录，在ROOT目录写每一个虚拟主机对应的首页文件。
```shell
  [root@serverc conf]# cd /home/tomcat/apache-tomcat-8.0.24/ 
  [root@serverc apache-tomcat-8.0.24]# mkdir tomcat1.com/ 
  [root@serverc apache-tomcat-8.0.24]# cd tomcat1.com/ 
  [root@serverc tomcat1.com]# mkdir ROOT 
  [root@serverc tomcat1.com]# echo tomcat1 > ROOT/index.html 
  [root@serverc tomcat1.com]# cd ../ 
  [root@serverc apache-tomcat-8.0.24]# mkdir tomcat2.com 
  [root@serverc apache-tomcat-8.0.24]# cd tomcat2.com 
  [root@serverc tomcat2.com]# mkdir ROOT 
  [root@serverc tomcat2.com]# echo tomcat2 > ROOT/index.html
```

4.重启tomcat服务

```shell
  [root@serverc tomcat2.com]# /etc/init.d/tomcat stop
  [root@serverc tomcat2.com]# /etc/init.d/tomcat start
```

5.在client端机器上测试

```shell
  [root@workstation ~]# echo 172.25.41.10 www.tomcat1.com >> /etc/hosts 
  [root@workstation ~]# echo 172.25.41.10 www.tomcat2.com >> /etc/hosts
  [root@workstation ~]# curl www.tomcat1.com:8080
  tomcat1
  [root@workstation ~]# curl www.tomcat2.com:8080
  tomcat2
```
 