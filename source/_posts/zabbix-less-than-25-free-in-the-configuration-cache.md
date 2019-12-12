---
layout: post
title: 'zabbix Less than 25% free in the configuration cache解决'
published: true
author: xiemx
comments: true
date: 2016-08-09 03:08:20
tags: [ zabbix, issue ]
categories:
    - zabbix
#permalink: '/2016/08/09/zabbix-less-than-25-free-in-the-configuration-cache'
---

在zabbix server默认配置下，出现告警

```markdown
Less than 25% free in the configuration cache
```
可增加zabbix配置缓存解决

```shell
vim zabbix_server.conf
##将缓存从8M提升到16M，可以调到最高8G
CacheSize=16M

##重启zabbix server
service zabbix_server restart
```