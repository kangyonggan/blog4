---
title: ubuntu16.0.4启动zookeeper报错
date: 2017-03-25 17:26:17
categories: 综合
tags:
- Linux
- Zookeeper
---

### 报错信息
```
bin/zkServer.sh: 81: /home/kyg/install/server1/zookeeper-3.4.9/bin/zkEnv.sh: Syntax error: "(" unexpected (expecting "fi")
```

### 解决方案
```
ls -l /bin/sh
lrwxrwxrwx 1 root root 4 12月 24 17:22 /bin/sh -> dash
ls -l /bin/sh
lrwxrwxrwx 1 root root 4 12月 24 20:01 /bin/sh -> bash
```