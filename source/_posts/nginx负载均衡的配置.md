---
title: nginx负载均衡的配置
date: 2017-03-25 18:07:36
categories: 综合
tags:
- Nginx
---

### 实验环境
1. 机器1的ip:10.10.10.248, tomcat端口:8088
2. 机器2的ip:10.10.10.166, tomcat端口:8088
3. nginx所在机器ip:10.10.10.248, 监听端口80

<!-- more -->

### 启动两个tomcat，配置nginx
我使用的是jenkins启动的tomcat，一键部署成功, 效果如下:

![tomcat-166.png](/uploads/20170101200131662.png)

![tomcat-248.png](/uploads/20170101200131669.png)

我的nginx是部署在248服务器上，监听的是80端口， 现在想做的就是:  
在访问http://10.10.10.248:80的时候，nginx把所有的请求均等转发到10.10.10.248:8088和10.10.10.166:8088

nginx配置`/etc/nginx/sites-available/default`如下:

```
upstream kyg.com {
       server  10.10.10.248:8088;
       server  10.10.10.166:8088;
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

修改nginx配置后需要重新加载配置`sudo nginx -s reload`

### 查看日志，分析结果
1. 分别在两台服务器上`tail -f /home/kyg/logs/blog/all.log`  
2. 然后访问http://10.10.10.248  
3. 观察哪台服务器会刷日志
4. 再次访问http://10.10.10.248
5. 观察哪台服务器会刷日志
6. 重复访问观察...

![log](/uploads/20170101205501760.png)

发现nginx会把所有的请求均等的（发给你一次发给我一次）转发到两台服务器, 当然你也可以配置权重，让某台服务分担的压力小一点，或者动态负载均衡等。

### kill其中一个tomcat
我现在把166服务器kill了，在访问并观察日志, 结果：
  
1. 网站仍然可以正常访问
2. 所有的访问全部转发到248服务器

### 分布式服务需要解决的几个问题
1. 会话共享，请参考我的另一篇博客[shiro集成redis实现session集群共享](http://kangyonggan.com#article/35)
2. 文件共享, 请参考我的另一篇博客[ubuntu下配置ftp服务器](http://kangyonggan.com#article/22)


