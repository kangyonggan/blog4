---
title: Ubuntu14.04从命令行启动
date: 2017-03-25 17:55:10
categories: 综合
tags:
- Linux
---

### 修改配置 /etc/default/grub 
- 注释此行：`#GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`
- `GRUB_CMDLINE_LINUX=""` 改为 `GRUB_CMDLINE_LINUX="text"`
- `#GRUB_TERMINAL=console` 的注释干掉

### 更新配置
```
update-grub
```

### 重启
不出意外会从命令行启动， 意外会发生在ubuntu16.04上
