---
title: Ubuntu14.0.4安装及配置系列-dubbo-monitor2.5.3的安装配置
date: 2017-03-25 17:00:38
categories: 系统运维
tags:
- Linux
- Dubbo
---

### 下载[dubbo-monitor2.5.3](http://download.csdn.net/detail/liweifengwf/7864009)到本地

### 上传本地dubbo-monitor到Ubuntu服务器
```
$ scp dubbo-monitor-simple-2.5.3-assembly.tar.gz root@121.40.66.176:/root/soft/
```

### 解压dubbo-monitor到指定目录
```
# tar -zxvf dubbo-monitor-simple-2.5.3-assembly.tar.gz -C /root/install/
```

### 配置
修改`conf/dubbo.properties`中`dubbo.registry.address`的值：

```
dubbo.registry.address=zookeeper://121.40.66.176:2181?backup=139.196.28.125:2181
```

<!-- more -->

### 启动
```
# ./bin/start.sh
```

### 停止
```
# ./bin/stop.sh
```

### 重启
```
# ./bin/restart.sh
```

### 访问
http://localhost:8080/

