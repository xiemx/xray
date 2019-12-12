---
layout: post
title: window系统上部署Zabbix_agent
published: true
author: xiemx
comments: true
date: 2016-04-13 09:04:53
tags: [ zabbix, windows ]
categories:
    - zabbix
#permalink: '/2016/04/13/deploy-zabbix_agent-to-windows'
---

1.获取windows的agent客户端，解压文件至指定位置

	![img](/images/img_570da13048078.png)

2.修改conf文件中的

```shell
Server=server端的IP地址

ServerActive=server端的IP地址

HostName=服务器在监控端上的监控名，可见名称可以不写默认为主机名称
```

![img](/images/img_570da1465a8dc.png)

3.命令行模式下安装Zabbix_agent监控程序

```shell
# 进入解压后的文件夹下

cd  C:\zabbix_agents\bin\win64

#安装程序

zabbix_agentd.exe –c c:\zabbix\zabbix_agentd.conf -i

#启动程序

zabbix_agentd.exe –c c:\zabbix\zabbix_agentd.conf -s

```

![img](/images/img_570da15f9ead1.png)


```markdown
参数含义：
-c    指定配置文件所在位置
-i     安装客户端
-s    启动客户端
-x    停止客户端
-d    卸载客户端
```


4.监控端添加agent服务器注意主机名为conf文件中定义的名称，否则会导致某些监控项目异常

![img](/images/img_570da173ef401.png)

 