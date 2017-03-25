---
title: 用jdk自带的keytool导入查看删除证书
date: 2017-03-25 17:18:51
categories: Java后台
tags:
- Java
---

## 导入证书
```
sudo keytool -import -keystore /Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre/lib/security/cacerts -storepass changeit -alias sh2_test -file /Users/kyg/Downloads/sh2_test.cer
```

## 删除证书
```
sudo keytool -delete -alias sh2_test -keystore /Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre/lib/security/cacerts -storepass changeit
```

## 查看证书
```
keytool -list -alias sh2_test -keystore /Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre/lib/security/cacerts -storepass changeit
```

## 提示
- 使用时，jdk路径请自行替换
- 一般cacerts的密码默认为changit
- keytool命令在bin目录下