---
title: ubuntu下mysql数据库恢复
date: 2017-03-25 17:45:07
categories: 综合
tags:
- Linux
- MySQL
---

前端时间我在玩主从库的时候， 一不小心把mysql数据库玩坏了， 连服务都启动不了了， 经过一番修复后还是不行， 可怜我的博客都在库里, 不过我却在庆幸， 因为这叫置之死地而后生，我又能学到牛逼技术了...

### 备份数据
mysql数据是放在/var/lib/mysql下面的，需要备份的内容有：

- ibdata1
- performance_schema
- blog文件夹 （对应数据库blog）
- 其他你需要恢复的数据库

<!-- more -->

### 重装mysql
重装教程网上一堆， 这里就不再赘述。

### 恢复
把备份的几个文件， 替换回去， 然后重启mysql， 这时候会报错， 查看mysql日志后发现是权限不足导致。

原因是：备份与恢复数据的用户一般都需要比较高的权限才能操作， 比如root，所以替换后的文件的所有者是root， 而不再是mysql， 因此需要改变文件拥有者和赋权：

```
cd /var/lib/mysql
chown mysql *

chmod 700 blog;
chmod 700 ibdata1;
chmod 700 performance_schema;
```

必要时还需要删除这两个日志文件`ib_logfile0`和`ib_logfile1`：

```
rm ib_logfile*
```

重启， 不出意外应该是可以登录mysql了， `show databases`能看到已经恢复的blog数据库了， `use blog;show tables`能看见article表了。

但是，在查询article表的时候，又会报权限不足的错， 道理是相同的， 我们去`/var/lib/mysql/blog`目录下，重新改变拥有者:

```
cd /var/lib/mysql/blog
chown mysql *
```

重启，验证， 成功！



