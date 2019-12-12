---
layout: post
title: nginx防盗链配置
published: true
author: xiemx
comments: true
date: 2017-01-18 02:01:18
tags: [ nginx ]
categories:
    - nginx
# permalink: '/2017/01/18/nginx-referer'
---
```
location ~* \.(gif|jpg|jpeg|png|ico)$ {
    valid_referers none blocked server_names *.xiemx.com;
    if ($invalid_referer) {
        return 444
    }
}
```