---
layout: post
title: Apache的访问控制
published: true
author: xiemx
comments: true
date: 2016-01-17 10:01:04
tags: [ apache, webserver ]
categories:
    - apache
---
Apache的访问控制两种，一是客户端限制，一是用户验证机制。

* 客户端限制：
```
<Directory /some/dir>
order allow,deny
deny from all
</Directory >
```
 

这就是一个目录限制，他限制所有IP对这个目录的访问。

* 用户验证机制：
```
<Directory /some/dir>
AuthType Basic
AuthName "My Auth File"
AuthUserFile /some/file/path
Require valid-user
</Directory >
```
这就是一个用户验证机制，他要求用户给出用户名和密码才能访问目录下的内容。