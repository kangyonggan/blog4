---
title: Ubuntu14.0.4安装及配置系列-jenkins的安装配置
date: 2017-03-25 17:20:33
categories: 系统运维
tags:
- Linux
- Jenkins
---

### 下载[jenkins](jenkins.io/index.html)到本地

### 上传本地jenkins到Ubuntu服务器

```
$ scp jenkins.war root@121.40.66.176:/root/soft/
```

### 直接把war包放入tomcat的webapps目录下并重启tomcat即可

<!-- more -->

### 安装maven插件
去这个地址[http://updates.jenkins-ci.org/download/plugins/](http://updates.jenkins-ci.org/download/plugins/)下载maven插件到本地

然后在jenkins-->系统管理-->管理插件-->高级-->上传插件-->选择maven插件(maven-plugin.hpi)-->上传-->重启

### jenkins执行shell重启tomcat无效
需要在Post steps中执行`export BUILD_ID=BUILD_BLOG`,如:

```
export BUILD_ID=BUILD_BLOG
sh /home/kyg/shell/deploy-blog.sh
```

### 构建时， 需要指定maven配置
如果没指定， 会使用默认配置，会联网下载依赖到.m2文件夹下，所以需要指定`settings.xml`配置文件的位置, 如下图:

![jenkins-maven](/uploads/20170101020115076.png)


