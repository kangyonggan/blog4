---
title: Ubuntu14.04盒盖不休眠
date: 2017-03-25 17:54:29
categories: 综合
tags:
- Linux
---

### 修改配置 /etc/systemd/logind.conf
然后将其中的：`#HandleLidSwitch=suspend` 改成： `HandleLidSwitch=ignore`

### 然后重启服务：
```
sudo restart systemd-logind
```
