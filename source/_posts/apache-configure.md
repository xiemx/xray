---
layout: post
title: Apache配置文件参数详解
published: true
author: xiemx
comments: true
date: 2016-01-17 11:01:54
tags: [ apache, webserver]
categories:
    - apache
---
```conf
[root@localhost ~]# grep -v '^#\|^$\|#' /etc/httpd/conf/httpd.conf
ServerTokens OS   当服务器响应主机头（header）信息时显示Apache的版本和

操作系统名称
ServerRoot "/etc/httpd"   设置服务器的根目录
PidFile run/httpd.pid     PID存放位置
Timeout 60                若60秒后没有收到或送出任何数据就切断该连接
KeepAlive Off             是否开启保持链接状态
MaxKeepAliveRequests 100  在使用保持连接功能时，设置客户一次请求连接能

响应文件的最大上限
KeepAliveTimeout 15       在使用保持连接功能时，两个相邻的连接的时间间

隔超过15秒，就切断连接
<IfModule prefork.c>      设置使用Prefork MPM运行方式的参数，此运行方式

是Red hat默认的方式
StartServers       8      设置服务器启动时运行的进程数
MinSpareServers    5      最小空闲进程数
MaxSpareServers   20      最大空闲进程数
ServerLimit      256      最大的进程数
MaxClients       256      最大的请求并发MaxClients=ServerLimit*进程的线

程数
MaxRequestsPerChild  4000 限制每个子进程在结束处理请求之前能处理的连接

请求为1000
</IfModule>
<IfModule worker.c>      设置使用Worker MPM运行方式的参数
StartServers         4
MaxClients         300
MinSpareThreads     25
MaxSpareThreads     75
ThreadsPerChild     25
MaxRequestsPerChild  0
</IfModule>
Listen 80    监听端口
LoadModule auth_basic_module modules/mod_auth_basic.so
LoadModule auth_digest_module modules/mod_auth_digest.so
LoadModule authn_file_module modules/mod_authn_file.so
LoadModule authn_alias_module modules/mod_authn_alias.so
LoadModule authn_anon_module modules/mod_authn_anon.so
LoadModule authn_dbm_module modules/mod_authn_dbm.so
LoadModule authn_default_module modules/mod_authn_default.so
LoadModule authz_host_module modules/mod_authz_host.so
LoadModule authz_user_module modules/mod_authz_user.so
LoadModule authz_owner_module modules/mod_authz_owner.so
LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
LoadModule authz_dbm_module modules/mod_authz_dbm.so
LoadModule authz_default_module modules/mod_authz_default.so
LoadModule ldap_module modules/mod_ldap.so
LoadModule authnz_ldap_module modules/mod_authnz_ldap.so
LoadModule include_module modules/mod_include.so
LoadModule log_config_module modules/mod_log_config.so
LoadModule logio_module modules/mod_logio.so
LoadModule env_module modules/mod_env.so
LoadModule ext_filter_module modules/mod_ext_filter.so
LoadModule mime_magic_module modules/mod_mime_magic.so
LoadModule expires_module modules/mod_expires.so
LoadModule deflate_module modules/mod_deflate.so
LoadModule headers_module modules/mod_headers.so
LoadModule usertrack_module modules/mod_usertrack.so
LoadModule setenvif_module modules/mod_setenvif.so
LoadModule mime_module modules/mod_mime.so
LoadModule dav_module modules/mod_dav.so
LoadModule status_module modules/mod_status.so
LoadModule autoindex_module modules/mod_autoindex.so
LoadModule info_module modules/mod_info.so
LoadModule dav_fs_module modules/mod_dav_fs.so
LoadModule vhost_alias_module modules/mod_vhost_alias.so
LoadModule negotiation_module modules/mod_negotiation.so
LoadModule dir_module modules/mod_dir.so
LoadModule actions_module modules/mod_actions.so
LoadModule speling_module modules/mod_speling.so
LoadModule userdir_module modules/mod_userdir.so
LoadModule alias_module modules/mod_alias.so
LoadModule substitute_module modules/mod_substitute.so
LoadModule rewrite_module modules/mod_rewrite.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
LoadModule cache_module modules/mod_cache.so
LoadModule suexec_module modules/mod_suexec.so
LoadModule disk_cache_module modules/mod_disk_cache.so
LoadModule cgi_module modules/mod_cgi.so
LoadModule version_module modules/mod_version.so
以上为动态加载的模块
Include conf.d/*.conf                   将/etc/httpd/conf.d目录下所有以conf结尾的配置文件包含进来
User apache              进程运行账户
Group apache           进程运行群组
ServerAdmin root@localhost             Apache服务器管理员的E_mail地址
UseCanonicalName Off
DocumentRoot "/var/www/html"         设置网站根目录
<Directory />  设置apache服务器根的访问权限
Options FollowSymLinks             允许符号链接
AllowOverride None               禁止读取.htaccess配置文件的内容
</Directory>
<Directory "/var/www/html"> 设置Apache根的访问权限
Options Indexes FollowSymLinks                      FollowSymLinks允许符号链接，Indexes无首页时罗列出目录下文件
AllowOverride None
Order allow,deny            先执行Allow访问规则，在执行Deny访问规则
Allow from all                 allow访问规则，允许所有链接
</Directory>
<IfModule mod_userdir.c>
UserDir disabled
</IfModule>
DirectoryIndex index.html index.html.var           网站首页文件名称定义
AccessFileName .htaccess            访问控制文件的名称
<Files ~ "^\.ht">         关于.ht开头文件的权限控制
Order allow,deny
Deny from all
Satisfy All            访问.ht开头文件需要满足两种访问控制都允许
</Files>
TypesConfig /etc/mime.types            MIME对应格式的配置文件的存放位置
DefaultType text/plain                        默认的MIME文件类型为纯文本或HTML文件
<IfModule mod_mime_magic.c>    当mod_mime_magic.c模块被加载时，指定magic信息码配置文件的存放位置
MIMEMagicFile conf/magic
</IfModule>
HostnameLookups Off          关闭记录访问web客户的hostname功能，只记录IP
ErrorLog logs/error_log        错误日志存放位置
LogLevel warn                        记录日志的等级
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i

\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %b" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent

以上为记录日志的四种格式
CustomLog logs/access_log combined             访问日志的纪录格式为combined（混合型），并指定访问日志存放位置
ServerSignature On                             apache自己产生的页面中使用apache服务器版本的签名
Alias /icons/ "/var/www/icons/"   定义别名/icons/
<Directory "/var/www/icons">
Options Indexes MultiViews FollowSymLinks              MultiViews 使用内容协商来决定被发送的网页的性质
AllowOverride None
Order allow,deny
Allow from all
</Directory>
<IfModule mod_dav_fs.c>   DAV加锁数据库文件的存放位置
DAVLockDB /var/lib/dav/lockdb
</IfModule>
ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"  设置CGI目录的访问别名
<Directory "/var/www/cgi-bin">
AllowOverride None
Options None
Order allow,deny
Allow from all
</Directory>
IndexOptions FancyIndexing VersionSort NameWidth=* HTMLTable       未找到首页文件时生存目录列表的方式，FancyIndexing 对每种类型的文件前加上一个小图标以示区别，VersionSort 对同一个软件的多个版本进行排序，NameWidth=* 文件名字段自动适应当前目录下的最长文件名。生成小图标的时候使用AddIcon选项。
Charset=UTF-8           字符编码
AddIconByEncoding (CMP,/icons/compressed.gif) x-compress x-gzip
AddIconByType (TXT,/icons/text.gif) text/*
AddIconByType (IMG,/icons/image2.gif) image/*
AddIconByType (SND,/icons/sound2.gif) audio/*
AddIconByType (VID,/icons/movie.gif) video/*
AddIcon /icons/binary.gif .bin .exe
AddIcon /icons/binhex.gif .hqx
AddIcon /icons/tar.gif .tar
AddIcon /icons/world2.gif .wrl .wrl.gz .vrml .vrm .iv
AddIcon /icons/compressed.gif .Z .z .tgz .gz .zip
AddIcon /icons/a.gif .ps .ai .eps
AddIcon /icons/layout.gif .html .shtml .htm .pdf
AddIcon /icons/text.gif .txt
AddIcon /icons/c.gif .c
AddIcon /icons/p.gif .pl .py
AddIcon /icons/f.gif .for
AddIcon /icons/dvi.gif .dvi
AddIcon /icons/uuencoded.gif .uu
AddIcon /icons/script.gif .conf .sh .shar .csh .ksh .tcl
AddIcon /icons/tex.gif .tex
AddIcon /icons/bomb.gif /core
AddIcon /icons/back.gif ..
AddIcon /icons/hand.right.gif README
AddIcon /icons/folder.gif ^^DIRECTORY^^
AddIcon /icons/blank.gif ^^BLANKICON^^
DefaultIcon /icons/unknown.gif      位置类型文件使用此图像作为图标
ReadmeName README.html          当服务器自动列出目录列表时，在所生成的页面之后显示readme.html的内容
HeaderName HEADER.html            当服务器自动列出目录列表时，在所生成的页面之前显示header.html的内容
AddLanguage ca .ca
AddLanguage cs .cz .cs
AddLanguage da .dk
AddLanguage de .de
AddLanguage el .el
AddLanguage en .en
AddLanguage eo .eo
AddLanguage es .es
AddLanguage et .et
AddLanguage fr .fr
AddLanguage he .he
AddLanguage hr .hr
AddLanguage it .it
AddLanguage ja .ja
AddLanguage ko .ko
AddLanguage ltz .ltz
AddLanguage nl .nl
AddLanguage nn .nn
AddLanguage no .no
AddLanguage pl .po
AddLanguage pt .pt
AddLanguage pt-BR .pt-br
AddLanguage ru .ru
AddLanguage sv .sv
AddLanguage zh-CN .zh-cn
AddLanguage zh-TW .zh-tw
以上为设置网页内容的语言种类
LanguagePriority en ca cs da de el eo es et fr he hr it ja ko ltz nl nn

no pl pt pt-BR ru sv zh-CN zh-TW 生效的先后顺序
ForceLanguagePriority Prefer Fallback
Prefer 当有多种语言可以匹配时，使用LanguagePriority 列表的第一项
Fallback 当没有语言可以匹配时，使用LanguagePriority 列表的第一项
AddDefaultCharset UTF-8   设置默认字符集
AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz
AddType application/x-x509-ca-cert .crt
AddType application/x-pkcs7-crl    .crl
以上为添加一些mime类型
AddHandler type-map var 设置apcche对某些扩展名的处理方式
AddType text/html .shtml
AddOutputFilter INCLUDES .shtml  使用过滤器执行SSI
Alias /error/ "/var/www/error/"  定义错误页面别名
<IfModule mod_negotiation.c>
<IfModule mod_include.c>
<Directory "/var/www/error"> 错误页的权限定义
AllowOverride None
Options IncludesNoExec
AddOutputFilter Includes html
AddHandler type-map var
Order allow,deny
Allow from all
LanguagePriority en es de fr
ForceLanguagePriority Prefer Fallback
</Directory>
</IfModule>
</IfModule>
以下为设置浏览器匹配
BrowserMatch "Mozilla/2" nokeepalive
BrowserMatch "MSIE 4\.0b2;" nokeepalive downgrade-1.0 force-response-

1.0
BrowserMatch "RealPlayer 4\.0" force-response-1.0
BrowserMatch "Java/1\.0" force-response-1.0
BrowserMatch "JDK/1\.0" force-response-1.0
BrowserMatch "Microsoft Data Access Internet Publishing Provider"

redirect-carefully
BrowserMatch "MS FrontPage" redirect-carefully
BrowserMatch "^WebDrive" redirect-carefully
BrowserMatch "^WebDAVFS/1.[0123]" redirect-carefully
BrowserMatch "^gnome-vfs/1.0" redirect-carefully
BrowserMatch "^XML Spy" redirect-carefully
BrowserMatch "^Dreamweaver-WebDAV-SCM1" redirect-carefully
```