---
title: windows防火墙导致ftp文件上传异常
date: 2017-03-25 18:29:46
categories: 综合
tags:
- Java
---

在win7系统下运行web项目，发现ftp文件上传异常，但是在linux下是好的，win7下报错信息如下：

```
[INFO ] 2017-01-21 21:24:03.388 [com.kangyonggan.archetype.cms.biz.util.Ftp.http-bio-8080-exec-10:53] - 连接文件服务器成功, 上传路径path:upload/
[ERROR] 2017-01-21 21:24:22.899 [com.kangyonggan.archetype.cms.biz.util.Ftp.http-bio-8080-exec-10:75] - 文件上传异常
java.net.SocketException: Software caused connection abort: socket write error
oat java.net.SocketOutputStream.socketWrite0(Native Method) ~[?:1.8.0_111]
oat java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:109) ~[?:1.8.0_111]
oat java.net.SocketOutputStream.write(SocketOutputStream.java:153) ~[?:1.8.0_111]

...
```

<!-- more -->

原因是win7防火墙的问题，关闭防火墙后一切就正常了。

![关闭防火墙](/uploads/20170121214431475.png)

