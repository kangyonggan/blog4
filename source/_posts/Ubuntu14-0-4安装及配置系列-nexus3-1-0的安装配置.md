---
title: Ubuntu14.0.4安装及配置系列-nexus3.1.0的安装配置
date: 2017-03-25 16:51:07
categories: 系统运维
tags: 
- Linux
- Nexus
---

### 下载[nexus3.1.0](https://www.sonatype.com/download-oss-sonatype)到本地

### 上传本地nexus到Ubuntu服务器
```
$ scp nexus-3.1.0-04-unix.tar.gz root@121.40.66.176:/root/soft/
```

### 解压nexus到指定目录
```
# tar -zxvf nexus-3.1.0-04-unix.tar.gz -C /root/install/
```

### 启动
```
# ./nexus start
```

<!-- more -->

### 停止
```
# ./nexus stop
```

### 访问
http://localhost:8081/

- 用户名:admin
- 密码:admin123

### 使用

配置`settings.xml`

```
<servers>
    <server>
      <id>releases</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <id>snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
  </servers>
```

配置`pom.xml`

```
<distributionManagement>
    <repository>
        <id>releases</id>
        <name>nexus releases</name>
        <url>http://kangyonggan.com:8081/repository/maven-releases/</url>
    </repository>

    <snapshotRepository>
        <id>snapshots</id>
        <name>nexus snapshots</name>
        <url>http://kangyonggan.com:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

