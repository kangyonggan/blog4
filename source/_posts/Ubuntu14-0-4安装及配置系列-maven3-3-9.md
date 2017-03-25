---
title: Ubuntu14.0.4安装及配置系列-maven3.3.9
date: 2017-03-25 16:42:26
categories: 系统运维
tags: 
- Linux
- Maven
---

### 下载[maven3.3.9](http://maven.apache.org/download.cgi)到本地

### 上传本地maven到Ubuntu服务器
```
$ scp apache-maven-3.3.9-bin.tar.gz root@121.40.66.176:/root/soft/
```

### 解压maven到指定目录
```
# tar -zxvf apache-maven-3.3.9-bin.tar.gz -C /root/install/
```

### 配置maven的环境变量
在`/etc/profile`文件尾追加下面的配置:

```
# maven environment
export M2_HOME=/root/install/apache-maven-3.3.9
export PATH=$PATH:$M2_HOME/bin
```

<!-- more -->

之后，重新加载配置，让配置生效:

```
# source profile
```

### 5. 测试环境变量是否生效
```
# mvn -version
```

### 6. 配置本地仓库路径
在`/root/install/apache-maven-3.3.9/conf/settings.xml`文件中配置:

```
<localRepository>/root/data/repository</localRepository>
```




