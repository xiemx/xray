---
layout: post
title: zabbix-server.conf文件详解
published: true
author: xiemx
comments: true
date: 2016-09-01 05:09:48
tags: [ ]
categories:
    - zabbix
#permalink: '/2016/09/01/zabbix-server-configure'
---

```markdown
AlertScriptsPath 默认值：/usr/local/share/zabbix/alertscripts 说明：告警脚本目录

AllowRoot 默认值：0 说明：是否允许使用root启动，0:不允许，1:允许，默认情况下她会使用zabbix用户来启动zabbix进程，不推荐使用root

CacheSize 取值范围： 128K-8G 默认值：8M 说明：配置缓存，用于存储host，item，trigger数据，2.2.3版本之前最大支持2G，目前最大支持8G，一般用不了多少的。

CacheUpdateFrequency 取值范围：1-3600 默认值：60 说明：多少秒更新一次配置缓存

DBHost 默认值：localhost 说明：数据库主机地址

DBName 默认值：无 必填：是

DBPassword： 默认值：空 说明：数据库密码

DBPort 取值范围：1024-65535 默认值:3306 说明：SQLite作为DB，这个选项请忽略，如果使用socket链接，也请忽略。

DBSchema 说明：Schema名称. 用于 IBM DB2 、 PostgreSQL.

DBSocket 默认值：/tmp/mysql.sock 说明：mysql sock文件路径

DebugLevel 取值范围：0-5 默认值：3 说明: 指定debug级别 0 - 基本信息 1 - critical信息 2 - error信息 3 - warnings信息 4 - 调试日志，日志内容很多，慎重使用 5 - 用于调试web和vmware监控

ExternalScripts 默认值： /usr/local/share/zabbix/externalscripts 说明： 外部脚本目录

Fping6Location 默认值：/usr/sbin/fping6 说明：fping6路径，不懂fping的人可以百度一下，如果zabbix非root启动，请给fping6 SUID

FpingLocation 默认值：/usr/sbin/fping 说明:和上面的一样

HistoryCacheSize 取值范围：128K-2G 默认值：8M 说明： 历史记录缓存大小，用于存储历史记录

HistoryTextCacheSize 取值范围：128K-2G 默认值：16M 说明：文本类型历史记录的缓存大小，存储character, text 、log历史记录.

HousekeepingFrequency 取值范围：0-24 默认值：1 说明：housekeep执行频率，默认每小时回去删除一些过期数据。如果server重启，那么30分钟之后才执行一次，接下来，每隔一小时在执行一次。

Include 说明：include配置文件，可以使用正则表达式，例如：/usr/local/zabbix-2.4.4/conf/ttlsa.com/*.conf

JavaGateway 说明：Zabbix Java gateway的主机名，需要启动Java pollers

JavaGatewayPort 取值范围：1024-32767 默认值：10052 Zabbix Java gateway监听端口

ListenIP 默认值：0.0.0.0 说明：监听地址，留空则会在所有的地址上监听，可以监听多个IP地址，ip之间使用逗号分隔，例如：127.0.0.1,10.10.0.2

ListenPort 取值范围：1024-32767 默认值：10051 说明：监听端口

LoadModule 说明：加载模块，格式: LoadModule=，文件必须在指定的LoadModulePath目录下，如果需要加载多个模块，那么写多个即可。

LoadModulePath 模块目录，参考上面

LogFile 日志文件，例如：/data/logs/zabbix/zabbix-server.log

LogFileSize 取值范围：0-1024 默认值：1 0表示禁用日志自动rotation，如果日志达到了限制，并且rotation失败，老日志文件将会被清空掉，重新生成一个新日志。

LogSlowQueries 取值范围：0-3600000 默认值：0 多慢的数据库查询将会被记录，单位：毫秒，0表示不记录慢查询。只有在DebugLevel=3时，这个配置才有效。

MaxHousekeeperDelete 取值范围： 0-1000000 默认值：5000 housekeeping一次删除的数据不能大于MaxHousekeeperDelete

PidFile 默认值：/tmp/zabbix_server.pid PID文件

ProxyConfigFrequency 取值范围：1-604800 默认值：3600 proxy被动模式下，server多少秒同步配置文件至proxy。

ProxyDataFrequency 取值范围：1-3600 默认值:1 被动模式下，zabbix server间隔多少秒向proxy请求历史数据

SenderFrequency 取值范围：5-3600 默认值：30 间隔多少秒，再尝试发送为发送的报警

SNMPTrapperFile 默认值：/tmp/zabbix_traps.tmp SNMP trap发送到server的数据临时存放文件。

SourceIP 出口IP地址

SSHKeyLocation SSH公钥私钥路径

SSLCertLocation SSL证书目录，用于web监控

SSLKeyLocation SSL认证私钥路径、用于web监控

SSLCALocation SSL认证,CA路径，如果为空，将会使用系统默认的CA

StartDBSyncers 取值范围：1-100 默认值：4 预先foke DB Syncers的数量，1.8.5以前最大值为64

StartDiscoverers 取值范围：0-250 默认值：1 pre-forked discoverers的数量，1.8.5版本以前最大可为255

StartHTTPPollers 取值范围：0-1000 默认值：1 pre-forked HTTP pollers的数量，1.8.5以前最大255

StartIPMIPollers 取值范围：0-1000 默认值：0 pre-forked IPMI pollers的数量，1.8.5之前，最大为255

Timeout 取值范围：1-30 默认值：3 agent，snmp，external check的超时时间，单位为秒

TmpDir 默认值：/tmp

TrapperTimeout 取值范围：1-300 默认值：300 处理trapper数据的超时时间

TrendCacheSize 取值范围：128K-2G 默认值：4M 历史数据缓存大小

UnavailableDelay 取值范围：1-3600 默认值：60 间隔多少秒再次检测主机是否可用

UnreachableDelay 取值范围：1-3600 默认值：15 间隔多少秒再次检测主机是否可达。

UnreachablePeriod 取值范围：1-3600 默认值：45 检测到主机不可用，多久将它置为不可达

User 默认值：zabbix 启动zabbix server的用户，在配置禁止root启动，并且当前shell用户是root得情况下有效。如果当前用户是xiemx，那么zabbix server的运行用户是xiemx

ValueCacheSize 取值范围：0,128K-64G 默认值：8M 0表示禁用，history value缓存大小，当缓存超标了，将会每隔5分钟往server日志里面记录。养成看日志的好习惯。
```