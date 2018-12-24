---
title: MAC使用nginx分发80至8080端口
date: 2018-12-21 19:13:03
tags: [mac,nginx]
categories: Mac
---

# 1、使用背景
由于项目必须要启动80端口，但是mac系统中非root用户无法直接使用1024以下的端口

# 2、释放apache的80端口
由于Mac OS是自带Apache服务的，它本身占用了80端口，首先你需要将Apache的监听端口改为其他端口或者将其直接卸载，我选用的是将其端口改为8011
```bash
sudo vim /etc/apache2/httpd.conf
```
Listen 8011

改动后，重启生效
```bash
sudo /usr/sbin/apachectl restart
```
到这里，你已经释放了80端口

# 3、使用Nginx分发80端口到8080端口
0. 安装brew

见官网：[https://brew.sh/index_zh-cn.html](https://brew.sh/index_zh-cn.html)

1. 使用Homebrew安装库
```bash
brew search nginx
brew install nginx
```

2. 安装好了后，修改配置
```bash
sudo vim /usr/local/etc/nginx/nginx.conf
```

```bash
    server {
        listen       80;
        server_name  localhost l.sogou.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location ~* ^/h5/{
                proxy_pass http://127.0.0.1:8091;
        }

        location ~* ^/weixin/{
            proxy_pass http://127.0.0.1:8093;
        }

        location ~* ^/api/{
            proxy_pass http://127.0.0.1:8087;
        }

        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://127.0.0.1:8080;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page             /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```
server下的结点：
listen：监听80端口
server_name：转发到哪个地址
proxy_pass：代理到哪个地址


3. Nginx开机启动

你需要了解的就是plist文件。plist就是property list format的意思，是苹果用来保存应用数据的格式，其实就是个xml。
可以在/usr/local/opt/nginx 下找到nginx对应的plist文件，比如在作者电脑上是 homebrew.mxcl.nginx.plist 。

需要把这个文件复制到 /Library/LaunchDaemons 下，系统启动时启动。
也可以复制到 /Library/LaunchAgents下，在用户登录时启动。
接着执行launchctl load -w，如下：

```bash
sudo cp /usr/local/opt/nginx/*.plist /Library/LaunchDaemons

sudo launchctl load -w /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

最后，重启你的机器，你会发现nginx在80端口启动了，试着通过http://localhost直接访问

4. 修改配置 重启生效
```bash
sudo vim /usr/local/etc/nginx/nginx.conf

cd /usr/local/opt/nginx/bin/
sudo ./nginx -s reload
```
