---
layout: post
title: http cache
published: true
author: xiemx
comments: true
date: 2019-05-13 11:05:02
tags: [ http, cache ]
categories:
    - http
---
#### **cache流程图**

![img](/images/image.png)

### “no-cache”和“no-store”

“no-cache”表示必须先与服务器确认返回的响应是否发生了变化，然后才能使用该响应来满足后续对同一网址的请求。 因此，如果存在合适的验证令牌 (ETag)，no-cache 会发起往返通信来验证缓存的响应，但如果资源未发生变化，则可避免下载。

“no-store”禁止浏览器以及所有中间缓存存储任何版本的返回响应，例如，包含个人隐私数据或银行业务数据的响应。 每次用户请求该资产时，都会向服务器发送请求，并下载完整的响应。

### “public”与 “private”

“public”则即使它有关联的 HTTP 身份验证，甚至响应状态代码通常无法缓存，也可以缓存响应。 大多数情况下，“public”不是必需的，因为明确的缓存信息（例如“max-age”）已表示响应是可以缓存的。

“private”浏览器可以缓存响应，不允许任何中间缓存对其进行缓存。 例如，用户的浏览器可以缓存包含用户私人信息的 HTML 网页，但 CDN 却不能缓存。

### “max-age”

指令指定从请求的时间开始，允许提取的响应被重用的最长时间（单位：秒）。 例如，“max-age=60”表示可在接下来的 60 秒缓存和重用响应。

## 通过 ETag 验证缓存的响应

在首次请求资源时服务器生成并返回"ETag" http请求头(通常是文件内容的哈希值或某个其他指纹)。 当120 秒后，浏览器又对该资源发起了新的请求。 首先，浏览器会检查本地缓存并找到之前的响应。如果发现缓存超过max-age, 浏览器将发起一个带有"If-None-Match"的http请求。 如果Etag相同，则返回304，使用本地缓存。

![HTTP Cache-Control 示例](/images/http-cache-control.png)

参考： https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching