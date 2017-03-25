---
title: Docker-06推送push和拉取pull镜像
date: 2017-03-25 18:57:21
categories: 综合
tags:
- Docker
---

> 官方文档:[https://docs.docker.com/engine/getstarted/step_six/](https://docs.docker.com/engine/getstarted/step_six/)

推送本地镜像到Docker Hub上你的镜像库中，然后再镜像从镜像库中拉取到本地，并运行它。

## 推送镜像到镜像库
### 查看本地所有镜像
```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker images 
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker-whale        latest              efb18db73358        About an hour ago   275 MB
hello-world         latest              48b5124b2768        5 weeks ago         1.84 kB
docker/whalesay     latest              6b362a9f73eb        21 months ago       247 MB
```

<!-- more -->

### 找到镜像ID
以`docker-whale`镜像为例，它的镜像ID是`efb18db73358`

> 现在, docker-whale镜像还没有命名空间， 你需要关联一个命名空间， 命名空间就是你的Docker Hub账户名（我的是kangyonggan, 然后给镜像名添加命名空间，比如: kangyonggan/docker-whale

### 用`docker tag`命令给镜像添加标签（命名空间）
![tagger](/uploads/20170223153853566.png)

```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker tag efb18db73358 kangyonggan/docker-whale:latest
```

#### 4. 再次运行`docker images`会发现镜像已经有了标签
```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker images 
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
docker-whale               latest              efb18db73358        About an hour ago   275 MB
kangyonggan/docker-whale   latest              efb18db73358        About an hour ago   275 MB
hello-world                latest              48b5124b2768        5 weeks ago         1.84 kB
docker/whalesay            latest              6b362a9f73eb        21 months ago       247 MB
```

还会发现，相同的镜像ID存在于两个不同的镜像库。

### 登录Docker Hub
在最终把镜像推送到Docker Hub上之前，你需要登录Docker Hub。

```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker login 
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: kangyonggan
Password: 
Login Succeeded
```

### 把镜像推送到自己的Docker Hub上
```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker push kangyonggan/docker-whale
The push refers to a repository [docker.io/kangyonggan/docker-whale]
4d8b662d1a5a: Pushing [========>                                          ] 5.001 MB/28.13 MB
5f70bf18a086: Mounted from docker/whalesay 
d061ee1340ec: Mounted from docker/whalesay 
d511ed9e12e1: Mounted from docker/whalesay 
091abc5148e4: Mounted from docker/whalesay 
b26122d57afa: Mounted from docker/whalesay 
37ee47034d9b: Mounted from docker/whalesay 
528c8710fd95: Mounted from docker/whalesay 
1154ba695078: Mounted from docker/whalesay 
```

网速有点慢(目测要翻墙才能快)，才推送了5/28M。

### 登录Docker Hub查看新推送的镜像
![resp_list.png](/uploads/20170223154255805.png)

由于网速太慢（怀疑是官网太慢），截图的时候还没推送完成。

## 从镜像库拉取镜像
`docker pull`是用来拉取镜像的，如果本地已经有最新版的镜像，`docker pull`命令将什么也不做，为了验证是真正的从镜像库中拉取下来的，你可以先删除本地镜像`docker image remove`。

### 用`docker image remove`删除本地镜像
```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker images 
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
docker-whale2              latest              efb18db73358        2 hours ago         275 MB
docker-whale               latest              efb18db73358        2 hours ago         275 MB
kangyonggan/docker-whale   latest              efb18db73358        2 hours ago         275 MB
hello-world                latest              48b5124b2768        5 weeks ago         1.84 kB
docker/whalesay            latest              6b362a9f73eb        21 months ago       247 MB

root@iZ23ldh8kudZ:~/code/mydockerbuild# docker image remove efb18db73358
Error response from daemon: conflict: unable to delete efb18db73358 (must be forced) - image is referenced in multiple repositories
```

通过镜像ID删除一个本地镜像，可能会报错，如果此时你又两个ID一样的镜像，另外也可以通过镜像标签删除镜像。

```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker image remove docker-whale2
Untagged: docker-whale2:latest

root@iZ23ldh8kudZ:~/code/mydockerbuild# docker images 
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
docker-whale               latest              efb18db73358        2 hours ago         275 MB
kangyonggan/docker-whale   latest              efb18db73358        2 hours ago         275 MB
hello-world                latest              48b5124b2768        5 weeks ago         1.84 kB
docker/whalesay            latest              6b362a9f73eb        21 months ago       247 MB
```

### 用`docker run`命令从Docker Hub拉取镜像，当你本地没这个镜像的时候。

```
root@iZ23ldh8kudZ:~/code/mydockerbuild# docker run kangyonggan/docker-whale
```

---

更多用法请移步:[https://docs.docker.com/engine/getstarted/last_page/](https://docs.docker.com/engine/getstarted/last_page/)

