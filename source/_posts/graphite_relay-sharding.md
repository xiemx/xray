---
layout: post
title: graphite_relay sharding
published: true
author: xiemx
comments: true
date: 2017-09-08 06:09:47
tags: [ ]
categories:
    - statsd
    - graphite
---

当statsd发送超量的metrics到graphite中，graphite单节点无法负载的情况，可以使用consistent-hashing的模式来将数据分片到backend中。同样的consistent-hashing模式下可以自动剔除／加入节点。

官方文档：

```
carbon-relay.py serves two distinct purposes: replication and sharding.

When running with RELAY_METHOD = rules, a carbon-relay.py instance can run in place of a carbon-cache.py server and relay all incoming metrics to multiple backend carbon-cache.py‘s running on different ports or hosts.

In RELAY_METHOD = consistent-hashing mode, a DESTINATIONS setting defines a sharding strategy across multiple carbon-cache.py backends. The same consistent hashing list can be provided to the graphite webapp via CARBONLINK_HOSTS to spread reads across the multiple backends.
本例模拟一个双后端的carbon-cache instance.单机器运行,使用carbon-cache.py 的instance功能

config文件：
#####carbon.conf
[cache:a]
LINE_RECEIVER_PORT = 2203
PICKLE_RECEIVER_PORT = 2204
CACHE_QUERY_PORT = 7102
LOCAL_DATA_DIR = /opt/graphite/storage/whisper_a/
[cache:b]
LINE_RECEIVER_PORT = 2205
PICKLE_RECEIVER_PORT = 2206
CACHE_QUERY_PORT = 7202
LOCAL_DATA_DIR = /opt/graphite/storage/whisper_b/
[relay]
LINE_RECEIVER_INTERFACE = 0.0.0.0
LINE_RECEIVER_PORT = 2003
PICKLE_RECEIVER_INTERFACE = 0.0.0.0
PICKLE_RECEIVER_PORT = 2004
DESTINATIONS = 127.0.0.1:2204:a, 127.0.0.1:2206:b 
启动服务：

启动instance:a
xmx@xiemx-test:/opt/graphisudo ./carbon-cache.py --instance=a --config=/opt/graphite/conf/carbon.conf start
Starting carbon-cache (instance a)

启动instance:b
xmx@xiemx-test:/opt/graphite/bin$ sudo ./carbon-cache.py --instance=b --config=/opt/graphite/conf/carbon.conf start
Starting carbon-cache (instance b)

启动relay:
xmx@xiemx-test:/opt/graphite/conf$ sudo /opt/graphite/bin/carbon-relay.py --debug --config=/opt/graphite/conf/carbon.conf start
Starting carbon-relay (instance a)
08/09/2017 16:02:16 :: [console] Using sorted write strategy for cache
08/09/2017 16:02:16 :: [clients] connecting to carbon daemon at 127.0.0.1:2204:a
08/09/2017 16:02:16 :: [clients] connecting to carbon daemon at 127.0.0.1:2206:b
08/09/2017 16:02:16 :: [console] twistd 16.4.1 (/usr/bin/python 2.7.6) starting up.
08/09/2017 16:02:16 :: [console] reactor class: twisted.internet.epollreactor.EPollReactor.
08/09/2017 16:02:16 :: [console] Starting factory CarbonClientFactory(127.0.0.1:2206:b)
08/09/2017 16:02:16 :: [clients] CarbonClientFactory(127.0.0.1:2206:b)::startedConnecting (127.0.0.1:2206)
08/09/2017 16:02:16 :: [console] Starting factory CarbonClientFactory(127.0.0.1:2204:a)
08/09/2017 16:02:16 :: [clients] CarbonClientFactory(127.0.0.1:2204:a)::startedConnecting (127.0.0.1:2204)
08/09/2017 16:02:16 :: [console] CarbonReceiverFactory starting on 2003
08/09/2017 16:02:16 :: [console] Starting factory 
08/09/2017 16:02:16 :: [console] CarbonReceiverFactory starting on 2004
08/09/2017 16:02:16 :: [console] Starting factory 
08/09/2017 16:02:16 :: [clients] CarbonClientProtocol(127.0.0.1:2206:b)::connectionMade
08/09/2017 16:02:16 :: [clients] CarbonClientFactory(127.0.0.1:2206:b)::connectionMade (CarbonClientProtocol(127.0.0.1:2206:b))
08/09/2017 16:02:16 :: [clients] Destination is up: 127.0.0.1:2206:b
08/09/2017 16:02:16 :: [clients] CarbonClientProtocol(127.0.0.1:2204:a)::connectionMade
08/09/2017 16:02:16 :: [clients] CarbonClientFactory(127.0.0.1:2204:a)::connectionMade (CarbonClientProtocol(127.0.0.1:2204:a))
08/09/2017 16:02:16 :: [clients] Destination is up: 127.0.0.1:2204:a
```

测试：

模拟5个客户端同时发送100个key

![img](/images/img_59b26b3126991.png)

![img](/images/img_59b26b4437729.png)

模拟node掉线重连

![img](/images/img_59b26b60dc7e7.png)

graphite-web数据聚合展示

```
修改local_settings.py
CARBONLINK_HOSTS = ["127.0.0.1:7102:a", "127.0.0.1:7202:b"]

启动django
sudo PYTHONPATH=/opt/graphite/webapp django-admin.py runserver 0.0.0.0:5000 --settings
```