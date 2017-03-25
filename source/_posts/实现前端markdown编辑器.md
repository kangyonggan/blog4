---
title: 实现前端markdown编辑器
date: 2017-03-25 17:46:19
categories: Web前端
tags:
- jQuery
---

在做网站的时候， 一帮都有后台管理可以发布一些文章或者公告之类的，一般情况下我们使用的是富文本编辑器（类似word），但我并不喜欢它， 我更喜欢markdown编辑器...

### 准备工作
- 下载bootstrap [http://v3.bootcss.com](http://v3.bootcss.com)
- 下载jquery [http://jquery.com/download](http://jquery.com/download)
- 下载marked [https://github.com/chjj/marked](https://github.com/chjj/marked)
- 下载bootstrap-markdown [http://www.codingdrama.com/bootstrap-markdown](http://www.codingdrama.com/bootstrap-markdown/)

<!-- more -->

## 使用方法
### 引入css
```
<div class="hidden ajax-append-link" rel="stylesheet" type="text/css" href="css/bootstrap.min.css">
<div class="hidden ajax-append-link" rel="stylesheet" type="text/css" href="css/bootstrap-markdown.min.css">
```

### 引入js

```
<script type="text/javascript" src="js/jquery.min.js"></script>
<script type="text/javascript" src="js/bootstrap.min.js"></script>
<script type="text/javascript" src="js/marked.min.js"></script>
<script type="text/javascript" src="js/bootstrap-markdown.min.js"></script>
```

### 文本域
```
<textarea name="content" id="content" rows="10"><textarea>
```

### 把文本域变为markdown编辑器
```
$("#content").markdown({resize: 'vertical'});
```

### 更多用法请参考：
[http://www.codingdrama.com/bootstrap-markdown](http://www.codingdrama.com/bootstrap-markdown/)