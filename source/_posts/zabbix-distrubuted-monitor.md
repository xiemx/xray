---
layout: post
title: zabbix distrubuted monitor
published: true
author: xiemx
comments: true
date: 2017-09-21 02:09:37
tags: [ zabbix ]
categories:
    - zabbix
#permalink: /2017/09/21/zabbix-distrubuted-monitor
---

zabbix 分布式监控2种模式
* node模式
* proxy模式

PS: node模式官方在2.4版本之后已经弃用，重点讨论proxy模式

### proxy 模式

Zabbix proxy可以代替Zabbix服务器收集性能和可用性数据，一个代理可以承担一些收集数据的负载。使用代理是实现集中式和分布式监控的最简单方法。proxy需要使用单独的数据库来缓存agent数据，在发给server防止出现因网络问题造成的数据丢失。zabbix proxy只是一个数据收集组件，不会触发任何trigger／alert.

![img](/images/img_59c35cd9ab71b.png)

### 使用场景

- Monitor remote locations
- Monitor locations having unreliable communications
- Offload the Zabbix server when monitoring thousands of devices
- Simplify the maintenance of distributed monitoring

### 安装配置

```shell
#同安装zabbix server 类似，不赘述,需要其他功能也可以在编译时自行开启。

./configure --prefix=/opt/zabbix_proxy/ --enable-proxy --with-mysql --with-libcurl
make install

create databases zabbix
grant all to zabbix.* to zabbix@'%' identified by "zabbix";
#导入schema.sql

#配置文件中hostname需要和zabbix上添加的保持一致
#其它参考server设置参数

#ps：设置适当的配置同步时间，默认一小时。建议设置短一点，这样如果有新机器加入配置修改都可以快速同步并监控。
ConfigFrequency=600
```

zabbix server 配置

![img](/images/img_59c35d1d930c1.png)

添加主机时选择指定的proxy

![img](/images/img_59c35d389b1c5.png)