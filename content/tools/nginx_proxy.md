---
title: "Nginx 反向代理"
date: 2021-01-27T13:34:41+08:00
draft: false
---

# 将静态文件交给nginx代理

    配置文件位置 /etc/nginx/conf.d/default.conf


~~~
server {
    listen 80;
    server_name www.moyrn.com;
    location / {
        root /usr/share/nginx/webmonitor;
        index index.html;
    }
}
~~~

#### 通过不同的域名，将80端口的消息转发到其他端口

~~~
server {
    listen  80;
    server_name go.moyrn.com;
    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}

server {
    listen 80;
    server_name moyrn.com;
    location / {
        proxy_pass http://127.0.0.1:8086;
    }
}
server {
    listen 80;
    server_name www.moyrn.com;
    location / {
        proxy_pass http://127.0.0.1:8086;
    }
}
~~~