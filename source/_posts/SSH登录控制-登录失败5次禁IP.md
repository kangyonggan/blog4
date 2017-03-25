---
title: 'SSH登录控制,登录失败5次禁IP'
date: 2017-03-25 19:15:29
categories: 综合
tags:
- Linux
---

今天在登录我服务器的时候，发现了下面这段话

```
kangyonggandeMacBook-Pro:~ kyg$ ./.login.sh 
Last failed login: Sun Mar  5 13:27:21 EST 2017 from 42.196.186.61 on ssh:notty
There were 721 failed login attempts since the last successful login.
Last login: Sat Mar  4 07:33:47 2017 from 192.168.2.222
```

<!-- more -->

发现是有人尝试爆破我的服务器，我的服务器是CentOS7,于是我查看日志：

```
[root@localhost log]# pwd
/var/log
[root@localhost log]# ll
总用量 10044
drwxr-xr-x. 2 root   root       176 2月  26 08:37 anaconda
drwx------. 2 root   root        99 3月   5 07:30 audit
-rw-r--r--. 1 root   root      9671 3月   1 08:05 boot.log
-rw-------. 1 root   utmp    473856 3月   5 13:27 btmp
-rw-------. 1 root   utmp    462720 2月  28 12:15 btmp-20170301
drwxr-xr-x. 2 chrony chrony       6 12月  6 17:42 chrony
-rw-------. 1 root   root     51771 3月   6 02:01 cron
-rw-r--r--. 1 root   root     59619 3月   1 08:05 dmesg
-rw-r--r--. 1 root   root     59102 2月  28 06:39 dmesg.old
-rw-r--r--. 1 root   root      2873 2月  27 07:19 firewalld
-rw-------. 1 root   root      1280 2月  26 08:42 grubby
-rw-r--r--. 1 root   root    291708 3月   6 02:06 lastlog
-rw-------. 1 root   root      3805 3月   5 03:00 maillog
-rw-------. 1 root   root   3661588 3月   6 02:20 messages
-rw-r--r--. 1 mysql  mysql    65242 3月   2 02:48 mysqld.log
drwx------. 2 root   root         6 6月  10 2014 ppp
drwxr-xr-x. 2 root   root         6 2月  26 08:37 rhsm
-rw-------. 1 root   root   3081270 3月   6 02:20 secure
-rw-------. 1 root   root         0 2月  26 08:35 spooler
-rw-------. 1 root   root         0 2月  26 08:34 tallylog
drwxr-xr-x. 2 root   root        23 12月  6 17:26 tuned
-rw-------. 1 root   root     29494 3月   6 02:17 wpa_supplicant.log
-rw-r--r--. 1 root   root     51059 2月  27 06:32 wpa_supplicant.log-20170227
-rw-rw-r--. 1 root   utmp     44160 3月   6 02:06 wtmp
-rw-------. 1 root   root      7438 2月  27 07:03 yum.log

[root@localhost log]# grep "Failed password for" secure 
...太多就不贴出来了
Mar  4 21:34:02 localhost sshd[24674]: Failed password for root from 42.196.186.61 port 56860 ssh2
Mar  4 21:34:05 localhost sshd[24674]: Failed password for root from 42.196.186.61 port 56860 ssh2
Mar  4 21:34:07 localhost sshd[24679]: Failed password for root from 42.196.186.61 port 56862 ssh2
Mar  4 21:34:09 localhost sshd[24679]: Failed password for root from 42.196.186.61 port 56862 ssh2
Mar  4 21:34:12 localhost sshd[24679]: Failed password for root from 42.196.186.61 port 56862 ssh2
Mar  4 21:34:14 localhost sshd[24684]: Failed password for root from 42.196.186.61 port 56962 ssh2
Mar  4 21:34:17 localhost sshd[24684]: Failed password for root from 42.196.186.61 port 56962 ssh2
Mar  5 04:37:23 localhost sshd[26486]: Failed password for root from 42.196.186.61 port 57172 ssh2
Mar  5 04:37:25 localhost sshd[26486]: Failed password for root from 42.196.186.61 port 57172 ssh2
Mar  5 04:37:26 localhost sshd[26486]: Failed password for root from 42.196.186.61 port 57172 ssh2
Mar  5 04:37:28 localhost sshd[26486]: Failed password for root from 42.196.186.61 port 57172 ssh2
Mar  5 04:37:30 localhost sshd[26486]: Failed password for root from 42.196.186.61 port 57172 ssh2
Mar  5 04:37:32 localhost sshd[26486]: Failed password for root from 42.196.186.61 port 57172 ssh2
Mar  5 05:06:04 localhost sshd[26627]: Failed password for root from 42.196.186.61 port 60976 ssh2
Mar  5 05:06:07 localhost sshd[26627]: Failed password for root from 42.196.186.61 port 60976 ssh2
Mar  5 05:06:09 localhost sshd[26627]: Failed password for root from 42.196.186.61 port 60976 ssh2
Mar  5 05:06:12 localhost sshd[26627]: Failed password for root from 42.196.186.61 port 60976 ssh2
Mar  5 05:06:14 localhost sshd[26627]: Failed password for root from 42.196.186.61 port 60976 ssh2
Mar  5 05:06:17 localhost sshd[26627]: Failed password for root from 42.196.186.61 port 60976 ssh2
Mar  5 10:05:12 localhost sshd[27881]: Failed password for root from 123.207.23.34 port 38482 ssh2
Mar  5 10:05:15 localhost sshd[27881]: Failed password for root from 123.207.23.34 port 38482 ssh2
Mar  5 10:05:17 localhost sshd[27881]: Failed password for root from 123.207.23.34 port 38482 ssh2
Mar  5 10:05:19 localhost sshd[27881]: Failed password for root from 123.207.23.34 port 38482 ssh2
Mar  5 10:05:21 localhost sshd[27881]: Failed password for root from 123.207.23.34 port 38482 ssh2
Mar  5 10:05:24 localhost sshd[27881]: Failed password for root from 123.207.23.34 port 38482 ssh2
Mar  5 13:27:07 localhost sshd[28721]: Failed password for root from 42.196.186.61 port 43224 ssh2
Mar  5 13:27:10 localhost sshd[28721]: Failed password for root from 42.196.186.61 port 43224 ssh2
Mar  5 13:27:13 localhost sshd[28721]: Failed password for root from 42.196.186.61 port 43224 ssh2
Mar  5 13:27:16 localhost sshd[28721]: Failed password for root from 42.196.186.61 port 43224 ssh2
Mar  5 13:27:18 localhost sshd[28721]: Failed password for root from 42.196.186.61 port 43224 ssh2
Mar  5 13:27:21 localhost sshd[28721]: Failed password for root from 42.196.186.61 port 43224 ssh2
```

