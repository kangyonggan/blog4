---
title: mysql数据库定时备份
date: 2017-03-25 17:27:12
categories: 数据库
tags:
- Linux
- MySQL
---

### 创建备份脚本
创建`.bak.sh`, 内容为:

```
#! /bin/sh

today=`date +%Y%m%d`

# bak to local
mysqldump -uroot -p123456 blog > /home/kyg/bak/blog-bak-$today.sql

# bak to remote
scp /home/kyg/bak/blog-bak-$today.sql root@121.40.66.176:/root/bak/
```

<!-- more -->

> 其中，备份到远程时，用到了免密登录，请参考我的另一篇博客

### 定时执行
定时执行用的是linux下自带的`crontab`命令

`crontab -l` 查看任务

`crontab -e` 编辑任务

我设置的是，每天凌晨3点执行备份:

```
# m h  dom mon dow   command
0 3 * * * sh /home/kyg/.bak.sh
```

