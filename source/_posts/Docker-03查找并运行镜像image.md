---
title: Docker-03查找并运行镜像image
date: 2017-03-25 18:51:02
categories: 综合
tags:
- Docker
---

> 官方文档:[https://docs.docker.com/engine/getstarted/step_three/](https://docs.docker.com/engine/getstarted/step_three/)

Docker Hub上存放着全世界的docker镜像，我们可以浏览、下载并运行镜像。

## 定位whalesay镜像
### 打开浏览器，浏览[Docker Hub](https://hub.docker.com/)
![browse_and_search](/uploads/20170223110047136.png)

Docker Hub上的镜像包含了个人（比如我）、官方和一些组织的，比如：RedHat, IBM, Google等。

<!-- more -->

### 搜索关键词`whalesay`
![image_found](/uploads/20170223110047137.png)

### 在结果页点击`docker/whalesay`镜像
浏览器会跳转到whalesay镜像库界面。

![whale_repo](/uploads/20170223110047137.png)

每个镜像库都包含镜像的信息，比如：镜像的分类是什么？怎么使用镜像？  

### 运行`whalesay`镜像
#### 执行命令
```
root@iZ23ldh8kudZ:~# docker run docker/whalesay cowsay boo
Unable to find image 'docker/whalesay:latest' locally
latest: Pulling from docker/whalesay
e190868d63f8: Pull complete 
909cd34c6fd7: Pull complete 
0b9bfabab7c1: Pull complete 
a3ed95caeb02: Pull complete 
00bf65475aba: Pull complete 
c57b6bcc83e3: Pull complete 
8978f6879e2f: Pull complete 
8eed3712d2cf: Pull complete 
Digest: sha256:178598e51a26abbc958b8a2e48825c90bc22e641de3d31e18aaf55f3258ba93b
Status: Downloaded newer image for docker/whalesay:latest
 _____ 
< boo >
 ----- 
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

第一次运行可能会比较慢，大概需要一分钟左右，需耐心等待，之所以第一次运行较慢，是因为本地仓库没有此镜像，需要去Docker Hub下载。

#### 运行`docker images`命令，查看本地仓库的所有镜像,就会发现`whalesay`镜像已经被下载到本地了。
```
root@iZ23ldh8kudZ:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              48b5124b2768        5 weeks ago         1.84 kB
docker/whalesay     latest              6b362a9f73eb        21 months ago       247 MB
```

当你在容器中运行一个镜像，Docker会把镜像下载到你本地，本地的副本会为你下次运行节省时间。当且仅当Docker Hub上的镜像来源发生改变时，Docker才会去重新下载。你也可以手动删除本地镜像。

#### 再次运行`whalesay`镜像
```
root@iZ23ldh8kudZ:~# docker run docker/whalesay cowsay boo-boo
 _________ 
< boo-boo >
 --------- 
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
          \____\______
```

