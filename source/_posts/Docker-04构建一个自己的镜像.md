---
title: Docker-04构建一个自己的镜像
date: 2017-03-25 18:53:50
categories: 综合
tags:
- Docker
---

> 官方文档:[https://docs.docker.com/engine/getstarted/step_four/](https://docs.docker.com/engine/getstarted/step_four/)

## 编写一个Dockerfile
你可以使用你喜欢的编辑器写一个简单的[Dockerfile](https://docs.docker.com/engine/reference/builder/), Dockerfile就是用来描述构建镜像的文件、环境和命令的清单，Dockerfile越小越好。

### 创建一个新的目录
```
root@iZ23ldh8kudZ:~/code# mkdir mydockerbuild
```

<!-- more -->

### 进入这个新的目录
```
root@iZ23ldh8kudZ:~/code# cd mydockerbuild
root@iZ23ldh8kudZ:~/code/mydockerbuild# pwd
/root/code/mydockerbuild
```

### 在当前目录创建文件`Dockerfile`
```
root@iZ23ldh8kudZ:~/code/mydockerbuild# vi Dockerfile
```

### 把`FROM`代码段写入文件
查看写入后的文件:

```
root@iZ23ldh8kudZ:~/code/mydockerbuild# cat Dockerfile 
FROM docker/whalesay:latest
```

`FROM`关键字告诉Docker我将以哪个镜像为基础。Whalesay是完美的，它已经有了cowsay程序，所以我们从它开始。

### 把`RUN`代码段写入镜像
查看写入后的文件:

```
root@iZ23ldh8kudZ:~/code/mydockerbuild# cat Dockerfile 
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
```

`RUN`关键字会安装镜像所需要的程序。whalesay镜像是基于Ubuntu的，所以它使用`apt-get`去安装所需要的包，这两个命令是请求此镜像可用的包，并且把fortunes程序安装到镜像，fortunes程序会打印出屋面所说的内容。


### 把`CMD`代码段写入镜像
查看写入后的文件:

```
root@iZ23ldh8kudZ:~/code/mydockerbuild# cat Dockerfile 
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay
```

`CMD`关键字告诉镜像当环境设置完成后运行最后的命令，这个命令是`fortune -a`，并且输出到`cowsay`命令。

## 从Dockerfile构建镜像
编译镜像使用的命令是`docker build`，参数`-t`是给镜像一个标签, 不要忽略了`.`，它会告诉`docker build`命令去当前目录下查找`Dockerfile`文件。

```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker build -t docker-whale .
Sending build context to Docker daemon 2.048 kB
Step 1/3 : FROM docker/whalesay:latest
 ---> 6b362a9f73eb
Step 2/3 : RUN apt-get -y update && apt-get install -y fortunes
 ---> Running in e7673f725ff2

...

Successfully built efb18db73358
```

第一次运行时有点慢，大概一分钟左右，此间它会去下载所需要的包，会输出很多看不懂的信息。

## 学习构建的过程
构建镜像的命令`docker build -t docker-whale .`，会在当前目录下读取`Dockerfile`文件，并在本地机器上一步一步的按照指令构建一个叫作`docker-whale`的镜像，构建需要一些时间，也会输出很多信息，下面来解析输出信息的意思。

### Docker检测以确保所有需要构建的都准备好了
```
Sending build context to Docker daemon 2.048 kB
```

### 检测依赖的基础镜像
```
Step 1/3 : FROM docker/whalesay:latest
 ---> 6b362a9f73eb
```

上面的输出信息对应的代码块是`FROM`, 如果本地没有whalesay镜像，则去Docker Hub下载，如果有，那就使用本地的whalesay镜像。

### Docker启动一个临时的容器去运行`whalesay`
在这个临时的容器中，Docker会运行Dockerfile中的下一行命令，即`RUN apt-get -y update && apt-get install -y fortunes`, 这个命令是去安装`fortunes`命令，此过程中输出了大量的信息。

```
Step 2/3 : RUN apt-get -y update && apt-get install -y fortunes
 ---> Running in e7673f725ff2
Ign http://archive.ubuntu.com trusty InRelease
Get:1 http://archive.ubuntu.com trusty-updates InRelease [65.9 kB]
Get:2 http://archive.ubuntu.com trusty-security InRelease [65.9 kB]
Hit http://archive.ubuntu.com trusty Release.gpg
Get:3 http://archive.ubuntu.com trusty-updates/main Sources [485 kB]
Get:4 http://archive.ubuntu.com trusty-updates/restricted Sources [5957 B]
Get:5 http://archive.ubuntu.com trusty-updates/universe Sources [220 kB]
Get:6 http://archive.ubuntu.com trusty-updates/main amd64 Packages [1197 kB]
Get:7 http://archive.ubuntu.com trusty-updates/restricted amd64 Packages [20.4 kB]
Get:8 http://archive.ubuntu.com trusty-updates/universe amd64 Packages [516 kB]
Get:9 http://archive.ubuntu.com trusty-security/main Sources [160 kB]
Get:10 http://archive.ubuntu.com trusty-security/restricted Sources [4667 B]
Get:11 http://archive.ubuntu.com trusty-security/universe Sources [59.4 kB]
Get:12 http://archive.ubuntu.com trusty-security/main amd64 Packages [730 kB]
Get:13 http://archive.ubuntu.com trusty-security/restricted amd64 Packages [17.0 kB]
Get:14 http://archive.ubuntu.com trusty-security/universe amd64 Packages [199 kB]
Hit http://archive.ubuntu.com trusty Release
Hit http://archive.ubuntu.com trusty/main Sources
Hit http://archive.ubuntu.com trusty/restricted Sources
Hit http://archive.ubuntu.com trusty/universe Sources
Hit http://archive.ubuntu.com trusty/main amd64 Packages
Hit http://archive.ubuntu.com trusty/restricted amd64 Packages
Hit http://archive.ubuntu.com trusty/universe amd64 Packages
Fetched 3745 kB in 43s (87.0 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following extra packages will be installed:
  fortune-mod fortunes-min librecode0
Suggested packages:
  x11-utils bsdmainutils
The following NEW packages will be installed:
  fortune-mod fortunes fortunes-min librecode0
0 upgraded, 4 newly installed, 0 to remove and 92 not upgraded.
Need to get 1961 kB of archives.
After this operation, 4817 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu/ trusty/main librecode0 amd64 3.6-21 [771 kB]
Get:2 http://archive.ubuntu.com/ubuntu/ trusty/universe fortune-mod amd64 1:1.99.1-7 [39.5 kB]
Get:3 http://archive.ubuntu.com/ubuntu/ trusty/universe fortunes-min all 1:1.99.1-7 [61.8 kB]
Get:4 http://archive.ubuntu.com/ubuntu/ trusty/universe fortunes all 1:1.99.1-7 [1089 kB]
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (This frontend requires a controlling tty.)
debconf: falling back to frontend: Teletype
dpkg-preconfigure: unable to re-open stdin: 
Fetched 1961 kB in 4s (466 kB/s)
Selecting previously unselected package librecode0:amd64.
(Reading database ... 13116 files and directories currently installed.)
Preparing to unpack .../librecode0_3.6-21_amd64.deb ...
Unpacking librecode0:amd64 (3.6-21) ...
Selecting previously unselected package fortune-mod.
Preparing to unpack .../fortune-mod_1%3a1.99.1-7_amd64.deb ...
Unpacking fortune-mod (1:1.99.1-7) ...
Selecting previously unselected package fortunes-min.
Preparing to unpack .../fortunes-min_1%3a1.99.1-7_all.deb ...
Unpacking fortunes-min (1:1.99.1-7) ...
Selecting previously unselected package fortunes.
Preparing to unpack .../fortunes_1%3a1.99.1-7_all.deb ...
Unpacking fortunes (1:1.99.1-7) ...
Setting up librecode0:amd64 (3.6-21) ...
Setting up fortune-mod (1:1.99.1-7) ...
Setting up fortunes-min (1:1.99.1-7) ...
Setting up fortunes (1:1.99.1-7) ...
Processing triggers for libc-bin (2.19-0ubuntu6.6) ...
 ---> 785085e9a520
Removing intermediate container e7673f725ff2
```

当`RUN`命令运行结束后，一个新的`layer`就会立即产生，并且销毁临时容器。

### 一个新的临时容器产生，并且Docker会添加`layer`，对应于Dockerfile中的`CMD`命令, 最后再销毁临时容器。
```
Step 3/3 : CMD /usr/games/fortune -a | cowsay
 ---> Using cache
 ---> efb18db73358
Successfully built efb18db73358
```

## 运行镜像
现在，校验这个新镜像是否在本地存在，并且去运行它。

### 用`docker images`命令查看本地镜像
```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker-whale        latest              efb18db73358        31 minutes ago      275 MB
hello-world         latest              48b5124b2768        5 weeks ago         1.84 kB
docker/whalesay     latest              6b362a9f73eb        21 months ago       247 MB
```

### 运行新镜像`docker run docker-whale`
```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker run docker-whale
 _________________________ 
/ manic-depressive, adj.: \
|                         |
\ Easy glum, easy glow.   /
 ------------------------- 
    \
     \
      \     
                    ##        .            
              ## ## ##       ==            
           ## ## ## ##      ===            
       /""""""""""""""""___/ ===        
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
       \______ o          __/            
        \    \        __/             
          \____\______/   
```

运行时，你会发现Docker并没有去再下载任何东西，因为镜像已经在本地构建了。

### 好不容易构建好了，再运行一次
```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker run docker-whale
 ________________________________________ 
/ Power, n.:                             \
|                                        |
| The only narcotic regulated by the SEC |
\ instead of the FDA.                    /
 ---------------------------------------- 
    \
     \
      \     
                    ##        .            
              ## ## ##       ==            
           ## ## ## ##      ===            
       /""""""""""""""""___/ ===        
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
       \______ o          __/            
        \    \        __/             
          \____\______/ 
```

你会发现，输出竟然不一样了，好像它有自己的思维，那你就多运行几次看看吧。

