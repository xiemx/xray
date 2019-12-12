---
layout: post
title: watch重复执行某个命令
published: true
author: xiemx
comments: true
date: 2016-08-11 04:08:29
tags: [ shell, command ]
categories:
    - 'command'
#permalink: '/2016/08/11/shell-command-watch'
---
当需要重复执行一个命令时，可使用`watch`

```shell
➜  ~ watch

Usage:
 watch [options] command

Options:
  -b, --beep             beep if command has a non-zero exit
  -c, --color            interpret ANSI color and style sequences
  -d, --differences[=<permanent>]
                         highlight changes between updates
  -e, --errexit          exit if command has a non-zero exit
  -g, --chgexit          exit when output from command changes
  -n, --interval <secs>  seconds to wait between updates
  -p, --precise          attempt run command in precise intervals
  -t, --no-title         turn off header
  -x, --exec             pass command to exec instead of "sh -c"

 -h, --help     display this help and exit
 -v, --version  output version information and exit

For more details see watch(1).

###每秒执行一次ls命令
watch -n 1 ls
```

