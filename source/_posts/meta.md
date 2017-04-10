---
title: meta 元素的使用
tags: [移动web开发, 基础知识]
categories: [FrontEnd]
---

# meta 标签

### viewport

设置布局视口为设备宽度，不允许用户缩放，在网页加载时隐藏地址栏与导航栏
```html
    <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no, minimal-ui" />
```

- `width`：`layout viewport` 的宽度


- `height`：`layout viewport` 的高度


- `initial-scale`：初始的缩放比例


- `maximum-scale`：允许用户缩放到的最大比例


- `user-scalable`：用户是否可以手动缩放


- `minimal-ui`：`iOS 7.1 Safari` 为 `meta` 标签新增 `minimal-ui` 属性，在网页加载时隐藏地址栏与导航栏

**PS: iOS 8 中移除了 'minimal-ui'**

### format-detection

`iPhone` 下会自动将一连串数字转换为拨号超链接，`email`则会调起邮件系统，去掉自动识别：
```html
<meta name="format-detection" content="telephone=no">
<meta name="format-detection" content="email=no">
<!-- or -->
<meta name="format-detection" content="telephone=no, email=no">
```
<!-- more -->

### apple-touch-icon

设置 `apple-touch-icon` 属性，在 `safari` 上可以使用 **添加到主屏幕** 将页面添加到主屏幕上，方便用户以后快速访问
```html
    <link rel="apple-touch-icon" sizes="120*120" href="/your-icon.png"/>
```
`sizes` 适配不同的设备，没有设置的话，默认大小为 `60 * 60`

- `180 * 180`：`iphone6@3x`

- `76  * 76`：`ipad 2 and ipad mini (@1x)`

- `120 * 120`：`iphone4, 5, 6@2x`

- `152 * 152`：`ipad and ipad mni (@2x)`

图标搜索的优先级如下:

- 没有和相应设备推荐尺寸一致的图标，优先使用比推荐尺寸大，但是最接近推荐尺寸的图标
- 如果没有比推荐尺寸大的图标， 选择尺寸最大的图标
- 如果link标签没有指定图标，则在根目录寻找以 `apple-touch-icon...` 为前缀命名的图标

PS：`IOS7` 开始设备图标的推荐尺寸变化了：

    default: 57 * 57 => 60 * 60

    iphone retina: 114 * 114 => 120 * 120

    ipad retina: 144 * 144 => 152 * 152

    ipad with no retina: 72 * 72 => 76 * 76

总结下来：
```html
<!-- For Chrome for Android: -->
<link rel="icon" sizes="192x192" href="touch-icon-192x192.png">

<!-- For iPhone 6 Plus with @3× display: -->
<link rel="apple-touch-icon-precomposed" sizes="180x180" href="apple-touch-icon-180x180-precomposed.png">

<!-- For iPad with @2× display running iOS ≥ 7: -->
<link rel="apple-touch-icon-precomposed" sizes="152x152" href="apple-touch-icon-152x152-precomposed.png">

<!-- For iPad with @2× display running iOS ≤ 6: -->
<link rel="apple-touch-icon-precomposed" sizes="144x144" href="apple-touch-icon-144x144-precomposed.png">

<!-- For iPhone with @2× display running iOS ≥ 7: -->
<link rel="apple-touch-icon-precomposed" sizes="120x120" href="apple-touch-icon-120x120-precomposed.png">

<!-- For iPhone with @2× display running iOS ≤ 6: -->
<link rel="apple-touch-icon-precomposed" sizes="114x114" href="apple-touch-icon-114x114-precomposed.png">

<!-- For the iPad mini and the first- and second-generation iPad (@1× display) on iOS ≥ 7: -->
<link rel="apple-touch-icon-precomposed" sizes="76x76" href="apple-touch-icon-76x76-precomposed.png">

<!-- For the iPad mini and the first- and second-generation iPad (@1× display) on iOS ≤ 6: -->
<link rel="apple-touch-icon-precomposed" sizes="72x72" href="apple-touch-icon-72x72-precomposed.png">

<!-- For non-Retina iPhone, iPod Touch, and Android 2.1+ devices: -->
<link rel="apple-touch-icon-precomposed" href="apple-touch-icon-precomposed.png"><!-- 57×57px -->
```
上面的代码我们使用了 `app-touch-icon-precomposed` 属性，和 `app-touch-icon` 的区别在于：**前者添加的是设计原图，不带有高光渐变效果，后者则是会带有 `IOS` 统一的高光效果**

