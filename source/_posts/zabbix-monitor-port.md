---
layout: post
title: zabbix监控指定端口
published: true
author: xiemx
comments: true
date: 2016-06-29 10:06:32
tags: [ ]
categories:
    - zabbix
#permalink: '/2016/06/29/zabbix-monitor-port'
---

1.登陆zabbix主界面

选择：配置-模板

选择模板组，这里我选择的是Template App Agentless，原因该自带模板组内包含各种常用服务的模板

单击该模板组 项目

![img](/images/img_577330bdc322e.png)点击右上角的 创建项目

![img](/images/img_577330e1e5d06.png)这里添加的是mysql服务，按照如图配置

![img](/images/img_577330f8871b5.png)单击下方 存档 保存。

如图点击 触发器

![img](/images/img_5773310c679a1.png)如图，点击右上方 创建触发器

![img](/images/img_5773313591d94.png)按照如图配置

![img](/images/img_57733153a13cb.png)点击 存档 保存，完毕。

以上是将自定义模板服务添加到Template App Agentless模板组

最后将Template App Agentless模板组添加到你需要监控的客户端主机内