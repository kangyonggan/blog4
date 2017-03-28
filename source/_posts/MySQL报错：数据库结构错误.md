---
title: MySQL报错：数据库结构错误
date: 2017-03-28 11:06:52
categories: 综合
tags:
- MySQL
---

> ERROR 1682 (HY000): Native table 'performance_schema'.'session_variables' has the wrong structure

## 重启MySQL
重启MySQL后还是报同样的错。

## 重启电脑
重启电脑后还是报同样的错。

## 谷歌搜一下
需要更新MySQL：

```
mysql_upgrade -u root -p
```

<!-- more -->

输出信息如下:

```
kangyonggandeMacBook-Pro:~ kyg$ mysql_upgrade -u root -p
Enter password: 
Checking if update is needed.
Checking server version.
Running queries to upgrade MySQL server.
Checking system database.
mysql.columns_priv                                 OK
mysql.db                                           OK
mysql.engine_cost                                  OK
mysql.event                                        OK
mysql.func                                         OK
mysql.general_log                                  OK
mysql.gtid_executed                                OK
mysql.help_category                                OK
mysql.help_keyword                                 OK
mysql.help_relation                                OK
mysql.help_topic                                   OK
mysql.innodb_index_stats                           OK
mysql.innodb_table_stats                           OK
mysql.ndb_binlog_index                             OK
mysql.plugin                                       OK
mysql.proc                                         OK
mysql.procs_priv                                   OK
mysql.proxies_priv                                 OK
mysql.server_cost                                  OK
mysql.servers                                      OK
mysql.slave_master_info                            OK
mysql.slave_relay_log_info                         OK
mysql.slave_worker_info                            OK
mysql.slow_log                                     OK
mysql.tables_priv                                  OK
mysql.time_zone                                    OK
mysql.time_zone_leap_second                        OK
mysql.time_zone_name                               OK
mysql.time_zone_transition                         OK
mysql.time_zone_transition_type                    OK
mysql.user                                         OK
The sys schema is already up to date (version 1.5.1).
Checking databases.
simconf.user_role                                  OK
simulator.bank_channel                             OK
simulator.bank_command                             OK
simulator.bank_command_log                         OK
simulator.bank_resp                                OK
simulator.bank_tran                                OK
simulator.dz_file                                  OK
simulator.menu                                     OK
simulator.role                                     OK
simulator.role_menu                                OK
simulator.sim_order                                OK
simulator.user                                     OK
simulator.user_role                                OK
sys.sys_config                                     OK
Upgrade process completed successfully.
Could not create the upgrade info file '/usr/local/mysql/data/mysql_upgrade_info' in the MySQL Servers datadir, errno: 13
kangyonggandeMacBook-Pro:~ kyg$
```

报错说是不能创建文件，可能是权限不足吧，于是

```
sudo mysql_upgrade -u root -p
```

输出:

```
...省略...
Upgrade process completed successfully.
Checking if update is needed.
```

这次没报错以为成功了，然后就测试了一把，发现还是报同样的错。

## 再次重启MySQL
测试后发现不报错了。

{% note success %} 问题是小问题，如果之前遇到过此类问题可以一步解决，但是如果没遇到过，就需要按部就班的去解决了。 {% endnote %}
