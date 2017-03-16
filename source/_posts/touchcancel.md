---
title: touchcancel的使用
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

通常我们常用的手势事件有 `touchstart, touchmove, touchend`，对 `touchcancel` 用之甚少，仅知道就是由于系统原因导致的手势中断而触发的事件。

在一个项目中我就用到了这个事件 `touchcancel`，有这么个需求：需要监听长按下载这个动作（浏览器默认支持长按下载，同时会弹出一个系统弹框）

很自然我们就想到了监听 `touchstart/touchend`，在 `touchstart` 记录起始点/时间，在 `touchend` 记录终止点/时间，满足长按要求则触发事件；

<!-- more -->

在实际测试过程中，有些浏览器并不触发 `touchend` 事件，这时候想到了 `touchcancel` ，猜测应该是这些浏览器弹出下载浮层属于系统中断 `touch` 事件，经测试猜想是正确的。

结论：某些浏览器下（`UC/QQ...`）系统弹层会阻止当前 `touch` 事件的发生，需要同时监听 `touchend/touchcancel`，代码如下：
```js
    define(function (require, exports, module) {

        var util = require('util');

        /**
         * init
         *
         * @param {DOM}         wrapper longtap'wrapper
         * @param {Object}      options options
         * @property {number}   options.thresold 阈值
         * @property {number}   options.allowedTIme 允许时间
         * @param {Function}    callback 回调函数
         */
        var init = function (wrapper, options, callback) {
            var settings = {
                thresold: 5,

                allowedTime: 300
            };

            if (!wrapper) {
                return;
            }

            if (typeof options === 'function') {
                callback = options;

                options = {};
            }

            callback = callback || function () {};

            util.extend(settings, options);

            var startX;
            var startY;
            var startTime;

            wrapper.addEventListener('touchstart', function (e) {
                var touchobj = e.changedTouches[0];

                startX = touchobj.pageX;

                startY = touchobj.pageY;

                startTime = +new Date();
            }, false);

            wrapper.addEventListener('touchend', function (e) {
                e.stopPropagation();
                e.preventDefault();

                var elapsedTime = +new Date() - startTime;

                var touchobj = e.changedTouches[0];
                var endX = touchobj.pageX;
                var endY = touchobj.pageY;

                if (
                    Math.abs(endX - startX) < settings.thresold
                    && Math.abs(endY - startY) < settings.thresold
                    && elapsedTime > settings.allowedTime
                ) {
                    callback();
                }
            }, false);

            wrapper.addEventListener('touchcancel', function () {
                callback();
            }, false);
        };

        exports.init = init;
    });
```
End.
