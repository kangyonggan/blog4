---
title: MySQL5.6主从复制的配置
date: 2017-03-25 17:59:52
categories: 数据库
tags:
- MySQL
---

### 实验环境
1. 主库机器IP: `10.10.10.248`
2. 从库机器IP: `10.10.10.166`
3. 数据库版本: `Mysql 5.6`
4. 操作系统: `Ubuntu 14.04`

<!-- more -->

### 配置主库
#### 修改`/etc/mysql/my.cfg`配置，在`[mysqld]`下面添加：

```
# 启动二进制文件
log-bin=mysql-bin

# 服务器ID
server-id=1

# 要同步的数据库(多个可重复指定)
#binlog-do-db=api
#binlog-do-db=blog

# 不要同步的数据库
#replicate-ignore-db=test
```

保存后重启mysql: 

```
sudo /etc/init.d/mysql restart
```

#### 创建一个用户，并授权给从服务器
登录mysql

```
mysql -uroot -p
```

创建用户`kyg`并授权给从服务器:  

```
grant replication slave on *.* to 'kyg'@'10.10.10.166' identified by 'kyg';
```

![master](http://cdn.kangyonggan.com/upload/20170101145536544.png)

最后别忘了刷新权限:

```
flush privileges;
```

#### 查看主库状态
```
show master status;
```

记录下`File`和`Position`的值，后面配置从库时会用到

![master-status](http://cdn.kangyonggan.com/upload/20170101145208095.png)

### 配置从库
#### 修改`/etc/mysql/my.cfg`配置，在`[mysqld]`下面添加：
```
# 从库ID
server-id=2
```

保存后重启mysql:

```
sudo /etc/init.d/mysql restart
```

### 登录mysql，并配置相关参数:

```
change master to  
master_host = '10.10.10.248',  
master_user = 'kyg',  
master_password = 'kyg',  
master_log_file = 'mysql-bin.000004',  
master_log_pos = 120;  
```

查看从库状态

```
show slave status \G;
```

会发现从库还没开启复制

![slave-status1](http://cdn.kangyonggan.com/upload/20170101145456162.png)

这时候就需要开启从库的复制功能`start slave;`

![slave-status2](http://cdn.kangyonggan.com/upload/20170101145208097.png)

上面的截图再往下滚动， 可能会看见报错:

```
error connecting to master 'kyg@10.10.10.248:3306' - retry-time: 60  retries: 7
```

这是由于主库配置了`bind-address:127.0.0.1`, 为了简单，我直接把这一行配置注释了(有安全隐患)

停止从库的复制功能:

```
stop slave
```

### 测试
### 在主库创建一个数据库`blog`
然后在从库中查看所有数据库

```
show databases;
```

![show-db](http://cdn.kangyonggan.com/upload/20170101145208096.png)

会发现从库从主库中复制了一个数据库, 实验已经成功，其他CRUD请自行实验


