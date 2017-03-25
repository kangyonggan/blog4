---
title: CentOS7下安装Nodejs
date: 2017-03-25 20:58:36
categories: 综合
tags:
- Linux
---

## 下载源码
在[https://nodejs.org/en/download/](https://nodejs.org/en/download/)下载最新的Nodejs版本，本文以v0.10.24为例:

```
cd /usr/local/src/
wget http://nodejs.org/dist/v0.10.24/node-v0.10.24.tar.gz
```

## 解压源码
```
tar zxvf node-v0.10.24.tar.gz
```

## 编译安装
```
cd node-v0.10.24
./configure --prefix=/usr/local/node/0.10.24
make
make install
```

<!-- more -->

## 配置NODE_HOME，进入profile编辑环境变量
```
#set for nodejs
export NODE_HOME=/usr/local/node/0.10.24
export PATH=$NODE_HOME/bin:$PATH
```

## 编译/etc/profile 使配置生效
```
source /etc/profile
```

## 验证是否安装配置成功
```
node -v
```

输出 v0.10.24 表示配置成功  
npm模块安装路径

```
/usr/local/node/0.10.24/lib/node_modules/
```

