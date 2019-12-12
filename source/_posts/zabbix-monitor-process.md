---
layout: post
title: zabbix监控特定进程
published: true
author: xiemx
comments: true
date: 2016-03-25 11:03:04
tags: [ zabbix ]
categories:
    - zabbix

---

1. 在特定机器或模板上创建新的监控项，点击Key 后面的Select 按钮，选择如下两项，一项是用来监控特定进程的数量，另一项是用来监控进程使用内存的大小。

![img](/images/img_56f4a9789a5c5.png)

2. 以下是对squid进程的监控配置，key中的参数说明，第一个参数是进程名字，没必要填写，填了反而会使监控不太准确（仅个人测试），第二个参数是运行进程的用户名，第三个为进程的状态 ，包括：*all*(default),*run*,*sleep*,*zomb*，第四个参数用来指定进程名中包含的字符，对进程进行过滤。

![img](/images/img_56f4a9875257e.png)

3. 配置好监控项后，添加触发器，如下触发器表示最后两次的值都是0，说明没有squid进程在运行，则出发报警。

![img](/images/img_56f4a99aa9435.png)

4. 当我们kill掉进程tomcat会如下报警

![img](/images/img_56f4a9ab888d0.png)