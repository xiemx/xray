---
layout: post
title: python中is和==的区别
published: true
author: xiemx
comments: true
date: 2016-03-29 02:03:08
tags: [ python ]
categories:
    - python
# permalink: '/2016/03/29/python%e4%b8%adis%e5%92%8c%e7%9a%84%e5%8c%ba%e5%88%ab'
---
Python中的对象包含三要素：`id`、`type`、`value`
  * id用来唯一标识一个对象
  * type标识对象的类型
  * value是对象的值。
  
`is` 通过id来判断对象是否相等
  
`==` 通过value来判断值是否相等

例：
```python
>>> a = {1:2}
>>> b = a.copy()
>>> a is b
False
>>> a == b
True
>>>
```