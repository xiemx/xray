---
layout: post
title: nginx websocket proxying
published: true
author: xiemx
comments: true
date: 2016-12-14 01:12:43
tags: [ nginx, websocket ]
categories:
    - nginx
# permalink: '/2016/12/14/nginx%e6%94%af%e6%8c%81ssh%e8%bd%ac%e5%8f%91%ef%bc%8dwebsocket-proxying'
---
```
server {
        ...

        location /chat/ {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }
    }

``` 
官方关于websocket proxying的文档：http://nginx.org/en/docs/http/websocket.html