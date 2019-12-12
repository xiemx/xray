---
layout: post
title: dns view功能
published: true
author: xiemx
comments: true
date: 2016-01-31 07:01:54
tags: [ dns, view ]
categories:
    - dns
---
假设现在有一个网站服务器存放在一个双线或者多线机房内，这个服务器要提供一个web给全国的用户访问。现在用的用户使用的是电信的网，有的是联通的宽带，有的可能是长城铁通之类的isp，如果让这些用户都去访问我的某一个ip，会导致一部分用户存在跨isp延迟较高的情况，要解决这样的一个情况我们可以在多线机房内申请多个isp的ip绑定到机器上，然后修改dns的解析，让dns根据请求的来源分配给不同的ip。通俗的说就是电信用户走电信线路，联通用户走联通线路。这就要用到dns的一个view功能。

具体配置如下：
```
view "电信-view"{
match-clients { 服务器ip1 };#匹配这个ip/列表，符合则使用下面的服务器进行解析，否则跳出
zone "abc.com" {
type master;
file "dx.abc.com.zone";该zone内服务器ipadd要与match-clients所属isp对应
};
include "/etc/named.rfc1912.zones";#需要包含该行。
}

view "网通-view"{
match-clients { 服务器ip2 };#匹配这个ip/列表，符合则使用下面的服务器进行解析，否则跳出
zone "abc.com" {
type master;
file "wt.abc.com.zone";该zone内服务器ipadd要与match-clients所属isp对应
};
include "/etc/named.rfc1912.zones";#需要包含该行。
}

view "other-view"{
match-clients { any };#匹配这个ip，符合则使用下面的服务器进行解析，否则跳出
zone "abc.com" {
type master;
file "other.abc.com.zone";
};
include "/etc/named.rfc1912.zones";#需要包含该行。
}
```
这里只写出view部分的设置，其他部分设置可以参考dns服务器搭建内容。