---
layout: post
title: SSH的AuthorizedKeysCommand、AuthorizedKeysCommandUser
published: true
author: xiemx
comments: true
date: 2019-05-06 11:05:17
tags: [ ssh, linux ]
categories:
    - linux
# permalink: '/2019/05/06/ssh-authorizedkeyscommand-cauthorizedkeyscommanduser'
---
* AuthorizedKeysCommand 可以指定运行一个脚本，而这个脚本主要是寻找登录用户的publickey，默认传参为登录用户名，若未认证成功，将继续使用AuthorizedKeysFile文件来做认证。
* AuthorizedKeysCommandUser就是指定以什么用户来运行这个脚本。 这两个配置选项的一个用处就是在用户管理上可以不再依靠本地管理，而可以通过脚本读取远程数据库系统中的用户的publickey进行认证，例如MySQL或者LDAP，这样的话，更便于用户的集中管理。