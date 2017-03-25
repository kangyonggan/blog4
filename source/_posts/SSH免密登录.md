---
title: SSH免密登录
date: 2017-03-25 17:09:17
categories: 系统运维
tags:
- Linux
- SSH
---

现在有两台机器A和B， 想要用ssh从A登录到B， 并且不使用密码

### 在A机器上生成密钥对
```
ssh-keygen -t rsa
```

然后会有三次提示用户输入， 什么也不要输入， 直接回车， 就会在用户根目录生成`.ssh`文件夹, 文件夹里会有`id_rsa`私钥和`id_rsa.pub`公钥

### 把公钥导入B机器
```
ssh-copy-id -i .ssh/id_rsa.pub root@192.168.20.212
```

<!-- more -->

执行这个命令会要求输入一次密码， 不过以后就不用再输密码了。  
如果A机器没安装ssh-copy-id命令， 也可以手动导入， 先把公钥scp到B机器（也要输入一次密码），   
然后输出到.ssh/authorized_keys文件中即可

```
cat id_rsa.pub > .ssh/authorized_keys
```

### 免密登录测试

```
ssh root@192.168.20.212
```
