---
layout: post
title: PHPCMS V9 “密码重试次数太多，请过-xxx分钟后重新登录！”的解决办法
published: true
author: xiemx
comments: true
date: 2015-09-29 02:09:35
tags: [ php, debug, CMS ]
categories:
    - php
# permalink: '/2015/09/29/phpcms-v9%e5%af%86%e7%a0%81%e9%87%8d%e8%af%95%e6%ac%a1%e6%95%b0%e5%a4%aa%e5%a4%9a%ef%bc%8c%e8%af%b7%e8%bf%87-xxx%e5%88%86%e9%92%9f%e5%90%8e%e9%87%8d%e6%96%b0%e7%99%bb%e5%bd%95%ef%bc%81'
---
找到文件 /phpcms/modules/admin/index.php

将如下代码注释掉：
```php
if($rtime['times'] >= $maxloginfailedtimes) {
$minute = 60-floor((SYS_TIME-$rtime['logintime'])/60);
showmessage(L('wait_1_hour',array('minute'=>$minute)));
}
```
注意哦，一共是4行。