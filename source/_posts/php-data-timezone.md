---
layout: post
title: PHP 调用date()函数时区报错
published: true
author: xiemx
comments: true
date: 2015-10-11 04:10:30
tags: [ php, debug ]
categories:
    - php
---
### 现象：
```
  “PHP Warning: date() [function.date]: It is not safe to rely on the system’s timezone settings.
  You are *required* to use the date.timezone setting or the date_default_timezone_set() function.
  In case you used any of those methods and you are still getting this warning, you most likely
  misspelled the timezone identifier. We selected ‘UTC’ for ’8.0/no DST’ instead in”
```
php 有些版本默认的时区为格林威治标准时间，我们需要调整时区为+0800 “Asia/shanghai”
### 解决
以下是三种方法(任选一种都行)：
```
1. 在页头使用date_default_timezone_set()设置 date_default_timezone_set('PRC');
2. 在页头使用 ini_set('date.timezone','Asia/Shanghai');
3. 修改php.ini。打开php.ini查找date.timezone 去掉前面的分号修改成为：date.timezone = PRC
```