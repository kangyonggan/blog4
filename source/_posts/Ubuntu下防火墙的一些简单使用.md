---
title: Ubuntu下防火墙的一些简单使用
date: 2017-03-25 17:44:00
categories: 系统运维
tags:
- Linux
---

### 摘要
公司的很多项目都是放在Linux机器上的，因此安装防火墙还是很有必要的...

### 安装防火墙
```
apt-get install firewalld
```

<!-- more -->

### 查看开放的端口
```
firewall-cmd --list-all
```

### 把一个端口开放
```
firewall-cmd --permanent --add-port=8080/tcp
```

### 端口开放之后要重新加载
```
firewall-cmd --reload
```

### 移除一个开放的端口
```
firewall-cmd --permanent --remove-port=8080/tcp
```
