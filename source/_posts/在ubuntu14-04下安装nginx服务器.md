---
title: 在ubuntu14.04下安装nginx服务器
date: 2017-03-25 17:57:50
categories: 综合
tags:
- Linux
- Nginx
---

### 下载安装
```
sudo apt-get install nginx
```

安装完成后打开浏览器输入`http://localhost`, 看到下图代表安装成功:

![nginx](/uploads/20170101125316443.png)

### 常用命令
1. 启动: `nginx -c /etc/nginx/nginx.conf`
2. 停止: `nginx -s stop`
3. 重新加载（配置）: `nginx -s reload`
4. 重新打开日志文件: `nginx -s reopen`

### 动静分离反向代理配置
修改`/etc/nginx/sites-available/default`配置:

```
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /usr/share/nginx/html;
        index index.html index.htm;

        # Make site accessible from http://localhost/
        server_name localhost;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
                proxy_pass http://localhost:8088;
        }

        # 设定访问静态文件直接读取不经过tomcat
        location ^~ /static/ {
                proxy_pass http://localhost:8088;
                root /WEB-INF/static/;
        }
}
```
