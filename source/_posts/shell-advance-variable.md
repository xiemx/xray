---
layout: post
title: shell 变量特殊操作
published: true
author: xiemx
comments: true
date: 2016-06-29 10:06:08
tags: [ shell, linux ]
categories:
    - shell
    - linux
# permalink: '/2016/06/29/shell-%e5%8f%98%e9%87%8f%e7%89%b9%e6%ae%8a%e6%93%8d%e4%bd%9c'
---

### 替换运算符
```
  ${var:-word}    如果var存在且非null，返回它的值；否则返回word
  ${var:=word}    如果var存在且非null，返回它的值；否则将word赋值给var，并返回var的值
  ${var:?word}    如果var存在且非null，返回它的值；否则显示var:word
  ${var:+word}    如果var存在且非null，返回word；否则返回null
```

### 模式匹配运算符
```
  ${var#pattern}    匹配前缀（最小匹配），并返回余下内容
  ${var##pattern}   匹配前缀（最大匹配），并返回余下内容
  ${var%pattern}    匹配结尾（最小匹配），并返回余下内容
  ${var%%pattern}   匹配结尾（最大匹配），并返回余下内容
```
  注：pattern为正则表达式匹配
