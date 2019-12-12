---
layout: post
title: 访问网站要求输入密码
published: true
author: xiemx
comments: true
date: 2015-08-24 02:08:15
tags: [ windows, iis]
categories:
    - windows
    - iis
---
访问网站要求输入密码：
![image-20191021135643975](/images/image-20191021135643975.png)

### 方法1

1. Internet—域名属性—目录安全性—编辑身份认证和访问控制——修改密码
![image-20191021135659882](/images/image-20191021135659882.png)

2. 网站应用程序池—属性—标识——修改密码(密码和上面相同)
![image-20191021135714997](/images/image-20191021135714997.png)

### 方法2：

1. 删除权限中不统一的用户
![image-20191021135724279](/images/image-20191021135724279.png)

