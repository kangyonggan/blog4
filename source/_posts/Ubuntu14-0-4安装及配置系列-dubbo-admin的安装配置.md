---
title: Ubuntu14.0.4安装及配置系列-dubbo-admin的安装配置
date: 2017-03-25 17:07:26
categories: 系统运维
tags: 
- Linux
- Dubbo
---

### 上传本地dubbo-admin到Ubuntu服务器
```
$ scp dubbo-admin.war root@121.40.66.176:/root/soft/
```

### 解压dubbo-monitor到tomcat/webapps/ROOT/目录下
```
# unzip dubbo-admin.war -d /root/install/apache-tomcat-8.5.6/webapps/ROOT/
```

<!-- more -->

### 配置
修改`webapps/ROOT/WEB-INF/dubbo.properties`中`dubbo.registry.address`的值：

```
dubbo.registry.address=zookeeper://121.40.66.176:2181?backup=139.196.28.125:2181
```

修改tomcat的server.xml：

```
<Connector port="9090" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" ></Connector>
```

### 启动
```
# sh bin/startup.sh
```

### 停止
```
# sh bin/shutdown.sh
```

### 访问
http://localhost:9090/  

- 用户名: 
- 密码:

