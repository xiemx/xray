---
layout: post
title: iis备份还原
published: true
author: xiemx
comments: true
date: 2017-03-31 10:03:43
tags: [ windows, iis, webserver]
categories:
    - windows
    - iis
---

1. 打开我们的IIS管理器，在功能视图里找到共享的配置这个功能然后双击进入。

![img](/images/img_58ddc18a1993a.png)

2. 进入后单机右上方的“导出配置”选项，选择导出配置文件的物理路径，然后设置一个密码，密码必须是包含数字、符号、大小写字母组合并且至少为8个字符长的强密码，确定导出后会在你导出配置文件目录下生成administration.config、applicationHost.config和configEncKey.key共3个文件，这3个文件就是我们备份的IIS站点配置信息文件。

![img](/images/img_58ddc1b2d8078.png)

3. 现在是还原IIS的配置信息，首先将你导出后的administration.config、applicationHost.config和configEncKey.key这个3个文件复制到你需要恢复IIS配置信息的电脑或服务器上，然后打开IIS，同样在功能视图里找到“共享的配置”并打开。

![img](/images/img_58ddc20c6f8c3.png)

4. 把“启用共享的配置”勾选上，物理路径就选择你备份的文件所在目录，用户名、密码输入框都不需要填写，直接点击右上方的应用，然后它要你输入密码，确定后重启下我们的IIS 就可以看到以前的站点信息都还原了。