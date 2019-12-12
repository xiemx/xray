---
layout: post
title: NTP服务器搭建
published: true
author: xiemx
comments: true
date: 2015-11-29 08:11:58
tags: [ linux, ntp ]
categories:
    - linux
# permalink: '/2015/11/29/ntp%e6%9c%8d%e5%8a%a1%e5%99%a8%e6%90%ad%e5%bb%ba'
---
NTP即网络时间协议，用来同步计算机时间，提高时间的精确度。

环境：`RHEL6.5`

安装包：`ntp-4.2.6p5-1.el6.x86_64`

安装方式：
```
yum install ntp
```
ntp主配置文件中并无太多配置，有效的配置默认如下
```shell
[root@localhost html]# egrep  -v "^$|^#" /etc/ntp.conf
driftfile /var/lib/ntp/drift
restrict default kod nomodify notrap nopeer noquery——拒绝ipv4查询
restrict -6 default kod nomodify notrap nopeer noquery——拒绝ipv6查询
restrict 127.0.0.1——允许本机查询
restrict -6 ::1——允许本机查询
server 0.rhel.pool.ntp.org iburst——server指定我们去哪里同步
server 1.rhel.pool.ntp.org iburst
server 2.rhel.pool.ntp.org iburst
server 3.rhel.pool.ntp.org iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
```
我们作为服务器端需要授权其他用户过来同步时间，需要在配置行中添加
```
restrict 192.168.10.10 mask 255.255.255.0 nomodify ——允许192.168.10.10/24这台机器过来同步
restrict 192.168.10.0 mask 255.255.255.0 nomodify ——允许192.168.10.0/24这个网段的机器过来同步
server 210.72.145.44 ——我们去这台机器上同步时间
```
以上设置设置好后我们即可通过ntpdate去同步时间
```
ntpdate 210.72.145.44
```
客户端也可以通过ntpdate向我们请求数据，ntp服务器搭建好后客户端可以通过计划任务固定时间来向我们请求时间。ntp配置简单，但要注意防火墙的限制。

#### 常用ntp服务器：
```
210.72.145.44 (国家授时中心服务器IP地址)
ntp.sjtu.edu.cn 202.120.2.101 (上海交通大学网络中心NTP服务器地址）
s1b.time.edu.cn 清华大学
s1c.time.edu.cn 北京大学
s1d.time.edu.cn 东南大学
s1e.time.edu.cn 清华大学
s2d.time.edu.cn 西南地区网络中心
s2e.time.edu.cn 西北地区网络中心
s2f.time.edu.cn 东北地区网络中心
s2g.time.edu.cn 华东南地区网络中心
```
 