PS：判断用户是否是“将网页添加到主屏后，再从主屏幕打开这个网页”

```
navigator.standalone
```

### app-touch-startup-image

前面我们成功的添加页面到主屏幕上，然后我们还可以设置 `app-touch-startup-image`，即一个类似 `NativeApp` 的启动画面。

```html
    <link rel="apple-touch-startup-image" href="startup-image.png" />
```

`iphone3, 4, 5, 6`只支持`竖屏模式`，而 `iPhone 6 Plus` 支持横屏，`iPad`有横屏竖屏。

`Apple` 官方文档建议竖屏模式的 `iPhone 3， 4` 启动动画的大小 `320 * 460`，之所以少了 `20px`，我们很容易想到是 `IOS` 状态栏的高度大小。对于 `retina` 屏幕，我们需要准备 `640 * 920` 大小的图片。不同于 `app-touch-icon`，启动画面没有`sizes`属性。

```html
<!-- iPhone -->
<link rel="apple-touch-startup-image" media="(device-width: 320px)" href="apple-touch-startup-image-320x460.png">

<!-- iPhone (Retina) -->
<link rel="apple-touch-startup-image" media="(device-width: 320px) and (-webkit-device-pixel-ratio: 2)" href="apple-touch-startup-image-640x920.png">

<!-- iPhone 5 (Retina) -->
<link rel="apple-touch-startup-image" media="(device-width: 320px) and (device-width: 568px) and (-webkit-device-pixel-ratio: 2)" href="apple-touch-startup-image-640x1096.png">

<!-- iPhone 6 (retina) -->
<link rel="apple-touch-startup-image" media="(device-width: 375px) and (-webkit-device-pixel-ratio: 2)" href="apple-touch-startup-image-750x1294.png">

<!-- iPhone 6+ (portrait )-->
<link rel="apple-touch-startup-image" media="(device-width: 414px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)" href="apple-touch-startup-image-1242x2148.png">

<!-- iPhone 6+ (landscape) -->
<link rel="apple-touch-startup-image" media="(device-width: 414px) and (-webkit-device-pixel-ratio: 3) and (orientation: landscape)" href="apple-touch-startup-image-1182x2208.png">

<!-- iPad (portrait) -->
<link rel="apple-touch-startup-image" media="(device-width: 768px) and (orientation: portrait)" href="apple-touch-startup-image-768x1004.png">

<!-- iPad (landscape) -->
<link rel="apple-touch-startup-image" media="(device-width: 768px) and (orientation: landscape)" href="apple-touch-startup-image-748x1024.png">

<!-- iPad (Retina, portrait) -->
<link rel="apple-touch-startup-image" media="(device-width: 768px) and (orientation: portrait) and (-webkit-device-pixel-ratio: 2)" href="apple-touch-startup-image-1536x2008.png">

<!-- iPad (Retina, landscape) -->
<link rel="apple-touch-startup-image" media="(device-width: 768px) and (orientation: landscape) and (-webkit-device-pixel-ratio: 2)" href="apple-touch-startup-image-1496x2048.png">
```
### apple-mobile-web-app-capable

删除默认的 `IOS` 工具栏和菜单栏式，即开启 `webapp` 全屏模式
```html
    <meta name="apple-mobile-web-app-capable" content="yes">
```
PS：当页面添加到主屏幕后再点击进行启动时有效，从浏览器跳转或输入链接进入并没有此效果

### apple-mobile-web-app-status-bar-style

设置 `IOS` 系统状态栏风格
```html
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
    <!-- or -->
    <meta name="apple-mobile-web-app-status-bar-style" content="black" />
```
`content` 参数的值：

- `default`：默认值白色

- `black`：黑色

