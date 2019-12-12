---
layout: post
title: DNS服务器搭建
published: true
author: xiemx
comments: true
date: 2015-11-30 10:11:12
tags: [ dns ]
categories:
    - dns
---
DNS即域名系统，用于将IP地址和域名之间的翻译。linux中搭建dns服务器一般使用BIND软件来实现。我们在网络中常常会有将域名转化成IP，和将IP转化成域名这两种需求，这两种需求对应了dns解析中的正向和反向解析。

* 正向解析：将域名解析成IP地址

* 反向解析：将ip地址解析成域名


`完全合格域名（FQDN）`：点结尾的域名，例如"www.xiemx.com." 就是一个完全合格域名。在一般的网络应用中，我们可以省略完全合格域名最右侧的点，但DNS对这个点不能随便省略。因为这个点代表了DNS的根，有了这个点，完全合格域名就可以表达为一个绝对路径，例如"www.xiemx.com."就可以表示为DNS根下的com子域下xiemx.com域中一个名为www的主机。如果DNS发现一个域名不是以点结尾的完全合格域名，就会把这个域名加上当前的区域名称作为后缀，让其满足完全合格域名的形式需求。

查询方法

1. 递归查询:
一般客户机和服务器之间属递归查询，即当客户机向DNS服务器发出请求后,若DNS服务器本身不能解析,则会向另外的DNS服务器发出查询请求，得到结果后转交给客户机；
2. 迭代查询(反复查询):
一般DNS服务器之间属迭代查询，如：若DNS2不能响应DNS1的请求，则它会将DNS3的IP给DNS1，以便其再向DNS3发出请求；DNS3不能响应DNS1的请求，则它会将DNS4的IP给DNS1。如此反复。

记录类型

* A记录：域名到IP地址的映射。

* CNAME记录：也叫别名记录，用于定义A记录的别名

* MX记录：邮件交换记录（MX）邮件服务器发送邮件时定位邮件服务器，可以有多个MX记录，但存在优先级差别

* NS记录：用于标识区域的DNS服务器，即是说负责此DNS区域的权威名称服务器，用哪一台DNS服务器来解析该区域。

* PTR记录：是IP地址到DNS名称的映射，用于反向解析

 

