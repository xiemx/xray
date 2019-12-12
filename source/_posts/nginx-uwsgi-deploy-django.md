---
layout: post
title: nginx+uwsgi部署django项目
published: true
author: xiemx
comments: true
date: 2016-07-26 06:07:48
tags: [ uwsgi, django, python, nginx ]
categories:
  - python
  - uwsgi
# permalink: '/2016/07/26/nginxuwsgi%e9%83%a8%e7%bd%b2django%e9%a1%b9%e7%9b%ae'
---
### 安装基本软件
```shell
sudo apt-get install python-dev nginx
pip install uwsgi
```
### 配置uwsgi和django的集成
```shell
vim test.py 创建test.py,添加如下代码
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return "Hello World"

然后执行shell命令：
uwsgi –http :8001 –wsgi-file test.py

访问网页：
curl http://127.0.0.1:8001/
Hello World
```
#编写django_wsgi.py文件，将其放在与文件manage.py同一个目录下：
```python
### vim django_wsgi.py 添加如下代码：
#!/usr/bin/env python
# coding: utf-8
import os
import sys
# 将系统的编码设置为UTF8
reload(sys)
sys.setdefaultencoding('utf8')

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "yoursite.settings")

from django.core.handlers.wsgi import WSGIHandler
application = WSGIHandler()
```
连接django和uwsgi，实现简单的WEB服务器。
我们假设你的Django项目的地址是`/home/work/src/sites/testdjango1/testdjango/mysite`，

然后，就可以执行以下命令：
`uwsgi –http :8000 –chdir /home/work/src/sites/testdjango1/testdjango/mysite –module django_wsgi`
这样，你就可以在浏览器中访问你的Django程序了。所有的请求都是经过uwsgi传递给Django程序的。

### 集成django,uwsgi和nginx部署：
* 在django项目根目录创建启动uwsgi的xml文件：
```
<uwsgi>
    <socket>:8077</socket>
    <chdir>/home/work/src/sites/testdjango1/testdjango/mysite</chdir>
    <module>django_wsgi</module>
    <processes>4</processes> <!-- 进程数 --> 
    <daemonize>uwsgi.log</daemonize>
</uwsgi>
```
* 配置Nginx服务器：
```
#备份nginx配置文件：
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
vim /etc/nginx/sites-available/default 修改如下：

server {

        listen   80;
        server_name localhost;
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
         include        uwsgi_params;
         uwsgi_pass     127.0.0.1:8077;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        #method first
        location ~ ^/static/ {
            root  /home/hz/PycharmProjects/myscrapy/check_ip/;
            expires 24h;
            access_log off;
        }
        #method second
        #location ~* ^.+\.(css|js|json|ttf|woff|map|woff2)${
        #    root /home/hz/PycharmProjects/myscrapy/check_ip/;
        #    access_log off;
        #    expires 24h;
        #}      
}
```
如果不能访问日志文件，修改相关文件的权限即可
* 验证测试各步骤结果
```shell
重启Nginx服务器，以使Nginx的配置生效。
nginx -s  reload
重启后检查Nginx日志是否有异常。

启动uWSGI服务器

cd /home/work/src/sites/testdjango1/testdjango/mysite

uwsgi -x djangochina_socket.xml
```
* django搜集静态文件
```shell
settings.py文件中指定STATIC_ROOT路径
python manage.py collectstatic

修改nginx配置文件指定目录到STATIC_ROOT
```