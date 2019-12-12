---
layout: post
title: Tomcat多实例运行
published: true
author: xiemx
comments: true
date: 2016-01-30 05:01:29
tags: [ tomcat, webserver, http ]
categories:
    - tomcat
# permalink: '/2016/01/30/multi-tomcat-server'
---
一个程序默认在系统中都是维护一个进程（或一个主进程多个子进程），这样的结构在进程出错终止时会导致该进程下所有的网站都会打不开。但tomcat我们可以通过调整启动时指定的tomcat程序位置来启动多个tomcat程序，分属不同的进程这样在某个进程终止时也不会影响到其他进程，不会导致其它网站无法访问。但一个端口只能被一个进程监听，当我们启动多进程就不能同时监听8080端口，我们可以依次监听8081，8082等，如果要实现直接输入域名访问可以在前段增加nginx来做代理即可。

```shell
[root@serverc ~]# /etc/init.d/tomcat stop
[root@serverc ~]# cd /home/tomcat/
[root@serverc tomcat]# mkdir tomcat1 tomcat2
[root@serverc tomcat]# cd apache-tomcat-8.0.24/
[root@serverc apache-tomcat-8.0.24]# cp -rp logs/ temp/ tomcat1.com/ work/ webapps/ conf/ ../tomcat1/
[root@serverc apache-tomcat-8.0.24]# cp -rp logs/ temp/ tomcat2.com/ work/ webapps/ conf/ ../tomcat2/
[root@serverc apache-tomcat-8.0.24]# rm -rf LICENSE NOTICE RELEASE-NOTES RUNNING.txt conf logs tomcat1.com tomcat2.com webapps
[root@serverc apache-tomcat-8.0.24]# ls
bin lib temp work
[root@serverc apache-tomcat-8.0.24]# cd ../tomcat1/
[root@serverc tomcat1]# vim conf/server.xml
<Host name="www.tomcat1.com" appBase="tomcat1.com"
unpackWARs="true" autoDeploy="true">
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
prefix="localhost_access_log" suffix=".txt"
pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>

[root@serverc tomcat1]# cd ../tomcat2/
[root@serverc tomcat2]# vim conf/server.xml
<Host name="www.tomcat2.com" appBase="tomcat2.com"
unpackWARs="true" autoDeploy="true">
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
prefix="localhost_access_log" suffix=".txt"
pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
=============================================================================
<Connector port="8081" protocol="HTTP/1.1"
connectionTimeout="20000"
redirectPort="8443" />
=============================================================================
<Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />

[root@serverc tomcat2]# cd /etc/init.d/
[root@serverc init.d]# mv tomcat tomcat1
[root@serverc init.d]# vim tomcat1
export CATALINA_HOME="/home/tomcat/apache-tomcat-8.0.24/"
export CATALINA_BASE="/home/tomcat/tomcat1"

[root@serverc init.d]# cp tomcat1 tomcat2
[root@serverc init.d]# vim tomcat2
export CATALINA_HOME="/home/tomcat/apache-tomcat-8.0.24/"
export CATALINA_BASE="/home/tomcat/tomcat2"

[root@serverc init.d]# /etc/init.d/tomcat1 start
[root@serverc init.d]# /etc/init.d/tomcat2 start
[root@serverc init.d]# netstat -ltunp | grep 8080
tcp6 0 0 :::8080 :::* LISTEN 5749/jsvc.exec
[root@serverc init.d]# netstat -ltunp | grep 8081
tcp6 0 0 :::8081 :::* LISTEN 5782/jsvc.exec
```