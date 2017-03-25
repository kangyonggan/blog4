---
title: SSH登录时指定密码
date: 2017-03-25 19:18:04
categories: 综合
tags:
- Linux
- SSH
---

## 下载`sshpass`
[http://sourceforge.net/projects/sshpass/](http://sourceforge.net/projects/sshpass/)

## 安装
```
[root@localhost ~]# tar -zxvf sshpass-1.06.tar.gz -C /root/install/
[root@localhost ~]# cd /root/install/sshpass-1.06
[root@localhost sshpass-1.06]# ./configure 
[root@localhost sshpass-1.06]# make
[root@localhost sshpass-1.06]# make install
```

## 使用
```
[root@localhost ~]# sshpass -p 123456 ssh root@121.40.66.176
```

