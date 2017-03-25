---
title: 使用ace admin搭建单页面网站
date: 2017-03-25 18:29:14
categories: Web前端
tags:
- jQuery
---

### 资料
- Ace Admin [ace.zip](/uploads/ace.zip)
- 在线实例（已经没了） [http://kangyonggan.com](http://kangyonggan.com)
- Github代码（已经没了） [https://github.com/kangyonggan/blog.git](https://github.com/kangyonggan/blog.git)

> 网上很少有ace admin相关的资料，如需使用和学习，只能去看源代码...

<!-- more -->

### 目的
做出一个类似cms后台管理系统的单页面网站，效果如下:

![加载中](/uploads/20170109223706060.png)

![地址](/uploads/20170109223708522.png) 

### 步骤
由于关于ace admin的文档很少，尤其是ace admin ajax的！所以这就要我们自己来读源代码，看实例来学习了。

#### 搭建本地demo
下载ace.zip并解压
![下载](/uploads/20170109223701593.png) 

拷贝到tomcat webapps目录下

![拷贝到tomcat](/uploads/20170109223709211.png) 

启动tomcat后，打开浏览器即可查看demo

![查看demo](/uploads/20170109223706510.png) 

可以先进入`Ajax Demo Pages`提前感受一下单页面的好处。

#### 拷贝源码+修改
查看源码：右键-->查看源代码

![源码](/uploads/20170109223707758.png)

把ace的一些核心css和js拷贝到你的html中，由于我们是要做单页面，从`Ajax Demo Pages`中可以看出，关键词是`ajax`  
所以你需要格外的关注源码中出现的`ajax`,经过搜索后发现有用的是下面两点

![ajax](/uploads/20170109223702751.png)
![ajax](/uploads/20170109223704068.png)

点开`ace.ajax-content.js`, 如下：

![ajax-content](/uploads/20170109223702338.png)

发现他是一个典型的jquery插件（不会jquery插件的看着会比较难，最好先去恶补一下），大致看一下这个插件，会发现它实现单页面的核心代码是:

![geturl](/uploads/20170109223704715.png)

当用户点击超链接或者按钮时，插件代替你发起异步请求，服务器返回一个“页面”，插件拿到页面后，用下面红框框的核心代码把内容局部清空并替换，实现类似iframe的效果，给人一种我是单页面的错觉

![replace](/uploads/20170109223707144.png)

对源码有个大致的了解就行，下面就可以动手搭建了。

### 解读源码
基础web环境搭建过程不再演示，我使用的freemarker模板，下面是我的html布局：

```html
<#assign ctx="${(rca.contextPath)!''}">

<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
    <meta charset="utf-8"/>
    <meta name="description" content=""/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0"/>

    <div class="hidden ajax-append-link" rel="shortcut icon" href="${ctx}/static/app/images/favicon.ico" type="image/x-icon">
    <!-- bootstrap & fontawesome -->
    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/ace/dist/css/bootstrap.min.css"></div>
    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/ace/dist/css/font-awesome.min.css"></div>
    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/ace/dist/css/jquery.gritter.min.css"></div>

    <!-- page specific plugin styles -->

    <!-- text fonts -->
    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/ace/dist/css/ace-fonts.min.css"></div>

<#--skin-->
    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/ace/dist/css/ace-skins.min.css"></div>

    <!-- ace styles -->
    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/ace/dist/css/ace.min.css" class="ace-main-stylesheet"
          id="main-ace-style"></div>

    <!--[if lte IE 9]>
    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/ace/dist/css/ace-part2.min.css" class="ace-main-stylesheet"></div>
    <![endif]-->

    <!--[if lte IE 9]>
    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/ace/dist/css/ace-ie.min.css"></div>
    <![endif]-->

    <div class="hidden ajax-append-link" rel="stylesheet" href="${ctx}/static/app/css/app.css"></div>

    <script src="${ctx}/static/ace/dist/js/jquery.min.js"></script>

    <!--[if lte IE 8]>
    <script src="${ctx}/static/ace/dist/js/html5shiv.js"></script>
    <script src="${ctx}/static/ace/dist/js/respond.min.js"></script>
    <![endif]-->
<@block name="app-style"/>
</head>
<body class="skin-3">
<#include "navbar.ftl"/>

<div class="main-container" id="main-container">

<#if hasSidebar?? && hasSidebar==true>
    <#include "sidebar.ftl"/>
</#if>

    <div class="main-content">
        <div class="main-content-inner">
        <#if home_name?? && home_name!=''>
            <div class="breadcrumbs" id="breadcrumbs">
                <ul class="breadcrumb">
                    <li>
                        <i class="ace-icon fa fa-tachometer home-icon"></i>
                        <a data-url="index" href="#index">${home_name}</a>
                    </li>
                </ul>
            </div>
        </#if>

            <div class="page-content">
                <div class="page-content-area"></div>
            </div>
        </div>
    </div>

<#include "footer.ftl"/>

<#include "modal.ftl"/>

    <a href="javascript:" id="btn-scroll-up" class="btn-scroll-up btn btn-sm btn-inverse">
        <i class="ace-icon fa fa-angle-double-up icon-only bigger-110"></i>
    </a>
</div>

<script>var ctx = '${ctx}';</script>
<script src="${ctx}/static/ace/dist/js/bootstrap.min.js"></script>
<script src="${ctx}/static/libs/jquery/jquery.bootstrap.min.js"></script>
<script src="${ctx}/static/ace/dist/js/jquery.gritter.min.js"></script>
<script src="${ctx}/static/ace/dist/js/ace-extra.min.js"></script>
<script src="${ctx}/static/ace/dist/js/ace-elements.min.js"></script>
<script src="${ctx}/static/ace/dist/js/ace.min.js"></script>
<script src="${ctx}/static/app/js/app.js"></script>
<@block name="app-script"/>
</body>
</html>
```

其中，`navbar.flt` `sidebar.ftl` `footer.ftl` 都是demo中的。  
如果之前使用的就是freemarker+ace admin，那么你要重点关注下面两处代码:

```html
<div class="page-content">
   <div class="page-content-area"></div>
</div>
```

核心js`app.js`

```js
$(function () {
    // 异步加载界面
    var $ajaxContent = $(".page-content-area");
    $ajaxContent.ace_ajax({
        'default_url': '#index',
        'content_url': function (hash) {
            return window.location.origin + window.location.pathname + "/" + hash;
        },
        'update_active': updateMenuActive,
        'update_breadcrumbs': updateBreadcrumbs,
        'update_title': updateTitle,
        'loading_text': '<span class="loading">正在加载, 请稍等...</span>'
    });

    // 监听异步加载失败事件
    $ajaxContent.on("ajaxloaderror", function (e, data) {
        window.location.href = ctx + '/#500';
    });
});


/**
 * 更新菜单激活状态
 *
 * @param hash
 */
function updateMenuActive(hash) {
    //  当前菜单
    var $menu = $($('a[data-url="' + hash + '"]')[0]).parent("li");

    // 所有菜单
    var $all_menus = $menu.parents("ul.nav-list").find("li");

    // 清除所有菜单状态
    $all_menus.removeClass("open");
    $all_menus.removeClass("active");

    // 父菜单
    var $parent = $menu.parents("li");
    if ($parent.length > 0) {
        $parent.addClass("open");
    }
    $menu.addClass("active");
}

/**
 * 更新面包屑
 *
 * @param hash
 */
function updateBreadcrumbs(hash) {
    var $menu = $('a[data-url="' + hash + '"]');

    // 下面这坨代码摘自ace.ajax-content.js
    var $breadcrumbs = $('.breadcrumb');
    if ($breadcrumbs.length > 0 && $breadcrumbs.is(':visible')) {
        $breadcrumbs.find('> li:not(:first-child)').remove();

        var i = 0;
        $menu.parents('.nav li').each(function () {
            var $link = $(this).find('> a');

            var $link_clone = $link.clone();
            $link_clone.find('i,.fa,.glyphicon,.ace-icon,.menu-icon,.badge,.label').remove();
            var text = $link_clone.text();
            $link_clone.remove();

            var href = $link.attr('href');

            if (i == 0) {
                var li = $('<li class="active"></li>').appendTo($breadcrumbs);
                li.text(text);
            } else {
                var li = $('<li><a ></a></li>').insertAfter($breadcrumbs.find('> li:first-child'));
                li.find('a').attr('href', href).text(text);
            }
            i++;
        })
    }
}

/**
 * 更新标题
 *
 * @param hash
 */
function updateTitle(hash) {
    var $menu = $($('a[data-url="' + hash + '"]')[0]);
    var title = $.trim($menu.text());

    if (title != '') {
        document.title = title;
    }
}

/**
 * 更新状态
 *
 * @param hash
 */
function updateState(hash) {
    updateBreadcrumbs(hash);
    updateMenuActive(hash);
    updateTitle(hash);
}
```

如果你对jquery插件了解的不多，估计很难去使用`ace_ajax`，因为网上和demo中都没有使用教程，我这是根据`ace.ajax-content.js`源代码中的jquery插件反推出来的插件使用方法。

在插件源代码的最后暴露出插件有哪些公共属性和方法可被你覆写和调用

```js
$.fn.aceAjax = $.fn.ace_ajax = function (option, value, value2, value3) {
 var method_call;

 var $set = this.each(function () {
 var $this = $(this);
 var data = $this.data('ace_ajax');
 var options = typeof option === 'object' && option;

 if (!data) $this.data('ace_ajax', (data = new AceAjax(this, options)));
 if (typeof option === 'string' && typeof data[option] === 'function') {
 if(value3 != undefined) method_call = data[option](value, value2, value3);
 else if(value2 != undefined) method_call = data[option](value, value2);
 else method_call = data[option](value);
 }
 });

 return (method_call === undefined) ? $set : method_call;
}

$.fn.aceAjax.defaults = $.fn.ace_ajax.defaults = {
 content_url: false,
 default_url: false,
 loading_icon: 'fa fa-spin fa-spinner fa-2x orange',
 loading_text: '',
 loading_overlay: null,
 update_breadcrumbs: true,
 update_title: true,
 update_active: true,
 close_active: false,
 max_load_wait: false
}
```

比如`loading_text`,看名称就大概猜出是在异步加载界面时，给用户的提示信息，例如`正在加载, 请稍等...`， 当然这只是大概的猜测，我们要真想使用它，肯定要读相应的源代码并且动手去验证一下。

过程中难免会遇到各种问题，只要耐心解读源码，一定会攻破的，我也遇到了好多坑，昨天花了一整天的时间才把我原本的博客给变成了单页面的，这主要是因为我的聪明才智和之前就使用的ace admin。

> 温馨提示: `ace.ajax-content.js`文件被包含在了`ace.min.js`中，因此只需要引入`ace.min.js`即可




