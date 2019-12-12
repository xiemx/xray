---
layout: post
title: rbash创建只读用户
published: true
author: xiemx
comments: true
date: 2016-06-02 02:06:03
tags: [ command, rbash, bash, linux ]
categories:
    - linux
# permalink: '/2016/06/02/rbash-add-readonly-user'
---
网上看到一篇用rbash来创建受限用户的文章，来源忘记了，mark在这里
### 受限bash
如果 bash 以 rbash 为程序名启动或者命令行参数有 -r 选项，则启动的这个 shell 会在某些功能上受限制．具体表现为如下操作都不能做：
```
通过 cd 来改变工作目录
设置或取消环境变量： SHELL， PATH， ENV， BASH_ENV
命令名中不能包含目录分隔符 ‘/’
包含有 ‘/’ 的文件名作为内置命令 ‘.’ 的参数
hash 内置命令有 -p 选项时的文件名参数包含 '/'
在启动时通过 shell 环境导入函数定义
在启动时通过 shell 环境解析 SHELLOPTS 的值
使用 >，>|， <>， >&， &>， >> 等重定向操作符
使用 exec 内置命令
通过 enable 内置命令的 -f 和 -d 选项增加或删除内置命令
使用 enable 内置命令来禁用或启用 shell 内置命令
执行 command 内置命令时加上 -p 选项
通过 set +r 或 set +o restricted 关闭受限模式
```
### 步骤
```shell
ln -s /bin/bash  /bin/rbash
useradd -s /bin/rbash rttlsa
passwd rttlsa
mkdir /home/rttlsa/bin
chown root. /home/rttlsa/.bash_profile 
chmod 755 /home/rttlsa/.bash_profile
vi /home/rttlsa/.bash_profile 
.bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
  . ~/.bashrc
fi

# User specific environment and startup programs
PATH=$HOME/bin
export PATH
ln -s /bin/cat  /home/rttlsa/bin/cat  将允许执行的命令链接到$HOME/bin目录
```
如此即可创建只允许查看日志的只读用户。