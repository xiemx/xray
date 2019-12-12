---
layout: post
title: dedecms织梦验证码无法正常显示
published: true
author: xiemx
comments: true
date: 2016-05-26 02:05:40
tags: [ php, debug, dedecms ]
categories:
    - php
# permalink: '/2016/05/26/dedecms%e7%bb%87%e6%a2%a6%e9%aa%8c%e8%af%81%e7%a0%81%e6%97%a0%e6%b3%95%e6%ad%a3%e5%b8%b8%e6%98%be%e7%a4%ba'
---
方案一：赋予sessions读、写、可执行的权限

修改根目录下/data/sessions/的sess_***文件修改权限为777（命令：chmod 777 filename）。

方案二：将vdimgck.php替换法

替换前请将当前的vdimgck.php备份。找回相同版本的DEDE安装包，找到/include/vdimgck.php 文件，并用其替换当前站点的vdimgck.php文件。

方案三：去掉登陆验证码代码
  
如果上面的两种解决办法都解决不了，那就直接去掉验证码功能。是修改data\safe\inc_safe_config.php 配置文件。

方法：$safe_gdopen = ’1,2,3,5,6′; 这个就是系统哪些地方开启验证码。与[验证码安全设置]界面是一对一的关系。

所以，如果当我们管理后台想关闭验证码(如果验证码无法正确输入，不支持GB库)的时候，只需要打开data\safe\inc_safe_config.php 将$safe_gdopen = ’1,2,3,5,6′; 中的6删除即可。