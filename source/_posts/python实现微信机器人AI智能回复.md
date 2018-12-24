---
title: 'python实现微信机器人AI智能回复'
date: 2018-12-24 19:08:06
tags: [python,wechat]
categories: python
---

# 注册图灵机器人网站的账号
链接：http://www.tuling123.com/
你可以获取自己的图灵机器人apikey

# 代码
```bash
import itchat
import requests

def get_response(_info):
    print(_info)
    api_url = 'http://www.tuling123.com/openapi/api'
    data = {
        'key': 'your_tuling_apikey', # 上一步注册的apikey
        'info': _info,
        'userid': 'robot',          # 随意填
    }
    r = requests.post(api_url, data=data).json()
    print(r.get('text'))
    return r


@itchat.msg_register(itchat.content.TEXT)
def text_reply(msg):
    return r"[Bao]" + get_response(msg["Text"])["text"]

if __name__ == '__main__':
    print("start")
    itchat.auto_login(hotReload=True)
    itchat.run()
```
