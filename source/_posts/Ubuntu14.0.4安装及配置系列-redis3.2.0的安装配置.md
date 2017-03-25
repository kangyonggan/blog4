---
title: Ubuntu14.0.4安装及配置系列-redis3.2.0的安装配置
date: 2017-03-25 16:41:14
categories: 系统运维
tags: 
- Linux
- Redis
---

### 下载[redis3.2](https://redis.io/)到本地

### 上传本地redis到Ubuntu服务器
```
$ scp redis-3.2.0.tar.gz root@121.40.66.176:/root/soft/
```

### 解压redis到指定目录
```
# tar -zxvf redis-3.2.0.tar.gz -C /root/install/
```

### 安装
```
make & make install
```

<!-- more -->

### 配置
在`/root/install/redis-3.2.0/redis.conf`后面追加下面的配置:

```
requirepass 123456
```

### 启动
```
# redis-server redis.conf &
```

### 测试

```
# redis-cli -a 123456
> ping
```

输出:`PONG`

### 停止
```
# redis-cli -a 123456 shutdown
```

### 清空所有缓存
```
# redis-cli -a 123456 KEYS "*" | xargs redis-cli -a 123456 DEL
```

