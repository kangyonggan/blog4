---
title: jdk1.8发送http请求报SSLv3的解决方案
date: 2017-03-25 17:51:02
categories: 综合
tags:
- Java
---

从 JDK 8u31 发行版开始，已停用 SSLv3 协议（安全套接字层），该协议在正常情况下不可用。如果确实需要 SSLv3，可以重新激活该协议...

### SSLv3激活步骤
切到${java_home}/jre/lib/security目录

```
cd /Library/Java/JavaVirtualMachines/jdk1.8.0_91.jdk/Contents/Home/jre/lib/security
```

<!-- more -->

修改`java.security`文件

```
vi java.security
```

找到`jdk.tls.disabledAlgorithms`属性， 删除`SSLv3`并保存

查找:`/jdk.tls.disabledAlgorithms`

> 温馨提示:需要root权限