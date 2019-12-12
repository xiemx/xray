---
layout: post
title: zabbix微信报警脚本
published: true
author: xiemx
comments: true
date: 2016-06-29 10:06:55
tags: [ ]
categories:
    - zabbix
#permalink: '/2016/06/29/zabbix-alert-to-wechat-script'
---

```python
#!/usr/bin/python

__author__ = 'xiemx'

import sys
import json,requests
import os
import logging

class Weixin(object):

    def get_token(self):
        CorpID = '-------4fa4'
        Secret = 'Aew6oxx-----------FaTClkjXlmw_zH'
        token_url = 'https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid={}&corpsecret={}'.format(CorpID, Secret)
        response = requests.get(token_url, verify=False).content
        p = json.loads(response)
        token = p['access_token']
        return token

    def send_msg(self, user_id, msg):
        token = self.get_token()
        url = 'https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token={}'.format(token)
        send_content={
            "touser": user_id,
            "msgtype": "text",
            "agentid": "2",
            "text": {
               "content": msg
            },
            "safe":"0"
            }
        
        p = requests.post(url, verify=False, data=json.dumps(send_content))
        print p.content
        logging.debug("weixin send success")

if __name__ == "__main__":
    user_id = sys.argv[1]
    msg = sys.argv[2] + '\n' + sys.argv[3]
    weixin = Weixin()
    weixin.send_msg(user_id, msg)
```