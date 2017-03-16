---
title: iphone上click事件不冒泡到document
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

## 问题

在一个带有 **展开/收起** 的列表里，使用了委托处理点击 **查看更多** 的逻辑，如下图：

![](/assets/img/more.png)

代码如下：

```js
    $(document).on('click', '.footer', function (e){
        
    });
```

这段代码在 `iphone` 上点开不了，安卓都是 `OK` 的。可以点一下 [测试DEMO](http://jsbin.com/zusapo/1/edit?html,css,js,output) 体验一下

<!-- more -->

## 为什么呢

从网上搜到几篇文章提到 [API-jQuery](http://api.jquery.com/live/) 有对 `click` 使用的提示：

> On mobile iOS (iPhone, iPad and iPod Touch) the click event does not bubble to the document body for most elements and cannot be used with .live() without applying one of the following workarounds:

> - Use natively clickable elements such as a or button, as both of these do bubble to document.
> - Use .on() or .delegate() attached to an element below the level of document.body, since mobile iOS does bubble within the body.
> - Apply the CSS style cursor:pointer to the element that needs to bubble clicks (or a parent including document.documentElement). Note however, this will disable copy\paste on the element and cause it to be highlighted when touched.

简单来说就是：

手机端的 `IOS`（`iPhone, iPad, iPod Touch`) 上，对于大多数元素来说，`click` 事件不会冒泡到 `document.body` 这个元素上，而且如果不满足下面的条件之一，不能和 `.live()/.on()` 使用:

- 使用原生可以冒泡到 `document` 的元素，如 `a，button`
- 委托事件到 `document.body` 的子元素
- 给需要冒泡 `click` 的元素或者其父元素（包括 `document.documentElement` ）设置 `cursor: pointer`，但是这样做不能 **复制/粘贴** 该元素内容，并且点击元素会高亮显示

## 解决方法

```html
<div class="wrap">
    <div class="fold">not A or button element, iphone 上 document委托失效</div>

    <div class="fold" style="cursor:pointer">set cursor: pointer</div>

    <a href="javascript:void(0);" class="fold">a element success</a>
</div>
```

1、委托到目标元素的父元素

```js
$('.wrap').on('click', '.fold', function () {
    alert('委托到目标元素的父元素, success！');
}); 
```

没啥兼容问题

2、设置目标元素 cursor: pointer
```js
$(document).on('click', '.fold', function (e) {
    alert('cursor: pointer, success！');
});
```

`iphone6` 上测试，复制粘贴没有问题，点击存在高亮

3、给body的子元素写一个click事件

参考 [这篇文章](http://www.uis.cc/content-9-380-1.html)

> 在冒泡阶段，有一个节点处理了该事件，它就不会丢弃该事件，会继续往上冒，冒到body 然后document 然后window

```js
$(document.body.children[0]).on('click', function () {
    alert('给body内其他元素绑定一个事件');
});
```

扫一扫 [Demo](http://jsbin.com/zexiqo)
![](/assets/img/click-qrcode.png)

## 总结

对比以上三种解决方法，委托到目标元素的父元素 是最简单、没有兼容性问题的方法。

End.