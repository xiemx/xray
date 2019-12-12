---
layout: post
title: uwsgi 报错 invalid option --“x” getopt_long() error
published: true
author: xiemx
comments: true
date: 2016-07-26 11:07:34
tags: [ python, issue, uwsgi ]
categories:
    - 'uwsgi'
    - 'python'
#permalink: '/2016/07/26/uwsgi-invalid-option-x-getopt_long-error'
---
### 报错：

```markdown
uwsgi: invalid option — ‘x’
getopt_long() error 
```

### 解决

这个问题是因为编译uwsgi的时候少了libxml2库导致的，应该先安装库再编译，否则会少了xml的支持。

1. uninstall uwsgi

2. install  libxml*  

3. install uwsgi

