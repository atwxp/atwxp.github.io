---
title: iscroll 的坑
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

## 问题描述

给 `iscroll` 的子元素绑定 `click` 事件，在 猎豹，`chrome`, `UC` 等失效

## 原因

后来查阅 [文档](https://iiunknown.gitbooks.io/iscroll-5-api-cn/content/basicfeatures.html) 发现已经提示我们了
 
> `options.click`
>为了重写原生滚动条，`iScroll` 禁止了一些默认的浏览器行为，比如鼠标的点击。如果你想你的应用程序响应 `click` 事件，那么该设置次属性为 `true` 。请注意，建议使用自定义的 `tap` 事件来代替它（见下面）。
>默认属性：`false`


> `options.tap`
设置此属性为 `true`，当滚动区域被点击或者触摸但并没有滚动时，可以让 `iScroll` 抛出一个自定义的tap事件。
这是处理与可以点击元素之间的用户交互的建议方式。侦听 `tap` 事件和侦听标准事件的方式一致。示例如下：
```js
element.addEventListener('tap', doSomething, false); \\ Native
$('#element').on('tap', doSomething); \\ jQuery
```
> 你可以通过传递一个字符串来自定义事件名称。比如：
tap: 'myCustomTapEvent'
> 在这个示例里你应该侦听名为myCustomTapEvent的事件。
默认值：false

<!-- more -->

在源码中可以找到实现
```js 
	_initEvents: function () {
		...
		// 注册 click
		if ( this.options.click ) {
            eventType(this.wrapper, 'click', this, true);
        }
		...
		// 注册移动端事件
		if ( utils.hasTouch && !this.options.disableTouch ) {
            eventType(this.wrapper, 'touchstart', this);
            eventType(target, 'touchmove', this);
            eventType(target, 'touchcancel', this);
            eventType(target, 'touchend', this);
        }
		...
	}
 
	_start: function (e) {
		...
		// 在 touchstart 阻止浏览器触摸行为，且禁用页面滚动
		// preventDefaultException: { tagName: /^(INPUT|TEXTAREA|BUTTON|SELECT)$/ },
		// 这里没有 `a` 标签，所以点击带有 `href` 的 `a` 标签也会失效
		if ( this.options.preventDefault && !utils.isBadAndroid && !utils.preventDefaultException(e.target, this.options.preventDefaultException) ) {
            e.preventDefault();
        }
		...
	}
 
	_end: function (e) {
		...
		// we scrolled less than 10 pixels
        if ( !this.moved ) {
			// 触发 tap 事件
            if ( this.options.tap ) {
                utils.tap(e, this.options.tap);
            }
			// 开启 click，使用自定义事件派发 click
            if ( this.options.click ) {
                utils.click(e);
            }
            this._execEvent('scrollCancel');
            return;
        }
		...
	}
 
	// addEventListener() a special function called handleEvent to catch any events
	handleEvent: function (e) {
		...		
		case 'touchstart':
			this._start(e);
		case 'click':
			if ( !e._constructed ) {
                e.preventDefault();
                e.stopPropagation();
            }
            break;
	}
 
	// utils.click
    me.click = function (e) {
        var target = e.target,
            ev;
        if ( !(/(SELECT|INPUT|TEXTAREA)/i).test(target.tagName) ) {
            ev = document.createEvent('MouseEvents');
            ev.initMouseEvent('click', true, true, e.view, 1,
                target.screenX, target.screenY, target.clientX, target.clientY,
                e.ctrlKey, e.altKey, e.shiftKey, e.metaKey,
                0, null);
            ev._constructed = true;
            target.dispatchEvent(ev);
        }
    };
```

## 配置方案
 
1、开启 `click` 配置
 ```js
	myScroll = new IScroll('#wrapper', {
		mouseWheel: true,
        click: true
    });
	document.document.querySelector('#wrapper').addEventListener('click', function () {});
```

- 带有 `href` 的 `a` 可以点击
- 不会有 300ms 延迟，在 `touchend` 触发的
- 不会冒泡，被 `e._constructed` 禁用了
 
2、开启 `tap` 配置
```
	myScroll = new IScroll('#wrapper', {
		mouseWheel: true,
        tap: true
    });
	document.document.querySelector('#wrapper').addEventListener('tap', function () {});
```
- 带有 `href` 的 `a` 还是点击不了

3、和 `zepto` 使用

如果是使用 `$('').on('click')` 和 原生的 `addEventListener` 没有差别，这个时候需要开启 `click: true`
 
如果是使用 `$('').on('tap')`，配置 `click: false, tap: false` 就好了；如果配置了 `tap` 反而会触发两次 `tap` 事件

4、 `fastclick` 使用

配置 `click: false, tap: false`，某些安卓机（三星）绑定 `click` 还是点不了

配置 `click: true`，`IOS` 点击不了（双击好像可以），三星反而能点击了

5、 `zepto, fastclick` 使用

配置 `click: false, tap: false`，使用 `$('').on('click')` 在某些安卓机不能点，开启 `click: true`，`IOS` 反而不能点

配置 `click: false, tap: false`，使用 `$('').on('tap')` 好像没啥问题；如果配置了 `tap: true` 反而会触发两次 `tap` 事件

## 总结

测试机型有限，也不知道还有哪些坑（使用了 `fastlick` 好晕）。综上，如果使用 `iscroll` 想要监听内部元素的点击事件：
- 无论有没有 `fastclick`，使用 `zepto('').on('tap'）` 应该是保险的；
- 或者开启 `tap: true` 配合原生的 `addEventListener(tap)`

End.