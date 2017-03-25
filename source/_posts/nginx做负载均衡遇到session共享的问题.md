---
title: nginx做负载均衡遇到session共享的问题
date: 2017-03-25 18:10:45
categories: 综合
tags:
- Nginx
---

### 常见的解决方案有:
#### 不使用session，换用cookie
- 简单方便对服务器无压力
- 如果客户端把cookie禁掉了的话，那么session就无法同步了
- cookie的安全性不高，虽然它已经加了密，但是还是可以伪造的

#### session存在数据库
- 会加大数据库的IO，增加数据库的负担
- 数据库读写速度较慢，不利于session的适时同步

<!-- more -->

#### session存在memcache或者Redis中
- 不会加大数据库的负担
- 并且安全性比用cookie大大的提高
- 把session放到内存里面，比从文件中读取要快很多
- 但偶尔会因网络较慢而出现掉线

#### 使用nginx中的ip_hash技术
- 能够将某个ip的请求定向到同一台后端
- nginx不是最前端的服务器的时候，就跪了，因为转发到nginx的ip是不变的
- nginx不是最后端的服务器的时候，也得跪

> 我个人的情况比较适合选择方案3+4

### 配置如下
```
upstream kyg.com {
        server  42.196.156.22:8088;
        server  42.196.156.22:18088;
        server  42.196.156.22:28088;
        ip_hash;
}

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
                proxy_pass http://kyg.com;
        }

        # 设定访问静态文件直接读取不经过tomcat
        location ^~ /static/ {
                proxy_pass http://kyg.com;
                root /WEB-INF/static/;
        }
}
```

就是在`upstream`下面加了`ip_hash;`配置

> 集群在设计之初最好设计成无状态的