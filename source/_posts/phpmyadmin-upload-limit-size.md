---
layout: post
title: phpmyadmin上传文件大小修改限制
published: true
author: xiemx
comments: true
date: 2015-08-24 02:08:12
tags: [ mysql, phpmyadmin, php, database]
categories:
    - mysql
# permalink: '/2015/08/24/phpmyadmin%e4%b8%8a%e4%bc%a0%e6%96%87%e4%bb%b6%e5%a4%a7%e5%b0%8f%e4%bf%ae%e6%94%b9%e9%99%90%e5%88%b6'
---
修改phpmyadmin上传文件大小限制主要分修改php.ini配置文件和phpmyadmin配置文件两个步骤。

#### 第一步：修改php.ini配置文件中文件上传大小配置**

此步骤与一般的php.ini 配置文件上传功能方法一致，需要修改php.ini配置文件中`upload_max_filesize`和`post_max_size`两个选项值

#### 第二步：修改php执行时间及内存限制实现phpmyadmin上传大文件功能

如果想要phpmyadmin上传大文件，还需修改php.ini配置文件中的`max_execution_time`（php页面执行最大时间）、 `max_input_time`（php页面接受数据最大时间）、`memory_limit`（php页面占用的最大内存）三个配置选项，这是因为 phpmyadmin上传大文件时，php页面的执行时间、内存占用也势必变得更长更大，其需要php运行环境的配合，光修改上传文件大小限制是不够的。

#### 第三步：修改phpmyadmin配置文件

在完成php.ini的相关配置后，还需要修改phpmyadmin配置。

1、修改phpmyadmin config配置文件中的`$cfg［'ExecTimeLimit'］`配置选项，默认值是300，需要修改为0，即没有时间限制。

2、修改phpmyadmin安装根目录下的import页面中的`$memory_limit`

说明：首选读取php.ini配置文件中的内存配置选项memory_limit，如果为空则默认内存大小限制为2M，如果没有限制则内存大小限制为10M，你可以结合你php.ini配置文件中的相关信息修改这段代码。

至此，经过修改php.ini配置文件中的文件上传配置选项以及phpmyadmin配置文件后，即可解决phpmyadmin上传文件大小限制问题，从而实现phpmyadmin上传大文件功能。
