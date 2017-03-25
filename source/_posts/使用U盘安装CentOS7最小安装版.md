---
title: 使用U盘安装CentOS7最小安装版
date: 2017-03-25 19:25:47
categories: 综合
tags:
- Linux
---

## 下载镜像文件和刻录工具
1. CentOS7最小安装版镜像文件下载地址：[http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1611.iso](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1611.iso)

2. 刻录工具下载地址：[https://cn.ultraiso.net/uiso9_cn.exe](https://cn.ultraiso.net/uiso9_cn.exe)

<!-- more -->

## 把系统镜像刻录到U盘
打开刻录工具`UltraISO`，【文件】-->【打开】-->【选择下载好的镜像CentOS-7-x86_64-Minimal-1611.iso】

![打开镜像](/uploads/centos-001.png)

【启动】-->【写入硬盘映像...】

![写入硬盘映像](/uploads/centos-002.png)

这个过程会格式化U盘，请先备份U盘内的数据，然后点击【写入】，写入过程需要一些时间。

![写入](/uploads/centos-003.png)

一会之后，写入完成！至此，你就完成了准备工作“把系统镜像刻录到U盘”，接下来就是要用U盘安装系统了。

![写入成功](/uploads/centos-004.png)

## 使用U盘启动
在按开机按钮后，点按F12（大多数品牌的电脑都是按F12进入一次性启动项），进入启动项后，选择带USB字样的那一项进入

![bios-usb](/uploads/centos-005.jpg)

进入CentOS7的安装界面，选择第一行`Install CentOS Linux 7`

![安装CentOS7](/uploads/centos-006.jpg)

## 安装系统
选择系统语言，这里我选择的是英语（推荐使用英语）。

![选择语言](/uploads/centos-015.jpg)

选择磁盘，点击【SYSTEM】-->【INSTALLION DESTINATION】

![选择磁盘](/uploads/centos-007.jpg)

选中自己电脑的硬盘，点击【Done】

![选中自己电脑的硬盘](/uploads/centos-008.jpg)

释放磁盘空间，点击【Reclaim space】

![释放磁盘空间](/uploads/centos-009.jpg)

删除老系统占用的空间，点击【Delete】

![删除老系统占用的空间](/uploads/centos-010.jpg)

删除后点击【Reclaim space】完成。

![删除老系统占用的空间完成](/uploads/centos-011.jpg)

开始安装，点击【Begin Installation】

![开始安装](/uploads/centos-012.jpg)

此时会进入漫长的安装过程（机器配置好的话会很快），点击【ROOT PASSWORD】设置root的密码。

![设置root的密码](/uploads/centos-013.jpg)

root密码设置完成后点击【DONE】

![设置root的密码完成](/uploads/centos-014.jpg)

然后就等吧，直到右下角出现【Reboot】，说明安装完毕！

![安装完毕](/uploads/centos-016.jpg)
