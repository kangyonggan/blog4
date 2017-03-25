---
title: Ubuntu14.0.4安装及配置系列-jdk1.8的安装配
date: 2017-03-25 16:39:05
categories: 系统运维
tags: 
- Linux
- Java
---

### 下载[jdk1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)到本地

### 上传本地jdk到Ubuntu服务器
```
$ scp jdk-8u111-linux-x64.tar.gz root@121.40.66.176:/root/soft/
```

### 解压jdk到指定目录
```
# tar -zxvf jdk-8u111-linux-x64.tar.gz -C /root/install/
```

### 配置jdk的环境变量
在`/etc/profile`文件尾追加下面的配置:

```
# jdk environment  
export JAVA_HOME=/root/install/jdk1.8.0_111  
export JRE_HOME=/root/install/jdk1.8.0_111/jre  
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH  
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

<!-- more -->

之后，重新加载配置，让配置生效:

```
# source profile
```

### 5. 测试环境变量是否生效
```
# java -version
```



