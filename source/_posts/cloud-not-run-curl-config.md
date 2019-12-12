---
layout: post
title: '错误Could not run curl-config: [Errno 2] No such file or directory解决'
published: true
author: xiemx
comments: true
date: 2016-05-27 05:05:22
tags: [ linux, debug ]
categories:
    - linux
---
```shell
root@06f95000953c:~/phantomjs# pip install pycurl
Downloading/unpacking pycurl
  Downloading pycurl-7.43.0.tar.gz (182kB): 182kB downloaded
  Running setup.py (path:/tmp/pip-build-XGZuU8/pycurl/setup.py) egg_info for package pycurl
    Traceback (most recent call last):
      File "", line 17, in 
      File "/tmp/pip-build-XGZuU8/pycurl/setup.py", line 823, in 
        ext = get_extension(sys.argv, split_extension_source=split_extension_source)
      File "/tmp/pip-build-XGZuU8/pycurl/setup.py", line 497, in get_extension
        ext_config = ExtensionConfiguration(argv)
      File "/tmp/pip-build-XGZuU8/pycurl/setup.py", line 71, in __init__
        self.configure()
      File "/tmp/pip-build-XGZuU8/pycurl/setup.py", line 107, in configure_unix
        raise ConfigurationError(msg)
    __main__.ConfigurationError: Could not run curl-config: [Errno 2] No such file or directory
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):

  File "", line 17, in 

  File "/tmp/pip-build-XGZuU8/pycurl/setup.py", line 823, in 

    ext = get_extension(sys.argv, split_extension_source=split_extension_source)

  File "/tmp/pip-build-XGZuU8/pycurl/setup.py", line 497, in get_extension

    ext_config = ExtensionConfiguration(argv)

  File "/tmp/pip-build-XGZuU8/pycurl/setup.py", line 71, in __init__

    self.configure()

  File "/tmp/pip-build-XGZuU8/pycurl/setup.py", line 107, in configure_unix

    raise ConfigurationError(msg)

__main__.ConfigurationError: Could not run curl-config: [Errno 2] No such file or directory

----------------------------------------
Cleaning up...
Command python setup.py egg_info failed with error code 1 in /tmp/pip-build-XGZuU8/pycurl
Storing debug log for failure in /root/.pip/pip.log

root@06f95000953c:~/phantomjs# apt-get install libcurl4-openssl-dev
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Suggested packages:
  libcurl4-doc libcurl3-dbg libidn11-dev libkrb5-dev libldap2-dev librtmp-dev
The following NEW packages will be installed:
  libcurl4-openssl-dev
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 253 kB of archives.
After this operation, 1262 kB of additional disk space will be used.
Get:1 http://mirrors.163.com/ubuntu/ vivid-security/main libcurl4-openssl-dev amd64 7.38.0-3ubuntu2.2 [253 kB]
Fetched 253 kB in 5s (47.1 kB/s)            
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libcurl4-openssl-dev:amd64.
(Reading database ... 25480 files and directories currently installed.)
Preparing to unpack .../libcurl4-openssl-dev_7.38.0-3ubuntu2.2_amd64.deb ...
Unpacking libcurl4-openssl-dev:amd64 (7.38.0-3ubuntu2.2) ...
Setting up libcurl4-openssl-dev:amd64 (7.38.0-3ubuntu2.2) ...
root@06f95000953c:~/phantomjs# 
root@06f95000953c:~/phantomjs# 
root@06f95000953c:~/phantomjs# curl
curl         curl-config  
root@06f95000953c:~/phantomjs# 
```


安装libcurl4-openssl-dev此开发包即可
```
sudo apt-get install libcurl4-openssl-dev
```