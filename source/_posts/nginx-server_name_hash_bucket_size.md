---
layout: post
title: nginx server_name_hash_bucket_size
published: true
author: xiemx
comments: true
date: 2017-01-18 11:01:15
tags: [ nginx ]
categories:
    - nginx
---

nginx默认的 `server_name_hash_bucket_size` 一般为32/64，
如果配置了超长的server_name建议适当增大此参数
```
http {
    server_names_hash_bucket_size 512;
}
```