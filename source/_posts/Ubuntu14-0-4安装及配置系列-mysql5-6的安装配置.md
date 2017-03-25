---
title: Ubuntu14.0.4安装及配置系列-mysql5.6的安装配置
date: 2017-03-25 16:46:11
categories: 系统运维
tags: 
- Linux
- MySQL
---

### 安装
```
apt-get install mysql-server-5.6
```

### 启动
```
# /etc/init.d/mysql start
```

### 停止
```
# /etc/init.d/mysql stop
```

### 重启
```
# /etc/init.d/mysql restart
```

### 调整内存
修改`/etc/mysql/my.cnf`,在`[mysqld]`后面追加

```
performance_schema_max_table_instances=200
table_definition_cache=200
table_open_cache=128
```




