---
layout: post
title: django重置管理员密码
published: true
author: xiemx
comments: true
date: 2016-09-29 03:09:37
tags: [ python, django, webserver ]
categories:
    - python
---
忘记Django密码，使用如下操作即可找回
```
> cd /you/project/path  # 进入你的项目目录
> python manage.py shell # 进入django shell

> from django.contrib.auth.models import User
> user =User.objects.get(username='admin')
> user.set_password('123456')
> user.save()
```
再使用123456密码登录即可