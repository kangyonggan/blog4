---
title: linux下杀死一个进程的常用手段
date: 2017-03-25 17:57:03
categories: 综合
tags:
- Linux
---

```
ps -ef | grep redis-server | cut -c 9-15 | xargs kill -9
```