---
layout: post
title: PHP_CURL 使用代理访问服务器
published: true
author: xiemx
comments: true
date: 2016-07-06 03:07:24
tags: [ php ]
categories:
    - php
---
```php
#使用代码时请加上php文件的括号
function curl_string ($url,$user_agent,$proxy){

       $ch = curl_init();
       curl_setopt ($ch, CURLOPT_PROXY, $proxy);
       curl_setopt ($ch, CURLOPT_URL, $url);
       curl_setopt ($ch, CURLOPT_USERAGENT, $user_agent);
       curl_setopt ($ch, CURLOPT_HEADER, 1);
       curl_setopt ($ch, CURLOPT_RETURNTRANSFER, 1);
       curl_setopt ($ch, CURLOPT_FOLLOWLOCATION, 1);
       curl_setopt ($ch, CURLOPT_TIMEOUT, 120);
       $result = curl_exec ($ch);
       curl_close($ch);
       return $result;

}

$url_page = "http://www.google.com";
$user_agent = "Mozilla/4.0";
$proxy = "http://192.11.222.124:8000";
$string = curl_string($url_page,$user_agent,$proxy);
echo $string;

```