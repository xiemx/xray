---
layout: post
title: redis-dump使用
published: true
author: xiemx
comments: true
date: 2019-05-06 11:05:33
tags: [ redis ]
categories:
    - redis
# permalink: '/2019/05/06/redis-dump'
---
* 安装redis-dump/redis-load

```shell
#需要ruby 2.2.2以上版本（可以直接使用ruby:2.2.3的docker images）
gem install redis-dum
```

* dump redis
```shell
root@5dba1bd8fa77:/# redis-dump -h
  Try: /usr/local/bundle/bin/redis-dump show-commands
Usage: /usr/local/bundle/bin/redis-dump [global options] COMMAND [command options]
    -u, --uri=S                      Redis URI (e.g. redis://hostname[:port])
    -d, --database=S                 Redis database (e.g. -d 15)
    -a, --password=S                 Redis password (e.g. -a 'my@pass/word')
    -s, --sleep=S                    Sleep for S seconds after dumping (for debugging)
    -c, --count=S                    Chunk size (default: 10000)
    -f, --filter=S                   Filter selected keys (passed directly to redis' KEYS command)
    -b, --base64                     Encode key values as base64 (useful for binary values)
    -O, --without_optimizations      Disable run time optimizations
    -V, --version                    Display version
    -D, --debug
        --nosafe

root@5dba1bd8fa77:/# redis-dump -u aux-redis.1uvkyf.0001.cnn1.cache.amazonaws.com.cn > redis-uat.json
```

* load redis
```shell
root@5dba1bd8fa77:/# redis-load -h
  Try: /usr/local/bundle/bin/redis-load show-commands
Usage: /usr/local/bundle/bin/redis-load [global options] COMMAND [command options]
    -u, --uri=S                      Redis URI (e.g. redis://hostname[:port])
    -d, --database=S                 Redis database (e.g. -d 15)
    -a, --password=S                 Redis password (e.g. -a 'my@pass/word')
    -s, --sleep=S                    Sleep for S seconds after dumping (for debugging)
    -b, --base64                     Decode key values from base64 (used with redis-dump -b)
    -n, --no_check_utf8
    -V, --version                    Display version
    -D, --debug
        --nosafe


root@5dba1bd8fa77:/# redis-load -u aux-redis.1uvkyf.0001.cnn1.cache.amazonaws.com.cn:6379/0 < redis-uat.json
```