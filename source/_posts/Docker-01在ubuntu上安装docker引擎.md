---
title: Docker-01在ubuntu上安装docker引擎
date: 2017-03-25 18:48:26
categories: 综合
tags:
- Docker
---

docker引擎支持Linux、Cloud、Windows和macOS，由于个人精力有限，我不可能一一实验，我选择在Ubuntu14.04上安装，本系列文章是通过学习官方文档整理而来。

> 官方文档:[https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

## 安装要求
- Yakkety 16.10
- Xenial 16.04 (LTS)
- Trusty 14.04 (LTS)✔️

<!-- more -->

我的实验环境是Ubuntu 14.04 LTS, `lsb_release -a`可以查看版本号

```
root@iZ23ldh8kudZ:~# lsb_release -a
No LSB modules are available.
Distributor ID:Ubuntu
Description:Ubuntu 14.04.4 LTS
Release:14.04
Codename:trusty
```

## 推荐安装额外包
`linux-image-extra-*`包，它允许Docker使用存储驱动，一般使用Docker都要安装，除非你有不得不说的理由。

```
root@iZ23ldh8kudZ:~# sudo apt-get update

root@iZ23ldh8kudZ:~# sudo apt-get install -y --no-install-recommends \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
```

## 使用镜像库安装Docker
安装Docker的方法有很多，选择一个你需要的即可，我是使用镜像库安装的。

- 配置镜像库安装Docker(大部分用户的选择）✔️
- 下载DEB包安装Docker

第一次在新机器上安装Docker的时候，需要配置镜像库，然后就可以从镜像库安装、更新或降级Docker

### 允许apt通过https使用镜像库
```
root@iZ23ldh8kudZ:~# sudo apt-get install -y --no-install-recommends \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

### 添加Docker官方公钥
```
root@iZ23ldh8kudZ:~# curl -fsSL https://apt.dockerproject.org/gpg | sudo apt-key add -
```

校验公钥`58118E89F3A912897C070ADBF76221572C52609D`:

```
root@iZ23ldh8kudZ:~# apt-key fingerprint 58118E89F3A912897C070ADBF76221572C52609D
/etc/apt/trusted.gpg
--------------------
pub   1024D/437D05B5 2004-09-12
      Key fingerprint = 6302 39CC 130E 1A7F D81A  27B1 4097 6EAF 437D 05B5
uid                  Ubuntu Archive Automatic Signing Key <ftpmaster@ubuntu.com>
sub   2048g/79164387 2004-09-12

...
```

### 用下面的命令去稳定你的镜像库
```
root@iZ23ldh8kudZ:~# sudo add-apt-repository \
       "deb https://apt.dockerproject.org/repo/ \
       ubuntu-$(lsb_release -cs) \
       main"
```

> `lsb_release -cs`这个子命令返回你的ubuntu系统的代号，如`trusty`

启用测试镜像库。通过编辑`/etc/apt/sources.list`,并在下面这行的最后添加`testing`。  
deb https://apt.dockerproject.org/repo/ ubuntu-trusty main  
添加后:  
deb https://apt.dockerproject.org/repo/ ubuntu-trusty main testing

## 安装Docker
### 更新`apt`包
```
root@iZ23ldh8kudZ:~# sudo apt-get update
```

### 安装最新版docker，或者在下一步安装指定版本的
用下面的命令安装最新版

```
root@iZ23ldh8kudZ:~# sudo apt-get -y install docker-engine
```

### 在生产机器，你需要安装指定版本的docker，不要总是使用最新版，下面的命令列出了所有可用版本
```
root@iZ23ldh8kudZ:~# apt-cache madison docker-engine
docker-engine | 17.03.0~ce~rc1-0~ubuntu-trusty | https://apt.dockerproject.org/repo/ ubuntu-trusty/testing amd64 Packages
docker-engine | 1.13.1-0~ubuntu-trusty | https://apt.dockerproject.org/repo/ ubuntu-trusty/main amd64 Packages
docker-engine | 1.13.1~rc2-0~ubuntu-trusty | https://apt.dockerproject.org/repo/ ubuntu-trusty/testing amd64 Packages
docker-engine | 1.13.1~rc1-0~ubuntu-trusty | https://apt.dockerproject.org/repo/ ubuntu-trusty/testing amd64 Packages

...
```
每行的第二列是版本号，第三列是镜像库名，然后选择一个指定的版本进行安装。

```
root@iZ23ldh8kudZ:~# sudo apt-get -y install docker-engine=<版本号>
```

### 运行`Hello World`来检验是否安装正确
```
root@iZ23ldh8kudZ:~# sudo docker run hello-world
```

运行后报错了:

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
78445dd45222: Pull complete 
docker: error pulling image configuration: Get https://registry-1.docker.io/v2/library/hello-world/blobs/sha256:48b5124b2768d2b917edcb640435044a97967015485e812545546cbed5cf0233: net/http: TLS handshake timeout.
See 'docker run --help'.
```

重启docker服务:

```
root@iZ23ldh8kudZ:~# service docker restart 
docker stop/waiting
docker start/running, process 18050
```

再次运行:

```
root@iZ23ldh8kudZ:~# docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```
