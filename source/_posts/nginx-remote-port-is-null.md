---
layout: post
title: nginx remote_port为空
published: true
author: xiemx
comments: true
date: 2019-12-12 10:01:04
tags: [ nginx, webserver ]
categories:
    - nginx
---
### nginx remote_port为空

#### 现象
由于网安要求记录用户请求来源端口到日志中，因此为了实现这个需求在log_format中增加了"ngx_remote_port: $remote_port"字段(如下)， 但实际日志系统中收录到的日志`ngx_remote_port:""`为空
```
log_format json '{ "time": "$time_iso8601", '
                    '"ngx_true_client_ip": "$http_true_client_ip", '
                    '"ngx_x_real_ip": "$http_x_real_ip", '
                    '"ngx_cdn_src_ip": "$http_cdn_src_ip", '
                    '"ngx_geo_location_ip": "$client_ip", '
                    '"ngx_remote_addr": "$remote_addr", '
                    '"ngx_remote_port": "$remote_port", '
                    '"ngx_x_user_site_proxy": "$http_x_user_site_proxy", '
                    '"ngx_x_forwarded_for": "$http_x_forwarded_for", '
                    '"ngx_host": "$host", '
                    '"ngx_http_user_agent": "$http_user_agent", '
                    '"ngx_body_bytes_sent": "$body_bytes_sent", '
                    '"ngx_request_time": "$request_time", '
                    '"ngx_upstream_response_time": "$upstream_response_time", '
                    '"ngx_upstream_connect_time": "$upstream_connect_time", '
                    '"ngx_upstream_header_time:": "$upstream_header_time", '
                    '"ngx_upstream_addr": "$upstream_addr", '
                    '"ngx_upstream_content_length": "$sent_http_content_length", '
                    '"ngx_status_code": "$status", '
                    '"ngx_scheme": "$scheme", '
                    '"ngx_request_method": "$request_method", '
                    '"ngx_request_uri": "$request_uri", '
                    '"ngx_request": "$request", '
                    '"ngx_http_referrer": "$http_referer"  }';
```

#### 解决

由于$remote_port这个变量是nginx核心模块提供的，因此猜测是由第三方模块再次操作导致为空，经过测试发现通过proxy进来的请求remote_port为空，  
而直接访问本地nginx的请求都能够正确获取port信息，对于以上2中情况的对比
怀疑是由于经过proxy之类的组建request header中有x-forworder-for的头，从而触发了realip 模块（这个怀疑没有具体验证），解决方案就是使用realip模块提供的变量'realip_remote_port'来记录来源port, 修改log_format
```
log_format json '{ "time": "$time_iso8601", '
                    '"ngx_true_client_ip": "$http_true_client_ip", '
                    '"ngx_x_real_ip": "$http_x_real_ip", '
                    '"ngx_cdn_src_ip": "$http_cdn_src_ip", '
                    '"ngx_geo_location_ip": "$client_ip", '
                    '"ngx_remote_addr": "$remote_addr", '
                    '"ngx_remote_port": "$remote_port", '
```