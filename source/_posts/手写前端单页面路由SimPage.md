---
title: 手写前端单页面路由SimPage
date: 2017-03-25 19:33:49
categories: Web前端
tags:
- jQuery
---

前端我一直在用ace admin，在16年底把ace admin ajax（单页面）用在了我的博客中。  
现在我想把博客重构一遍，不准备用ace了，可是又想用它的单页面，所以就动手自己写了一个。

<!-- more -->

## simpage.js

```
/**
 * 单页面路由
 *
 * @author kangyonggan
 * @since 2017/3/21
 */
(function ($) {

    /**
     * 获取内容的url
     *
     * @param hash
     * @returns {string}
     */
    function getContentUrl(hash) {
        return window.location.origin + window.location.pathname + hash
    }

    /**
     * 加载成功
     */
    function onSuccess() {
    }

    /**
     * 加载失败
     */
    function onError() {
    }

    /**
     * 总入口
     *
     * @param contentArea
     * @param settings
     */
    function simPage(contentArea, settings) {
        var $contentArea = $(contentArea);
        var $overlay = $();//empty set
        var loadingTimer;

        // 把settings的属性合并到defaults，并且不改变defaults
        settings = $.extend(true, $.fn.simPage.defaults, settings);

        /**
         * 开始加载
         */
        function startLoading() {
            // 清除定时器
            clearInterval(loadingTimer);

            $overlay.remove();
            $overlay = $('<div>' + settings.loadingText + '</div>').css({
                position: "absolute",
                left: 0,
                top: 0,
                bottom: 0,
                right: 0,
                zIndex: 9999,
                textAlign: "center",
                marginTop: "100px"
            }).addClass("sim-page-loading");

            $contentArea.append($overlay);
            var text = $overlay.html();
            var count = 0;

            // 定时输出正在加载...
            loadingTimer = setInterval(function () {
                var temp = ".";
                for (var i = 0; i < count; i++) {
                    temp += ".";
                }
                $overlay.html(text + temp);

                count++;
                count %= 3;
            }, 1000);
        }

        /**
         * 停止加载
         *
         * @param isSuccess
         */
        function stopLoading(isSuccess) {
            // 清除定时器
            clearInterval(loadingTimer);
            $overlay.remove();

            if (isSuccess) {
                onSuccess();
            } else {
                onError();
            }
        }

        /**
         * 加载url
         *
         * @param url
         */
        function getUrl(url) {
            // 开始加载
            startLoading();

            $.ajax({
                'url': url,
                'cache': false
            }).error(function () {
                // 停止加载
                stopLoading(false);
            }).done(function (result) {
                // 停止加载
                stopLoading(true);
                // 内容替换
                $contentArea.empty().html(result);
            });
        }

        /**
         * 异步加载url
         *
         * @param hash
         * @param cache
         */
        function loadUrl(hash) {
            hash = hash.replace(/^(\#\!)?\#/, '');
            var url = settings.contentUrl(hash);

            if (typeof url === 'string') {
                getUrl(url);
            }
        };

        // 监听hash改变
        $(window).off('hashchange').on('hashchange', function (e, manual_trigger) {
            var hash = $.trim(window.location.hash);

            if (!hash || hash.length == 0) {
                return;
            }

            // 异步加载url
            loadUrl(hash);
        });

        // 是否使用默认URL
        var hash = $.trim(window.location.hash);
        if (!hash && settings.defaultUrl) {
            window.location.hash = settings.defaultUrl;
        }
        loadUrl(hash);

        // 相对定位，给【加载中...】使用
        $contentArea.css("position", "relative");
    }

    /**
     * 单页面路由
     *
     * @param option
     * @returns {*}
     */
    $.fn.simPage = function (option) {
        return this.each(function () {
            $(this).data('simPage', new simPage(this, option));
        });
    };

    /**
     * 默认配置
     *
     * @type {{defaultUrl: string, contentUrl: getContentUrl, loadingText: string, success: onSuccess, error: onError}}
     */
    $.fn.simPage.defaults = {
        // 默认URL（默认index）
        defaultUrl: '#index',
        // 内容URL（一般默认即可）
        contentUrl: getContentUrl,
        // 加载中的文字
        loadingText: '正在加载',
        // success回调方法
        success: onSuccess,
        // error回调方法
        error: onError
    }
})(window.jQuery);

```

## 用法
```
html
<div class="sim-page"></div>

js
$(".sim-page").simPage();
```

