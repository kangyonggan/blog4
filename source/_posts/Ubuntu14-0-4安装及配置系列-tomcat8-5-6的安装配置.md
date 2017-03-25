---
title: Ubuntu14.0.4安装及配置系列-tomcat8.5.6的安装配置
date: 2017-03-25 16:47:49
categories: 系统运维
tags: 
- Linux
- Tomcat
---

### 下载[tomcat8.5.6](http://tomcat.apache.org/download-80.cgi)到本地

### 上传本地tomcat到Ubuntu服务器
```
$ scp apache-tomcat-8.5.6.tar.gz root@121.40.66.176:/root/soft/
```

### 解压tomcat到指定目录
```
# tar -zxvf apache-tomcat-8.5.6.tar.gz -C /root/install/
```

### 启动
```
# sh startup.sh
```

### 停止
```
# sh shutdown.sh
```


