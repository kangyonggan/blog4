---
title: Nginx报错：413 Request Entity Too Large
date: 2017-03-25 16:31:14
categories: 综合
tags:
- Nginx
---

### 解决方案
在http模块下添加配置:

```
http {
    ...
    client_max_body_size 10m;
    ...
}

```

