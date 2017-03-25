---
title: ubuntu配置DNS服务器
date: 2017-03-25 18:45:10
categories: 综合
tags:
- Linux
---

## 实验目的
在ubuntu 14.04上搭建一个自己的dns服务器，并实现重新解析baidu.com到自定义服务器。

## 安装并配置

```
kyg@kyg-Latitude-5450:~$ sudo apt-get bind9
```

<!-- more -->

检查是否安装成功

```
kyg@kyg-Latitude-5450:~$ cd /etc/bind/
kyg@kyg-Latitude-5450:/etc/bind$ ll
total 68
drwxr-sr-x   2 root bind  4096  2月 19 12:25 ./
drwxr-xr-x 129 root root 12288  2月 19 12:25 ../
-rw-r--r--   1 root root  2389  2月 16 00:33 bind.keys
-rw-r--r--   1 root root   237  2月 16 00:33 db.0
-rw-r--r--   1 root root   271  2月 16 00:33 db.127
-rw-r--r--   1 root root   237  2月 16 00:33 db.255
-rw-r--r--   1 root root   353  2月 16 00:33 db.empty
-rw-r--r--   1 root root   270  2月 16 00:33 db.local
-rw-r--r--   1 root root  3048  2月 16 00:33 db.root
-rw-r--r--   1 root bind   463  2月 16 00:33 named.conf
-rw-r--r--   1 root bind   490  2月 16 00:33 named.conf.default-zones
-rw-r--r--   1 root bind   165  2月 16 00:33 named.conf.local
-rw-r--r--   1 root bind   890  2月 19 12:25 named.conf.options
-rw-r-----   1 bind bind    77  2月 19 12:25 rndc.key
-rw-r--r--   1 root root  1317  2月 16 00:33 zones.rfc1918
```

在实验之前先ping一下百度，好做个对比

```
kyg@kyg-Latitude-5450:~$ ping baidu.com
PING baidu.com (123.125.114.144) 56(84) bytes of data.
64 bytes from 123.125.114.144: icmp_seq=1 ttl=54 time=33.4 ms
64 bytes from 123.125.114.144: icmp_seq=2 ttl=54 time=42.4 ms
64 bytes from 123.125.114.144: icmp_seq=3 ttl=54 time=37.3 ms
```

可以看到dns把baidu.com解析成的ip是123.125.114.144  
现在我就搭建一个简单的dns，让自己的dns把baidu.com解析成192.168.2.112  
这是自己的服务器，然后就可以把xxx展现给用户了。

配置dns：

```
kyg@kyg-Latitude-5450:~$ sudo vim /etc/bind/named.conf.local
添加如下内容:

zone "baidu.com" { type master; file "/etc/bind/db.baidu.com"; };
```

修改db的配置文件:

```
kyg@kyg-Latitude-5450:~$ cd /etc/bind/
kyg@kyg-Latitude-5450:/etc/bind# sudo cp db.local db.baidu.com
kyg@kyg-Latitude-5450:/etc/bind# vi db.baidu.com

内容如下：

;
; BIND data file for local loopback interface
;
$TTL604800
@INSOAlocalhost. root.localhost. (
o      2; Serial
o 604800; Refresh
o  86400; Retry
o2419200; Expire
o 604800 ); Negative Cache TTL
;
@INNSlocalhost.
@INA192.168.2.112
@INAAAA::1
```

最后重启服务！

```
kyg@kyg-Latitude-5450:/etc/bind# sudo /etc/init.d/bind9 restart
 * Stopping domain name service... bind9                                                waiting for pid 5141 to die
                                                                                 [ OK ]
 * Starting domain name service... bind9  
```

如果启动失败，可以运行`named -g`查看错误原因.

## 测试
```
kyg@kyg-Latitude-5450:/etc/bind$ ping baidu.com
PING baidu.com (192.168.2.112) 56(84) bytes of data.
64 bytes from 192.168.2.112: icmp_seq=1 ttl=63 time=4.73 ms
64 bytes from 192.168.2.112: icmp_seq=2 ttl=63 time=2.12 ms
64 bytes from 192.168.2.112: icmp_seq=3 ttl=63 time=2.22 ms
64 bytes from 192.168.2.112: icmp_seq=4 ttl=63 time=5.15 ms
64 bytes from 192.168.2.112: icmp_seq=5 ttl=63 time=18.8 ms
64 bytes from 192.168.2.112: icmp_seq=6 ttl=63 time=2.11 ms
64 bytes from 192.168.2.112: icmp_seq=7 ttl=63 time=2.81 ms
64 bytes from 192.168.2.112: icmp_seq=8 ttl=63 time=2.79 ms
```

## 感悟
其实在我们没有安装DNS服务之前，可以将/etc/hosts文件比作一个DNS服务配置文件，因为它实现和DNS类似。

之所以会独立出DNS服务，是因为因特网主机多，如果每个主机都靠/etc/hosts文件来维护主机名到ip的映射，那么工作量非常大，对本地更新、网络资源占用都很浪费，所以出现了DNS。