安装软件：
```shell
yum  -y  install  bind

我们使用的是软件版本是9.8.2，不同的版本配置的语法会有区别。

[root@localhost ~]# yum info bind
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
localyumdata | 3.9 kB 00:00 ...
Installed Packages
Name : bind
Arch : x86_64
Epoch : 32
Version : 9.8.2
Release : 0.17.rc1.el6_4.6
```
配置文件：
```
/var/run/named/  ——进程pid文件
/var/named/  ——数据文件目录
/etc/named.conf ——主配置文件
/etc/named.rfc1912.zones ——辅配置文件
/etc/rc.d/init.d/named ——进程控制脚本文件

[root@localhost ~]# cat /etc/named.conf
options {
  listen-on port 53 { any; };——设置监听那些ipv4地址的53端口，默认为127.0.0.1。设置any允许所有，也可以；号间隔一个个罗列IP。
  listen-on-v6 port 53 { any; };——ipv6
  directory "/var/named";——数据目录
  dump-file "/var/named/data/cache_dump.db";
  statistics-file "/var/named/data/named_stats.txt";
  memstatistics-file "/var/named/data/named_mem_stats.txt";
  allow-query { any; };——允许哪些IP来查询，默认为localhost。any为允许所有
  recursion yes;——是否允许递归

  dnssec-enable yes; ——是否启用DNSSEC支持，DNS安全扩展（DNSSEC）提供了验证DNS数据由效性的系统
  dnssec-validation yes;
  dnssec-lookaside auto;

  /* Path to ISC DLV key */
  bindkeys-file "/etc/named.iscdlv.key";

  managed-keys-directory "/var/named/dynamic";
};

logging {
  channel default_debug {
    file "data/named.run";
    severity dynamic;
  };
};

zone "." IN {
  type hint;
  file "named.ca";
};  ——定义根域服务器

include "/etc/named.rfc1912.zones"; ——装载辅配置文件，我们一般将需要解析的域名配置写在这个文件中。
include "/etc/named.root.key";
```
```shell
[root@localhost ~]# cat /etc/named.rfc1912.zones

zone "xiemx.com" IN {                    ——" "内写要解析的域名
  type master;                                      ——设置类型为master主类型，还有其它的hint根类型、slave从类型。当2台dns做主辅同步时会设置辅DNS中的域类型为slave。
  file "xiemx.com.zone";                    ——xiemx.com域的数据文件，默认在/var/named/中，缺省目录定义在主配置中directory行。
  allow-update { none; };
};                                                          —— xiemx.com域名的正向解析配置

zone "1.168.192.in-addr.arpa" IN { ——""内的IP反向写要解析的ip网段.in-addr-arpa,本条就是解析192.168.1这个网段
  type master;
  file "192.168.1.zone";
  allow-update { none; };
};                                                          ——192.168.1这个网段的反向解析配置

 以上的named.rfc1912.zones内容只是节选了部分，如我们手动添加容易出现语法错误时，可以复制文件中原有的localhost区域信息，直接修改即可。
```
```shell
[root@localhost ~]# cat /var/named/xiemx.com.zone
$TTL    1D       ——TTL条目更新时间默认1D，当时间小时解析精度高，时间太大会导致更改解析记录后生效时间太长，TTL太小的话频繁更新也会影响域名访问速度。
@          IN       SOA      @   rname.invalid. (
0    ; serial   ——时间标识，一般都是写当前的日期，由于主辅同步时确定条目是不是新的
1D    ; refresh——slave多长时间更新一次
1H    ; retry——slave同步失败时间隔多长时间再次尝试
1W    ; expire——当主辅不能同步时，1周之后认为其死亡，不再尝试同步
3H )    ; minimum——条目缓存时间，一般为错误的记录保存此时间。当一个主机请求解析一条错误的条目时，3小时后才能再次请求查询
NS              @
A                 192.168.1.10
www    A                 192.168.1.100
MX  5          mail                 ——MX记录后接5为优先级，mx记录有优先级属性需要定义
mail     A                 192.168.1.101
ftp       CNAME      ftp1                
ftp1       A                192.168.1.103  
$generate  150-200   test$        A           192.168.1.$      ——用变量定义一个连续范围的解析一一对应别名和IP 。例：test150.xiemx.com<——>192.168.1.150
```
```shell
[root@localhost ~]# cat /var/named/192.168.1.zone
$TTL    1D
@          IN       SOA      @   rname.invalid. (
0    ; serial
1D    ; refresh
1H    ; retry
1W    ; expire
3H )    ; minimum
NS              xiemx.com.
101    PTR    www.xiemx.com.
102    PTR    mail.xiemx.com.
103    PTR    ftp.xiemx.com.
103    PTR    xxx.xiemx.com.

以上的正反解只是测试时随便写的，一般正反解会相互对应。另外在写条目是要注意：

1.mx记录、cname记录的值一般都会有对应A记录
2.正反解中的@会继承配置文件中" "内的内容
3.正反解时如果写的域名不是完全合格域名，dns会自动用继承的@来补全你的域名：如正解中的www，dns会自动补全为www.xiemx.com.。
4.同一个IP可对应多个域名（PTR记录可以存在多条），一个域名只能有1条A记录（一个域名只能对应一个IP）
```
```shell
测试：

[root@localhost ~]# nslookup
> server 192.168.17.6
Default server: 192.168.17.6
Address: 192.168.17.6#53

> xiemx.com
Server:        192.168.17.6
Address:    192.168.17.6#53

Name:    xiemx.com
Address: 127.0.0.1

> www.xiemx.com
Server:        192.168.17.6
Address:    192.168.17.6#53

Name:    www.xiemx.com
Address: 192.168.1.100
> mail.xiemx.com
Server:        192.168.17.6
Address:    192.168.17.6#53

Name:    mail.xiemx.com
Address: 192.168.1.101

> ftp.xiemx.com
Server:        192.168.17.6
Address:    192.168.17.6#53

ftp.xiemx.com    canonical name = ftp1.xiemx.com.
Name:    xxx.xiemx.com
Address: 192.168.1.103

> test88.xiemx.com
Server:        192.168.17.6
Address:    192.168.17.6#53

Name:    test88.xiemx.com
Address: 192.168.1.88

> 192.168.1.181
Server:        192.168.17.6
Address:    192.168.17.6#53

181.1.168.192.in-addr.arpa    name = test181.xiemx.com.
```