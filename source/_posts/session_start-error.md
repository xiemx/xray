---
layout: post
title: session_start()函数报错
published: true
author: xiemx
comments: true
date: 2015-09-30 01:09:15
tags: [ php, debug ]
categories:
    - php
    - debug
# permalink: '/2015/09/30/session_start-error'
---
### 现象

Warning: session_start() [function.session-start]: Cannot send session cache limiter - headers already sent

### 解决

(PHP 4, PHP 5)

session_start — 启动新会话或者重用现有会话

在调用此函数是不能向浏览器输出内容，如出现上面报错可以将函数放到 ` <？php `最上方顶句如下

```php
<?php
session_start();
include xxxxxxxx;
?>
```