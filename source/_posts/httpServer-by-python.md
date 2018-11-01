---
title: 用python写的一个简易的Http服务器
date: 2018-11-01 11:37:30
tags: [http,python]
categories: [python]
---

昨天在看《Http权威指南》的时候，看到里面用Perl实现了一个最简单的Http的服务器。
于是我参考着里面的逻辑写了一个python版本的。

1. 创建服务器套接字（socket），把地址绑定到套接字上，并监听连接
2. 服务器无限循环，接受客户端连接
3. 客户端连接进来后，读取客户端发送的消息，并且打印Http请求报文
4. 返回Http的响应报文

运行以下的python脚本后，使用浏览器访问http://localhost:8080/即可。

```python
#!/usr/bin/env python
from socket import *

HOST = ''
PORT = 8080
BUFSIZE = 1024
ADDR = (HOST,PORT)

tcpSerSock = socket(AF_INET,SOCK_STREAM)
tcpSerSock.bind(ADDR)
tcpSerSock.listen(5)

responseStr = '''
HTTP/1.0 200 OK
Connection:close
Content-type;text:plain

Hi there!
'''

while True:
    print 'waiting for connection ...'
    tcpCliSock , addr = tcpSerSock.accept()
    print '...connected from:',addr

    while True:
        data = tcpCliSock.recv(BUFSIZE)
        if not data:
            break
        print data
        tcpCliSock.send(responseStr)
        break
    tcpCliSock.close()

tcpSerSock.close()
```
