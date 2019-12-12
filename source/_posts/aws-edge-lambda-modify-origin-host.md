---
layout: post
title: 通过lambda修改AWS CloudFront回源host
published: true
author: xiemx
comments: true
date: 2018-10-11 03:10:17
tags: [ aws, lambda, cloudfront ]
categories:
    - aws
---
在使用aws cloudfront时发现cloudfront默认不允许自定义回源请求头的Host字段，对于一些情况我们需要使用这个host+ip来回源的时候就有点坑了，这个时候我们可以通过使用aws的lambda@edge，去修改request的header来实现自定义host来回源。

aws lambda@edge文档：https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/lambda-edge.html

1. 在创建lambda函数，并发布一个版本注意只能在us-east-1这个区创建，否则在附加到cloudfront的时候会报错不支持的区域，代码如下

![img](/images/img_5bbef98529a61.png)

```
'use strict';

// force a specific Host header to be sent to the origin

exports.handler = (event, context, callback) => {
    const request = event.Records[0].cf.request;
    request.headers.host[0].value = 'www.xiemx.com';
    return callback(null, request);
};
```

2. 在cloudfront的Behavior菜单中

![img](/images/img_5bbefa0f833d9.png)

3. 重新deploy cdn和刷新一次cdn缓存