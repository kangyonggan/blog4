---
title: ubuntu下方向键不好使的解决方案
date: 2017-03-25 17:24:28
categories: 综合
tags:
- Linux
---

### 先卸载vim-tiny

```
$ sudo apt-get remove vim-common
```

### 再安装vim full：

```
$ sudo apt-get install vim
```