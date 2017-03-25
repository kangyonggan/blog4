---
title: 手动安装本地jar包到仓库
date: 2017-03-25 18:44:36
categories: 综合
tags:
- Maven
---

```
mvn install:install-file -DgroupId=com.kangyonggan.app -DartifactId=simclient -Dversion=1.0-SNAPSHOT -Dpackaging=jar -Dfile=/home/kyg/data/jar/simclient-1.0-SNAPSHOT.jar
```
