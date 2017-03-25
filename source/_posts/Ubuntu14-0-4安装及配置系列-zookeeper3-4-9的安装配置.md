---
title: Ubuntu14.0.4安装及配置系列-zookeeper3.4.9的安装配置
date: 2017-03-25 16:48:55
categories: 系统运维
tags: 
- Linux
- Zookeeper
---

### 下载[zookeeper3.4.9](http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.9/)到本地

### 上传本地zookeeper到Ubuntu服务器
```
$ scp zookeeper-3.4.9.tar.gz root@121.40.66.176:/root/soft/
```

### 解压zookeeper到指定目录

```
mkdir /root/install/server1
# tar zxvf zookeeper-3.4.9.tar.gz -C /root/install/server1/
```

<!-- more -->

### 配置（集群）
在`server1`目录下创建两个目录，分别是`data`和`logs`

```
cd /root/install/server1/
mkdir data
mkdir logs
```

此时，server1目录下有三个文件

```
root@iZ23ldh8kudZ:~/install/server1# pwd
/root/install/server1
root@iZ23ldh8kudZ:~/install/server1# ll
total 20
drwxr-xr-x  5 root root 4096 Dec 17 15:43 ./
drwxr-xr-x  8 root root 4096 Dec 17 15:39 ../
drwxr-xr-x  2 root root 4096 Dec 17 15:43 data/
drwxr-xr-x  2 root root 4096 Dec 17 15:43 logs/
drwxr-xr-x 10 1001 1001 4096 Aug 23 15:42 zookeeper-3.4.9/
```

在`data`目录下创建文件`myid`,其内容为`1`

```
vi /data/myid
# 内容为1
```

复制配置文件`zookeeper-3.4.9/conf/zoo_sample.cfg`到`zookeeper-3.4.9/conf/zoo.cfg`, 并修改其内容:

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/install/server1/data
dataLogsDir=/root/install/server1/logs
clientPort=2181

server.1=121.40.66.176:2888:3888
server.2=139.196.28.125:2888:3888
```

> 每台服务器都要有类似的配置

### 环境变量

在`/etc/profile`文件最后追加:

```
# zookeeper environment
export PATH=$PATH:/root/install/server1/zookeeper-3.4.9/bin
```

立即生效

```
# source /etc/profile
```

### 启动

```
# zkServer.sh start
```

### 查看是否启动

```
root@iZ23ldh8kudZ:~/install/server1/zookeeper-3.4.9# jps
7096 QuorumPeerMain
7114 Jps
```

看到`QuorumPeerMain`就说明启动成功了

### 停止

```
# zkServer.sh stop
```


