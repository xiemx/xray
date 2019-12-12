---
layout: post
title: >
    Zabbix自定义Item Not
    Supported,页面出现不支持解决
published: true
author: xiemx
comments: true
date: 2016-11-23 04:11:59
tags: [ zabbix ]
categories:
    - zabbix
    - issue
#permalink: '/2016/11/23/fix-zabbix-item-not-supported'
---

使用Zabbix的时候往往会自定义Item。
但是经常会遇到自定义的`Item Not Supported`了。
Zabbix Agent默认的超时时间是`3`秒。往往我们自定义的Item由于各种原因返回时间会比较长。建议修改一个适合自己实际的值。


```shell
vim /etc/zabbix/zabbix_agent.conf
#Range: 1-30
Timeout=8

###修改完毕后重启zabbix-agent
/etc/init.d/zabbix-agent restart
```