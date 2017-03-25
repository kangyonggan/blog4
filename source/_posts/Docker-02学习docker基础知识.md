---
title: Docker-02学习docker基础知识
date: 2017-03-25 18:49:46
categories: 综合
tags:
- Docker
---

> 官方文档:[https://docs.docker.com/engine/getstarted/](https://docs.docker.com/engine/getstarted/)

## 将会学到
- 怎么在docker容器中运行软件镜像
- 怎么Docker Hub上浏览软件镜像
- 怎么创建自己的镜像并运行在容器中
- 怎么创建自己的Docker Hub账户和镜像库
- 怎么创建一个镜像
- 上传你的镜像到Docker Hub给其他人使用

<!-- more -->

## 理解镜像和容器
Docker引擎提供的核心技术是镜像和容器。  
`docker run hello-world`这条命令由下面三部分组成:

![container_explainer.png](/uploads/20170223100628926.png) 
镜像是文件系统和运行时技术。它没有状态也永远不会改变。一个容器就是一个运行中的镜像实例。当你执行hello-world命令，docker引擎：

- 校验你是否有hello-world镜像
- 从docker hub下载镜像
- 把镜像加载进容器中并且运行