看到这么多的登录失败，我觉得我需要做些什么才行，比如：禁止用户名密码登录，只允许秘钥登录，但是有时候用别人的电脑没秘钥会不方便，所以，我决定`登录失败超过5次禁IP`

下面是脚本`.ip-deny.sh`：

```
#! /bin/sh

cat /var/log/secure | awk '/Failed/{print $(NF-3)}' | sort | uniq -c | awk '{print $2"="$1;}' > /root/ip-black.txt

MAX=5

for i in `cat  /root/ip-black.txt`
do
	ip=`echo $i |awk -F= '{print $1}'`
	cnt=`echo $i |awk -F= '{print $2}'`
	if [ $cnt -gt $MAX ] 
	then
		grep $ip /etc/hosts.deny > /dev/null
		if [ $? -gt 0 ]
		then
			echo "sshd:$ip:deny" >> /etc/hosts.deny
		fi
	fi

done

cat /etc/hosts.deny
```

最后在定时任务中，每隔1分钟执行一次脚本：

```
[root@localhost ~]# crontab -e
```

查看所有定时任务：

```
[root@localhost ~]# crontab -l
# m h  dom mon dow   command


# 每天凌晨三点备份数据
0 3 * * * sh /root/.back.sh

# 每隔1分钟执行一次，登录失败超过5次禁IP
*/1 * * * *  sh /root/.ip-deny.sh
```

经试验，登录失败五次，然后再经过1分钟之后（之内），试验生效！