- `black-translucent`：半透明

如果设置为 `default` 或者 `black`，页面内容从状态栏底部开始；如果设置为 `black-translucent`，页面内容充满整个屏幕，所以页面内容会被状态栏遮挡。

![Alt text](/assets/img/statusbar-black.png) ![Alt text](/assets/img/statusbar-blacktrans.png)

PS：只有在 `<meta name="apple-mobile-web-app-capable" content="yes"> `时生效

### 添加到主屏后的标题

    <meta name="apple-mobile-web-app-title" content="百度糯米">

# 其他

1、IOS上长按`链接`或者`图片`会默认弹出系统菜单，`-webkit-touch-callout` 可以禁止这个菜单的弹出（菜单默认是开启的），安卓不起作用
```less
a,
img {
    -webkit-touch-callout: none;
}
```
2、禁止选择文本（如果没有选择文本需要，建议最好加上）
```less
html,
body {
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
```
3、避免屏幕旋转导致字体大小缩放
```less
body {
    -webkit-text-size-adjust: 100%;
}
```
4、更改`IOS` 可点击元素的高亮颜色：当透明度设为0，则会禁用此属性；当透明度设为1，元素在点击时不可见；除了`iOS Safari`，大部分 `Android` 手机也是支持的

```less
body {
    -webkit-tap-highlight-color: rba(255, 255, 255, 0);
}

```
5、隐藏地址栏

```javascript
setTimeout(function (){
    window.scrollTo(0, 1);
}, 0);
```

6、开启电话、短信功能

 ```html
// 电话
<a href="tel:10086">10086</a>

// 短信
<a href="sms:10086">10086</a>

// 邮箱
<a href="mailto:example@gmai.com">example@gmail.com</a>
```

7、判断屏幕方向
```js
switch (window.orientation) {
    case -90:
    case 90:
        alert('横屏:' + window.orientation)
        break

    case 0:
    case 180:
        alert('竖屏:' + window.orientation)
        break

    default:
    }
```
8、关闭 `IOS` 输入框首字母大写

```html
<input type="text" autocapitalize="off" />
```

9、关闭 `IOS` 输入自动修正

```html
<input type="text" autocorrect="off" />
```
10、-webkit-appearance

>  display an element using a platform-native styling based on the operating system's theme.

```less
/* Partial list of available values in Gecko */
-moz-appearance: none;
-moz-appearance: button;
-moz-appearance: checkbox;
-moz-appearance: scrollbarbutton-up;

/* Partial list of available values in WebKit/Blink */
-webkit-appearance: none;
-webkit-appearance: button;
-webkit-appearance: checkbox;
-webkit-appearance: scrollbarbutton-up;
```
通常我们使用  `none` 重置表单外观

11、修改 placeholder 文字颜色

```less
::-webkit-input-placeholder {
    color: #f00;
}
/* firefox 18- */
:-moz-input-placeholder {
    color: #f00;
}
/* firefox 19+ */
::-moz-placeholder {
    color: #f00;
}
/* ie 10+ */
:-ms-input-placeholder {
    color: #f00;
}
```
# Refer

- [`<head>` Cheat Sheet](https://github.com/joshbuchea/HEAD)
- [specifying a webpage icon for web clip](https://developer.apple.com/library/safari/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html)
- [Meta Tags](https://developer.apple.com/library/safari/documentation/AppleApplications/Reference/SafariHTMLRef/Articles/MetaTags.html)
- [Everything you always wanted to know about touch icons](https://mathiasbynens.be/notes/touch-icons)
- [ios-web-app-icons-and-startup-images/](http://taylor.fausak.me/2012/03/27/ios-web-app-icons-and-startup-images/)
- [WebApp化（二）apple-touch-startup-image](http://www.motype.org/post/design/apple-touch-startup-image)
- [移动Web开发技巧汇总](http://www.html-js.com/article/2983)
- [what-size-should-apple-touch-icon-png-be-for-ipad-and-iphone-4](http://stackoverflow.com/questions/2997437/what-size-should-apple-touch-icon-png-be-for-ipad-and-iphone-4)

End.
