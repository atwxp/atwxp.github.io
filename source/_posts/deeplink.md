---
title: DeepLink && Universal Link
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

`webapp` 开发中最常见的一个需求就是在浏览器下尝试调起 `App`，在此总结通用的解决方案和原理。

## 1、Deep Link

> In the context of mobile apps, deep linking consists of using a URL to link to specific content within an app.

> In order for your app to respond to URL requests correctly you need to implement something called a URL scheme. You can specify the URL Scheme in your app and decide what content to display to the user once the link has been clicked on and the app opened.

### 1.1 URI Scheme

`Android 1.0` 就已经有了 `URI Scheme Deep Link`，它允许开发者给他们的 `APP` 注册一个 `URI`，然后我们在页面内可以通过一个链接如 `myapp://` 打开这个 `app`，如果没有安装会报 `page not found` 类似的错误或者没有任何反应

**不足：**
这种方法在用户安装了 `app` 的情况下，体验很好，如果没有安装 `app`，则会弹出 `页面未找到` 的错误弹框或者 **没有反应**，所以可能需要我们对这种情况降级处理

<!-- more -->

### 1.2 Android Chrome Intent

`Android Chrome 25+` 以后不在支持前面所说的 `uri scheme`，必须使用它规定的 `intent` 字符串形式，参考 [Chrome documentation](https://developer.chrome.com/multidevice/android/intents)

```html
intent://<optional path>#Intent;scheme=<URI scheme>;package=<package>;S.browser_fallback_url=<optional encoded fallback URL>
```

这种形式提供了更好的解决方案：
- Open app via URI scheme if installed
- Fall back to Play Store page if not installed
- [Optional] Specify a URL to fall back to, instead of Play Store if not installed

这就意味着对 `Android chrome 25+` 不必考虑未安装 `App` 的情况

## 2、Deferred Deep Link

`Deferred Deeplink` 先判断用户是否已经安装了 `App` 应用，如果没有则先引导至 `App` 应用商店中下载App，不同点在于用户安装 `App` 的情况下，跳转到的是指定的内容页面

Deeplink中，有几个服务可以用：`tune/branch/deepshare/LinkedMe/魔窗`

## 3、Universal Link

`Android 6.0` 支持 `app Link` ，`IOS 9` 支持 `Universal Link`

[IOS 9](https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html) 这样介绍 `Universal Link` ：

> 只要点击一个指向你网站的链接，就会直接跳到你的App的页面，无需通过Safari。如果设备上没有安装你的应用，则会在Safari中打开你的网址;
> 除了其他调用openURL的App外，只有WKWebView、UIWebView、Safari内点击的才支持跳转。像邮件、信息也是可以的;
> 这功能只支持iOS 9.0以上系统，更早的系统版本会直接在Safari打开链接

在 `Android M(6.0)` 之前，我们用的是 `uri scheme` ，这种方法会弹出一个框，询问用户选择哪个 `App` 或者提示 `xx 想打开 xx`，`Google` 在 `Android M` 上添加了自动验证的功能，避免了这个弹框，使用户直达想要的 `App`

### 3.1 为什么要这么做

- `Custom URL scheme` 因为是自定义的协议，所以在没有安装 `app` 的情况下是无法直接打开的，而 `universal links` 本身是一个 `HTTP/HTTPS` 链接，所以有更好的兼容性

- 不同的 `app` 是可以定义相同的 `ustom URL scheme` 的，所以会存在抢占或冲突的问题，而 `universal links` 是从 `server` 查询由哪个 `app` 打开的，所以不存在上述问题

- `Universal links` 支持从其他 `app` 的 `MKWebView` 或 `UIWebView` 中跳转到目标 app

- `Universal links` 本身可以被搜索引擎索引

### 3.2 怎么配置

一般都是端配置，大致是 在 Web 服务器上传 `apple-app-site-association` 的 json 文件，且能通过 `https` 访问到

我们不用关心

### 3.3 Universal Links的配置检查

[App Search API Validation Tool](https://search.developer.apple.com/appsearch-validation-tool/) 提供了验证  `universal link` 的工具，如果 `Link to application` 的状态是 `passed` 就对了

![](/assets/img/universal-pass.png)


### 3.4 产品体验

目前可以在网易新闻、糯米等很多地方看到这种技术，下图就是微信中打开网易新闻后的界面，可以看到左侧是“返回微信”右侧是 `163.com`（点击的话会在 `safari` 中打开 `universal link` 调起不成功情况下跳转的链接）

![图片](http://bos.nj.bpc.baidu.com/v1/agroup/2788523d018e5c5f25fe7698471fe9e976d385d9)

### 3.5 坑

- `universal link` 仅在 `safari/chrome` 有效，测试在 `IOS 9 UC/QQ/Chrome/Safari` 都有效

- 复制 `universal link` 到地址栏是无效的

- 因为是和 `App & webapp` 绑定的，所以 `App` 要有一个与之相关的 `webapp`(废话...)

- 在 `a` 标签写上了 `target="_blank"` 也是无效的（测试在 `chrome` 无效，所以建议不加）

- Universal Links cannot be triggered via Javascript (in window.onload or via a .click() call on an `<a>` element), unless it is part of a user action. （待测试。。。）

- 从 iOS 9.2 开始，在相同的 domain 内 Universal Links 是不work的，必须要跨域才生效，实测只需要跨子域名即可，比如 m.domain.com 跳转 o.domain.com 是可以触发的(不大晓得...)

- `Universal Links` 可以由系统来做选择，在短信或其他应用中，长按选择打开方式，若选择 `Safari` 打开，则后续的跳转会默认跳 `Safari`

- 如前面的图所示，如果用户点击了右上角的 `163.com` 链接，那么再次进入页面触发 `universal link` 不会在去调起 `App`，[stackoverflow](http://stackoverflow.com/questions/32751225/ios9-universal-links-does-not-work/32751734) 有回答：

	> Note that if a Universal Link succeeds in opening your app and then you click through to Safari (by tapping your site in the top right corner of the nav bar in app), then iOS stops opening the app when you visit that URL. Then in Safari, you can pull down to reveal a banner at the top of the page with "Open". I wasted a lot of time on this. Note that clicking through to the site => disabling UL seems path specific, based on the paths you specify in the apple-app-site-assocation file. So if you have separate routes, yoursite.com/a/* and yoursite.com/b/*, if you click yoursite.com/a/* and it opens your app directly, you then have the option in the top right corner of the app to click through to yoursite.com/a/*. If you do that, subsequent visits to yoursite.com/a/* will open in browser, not app. However, yoursite.com/b/* should be unaffected and still open your app directly.

更多的坑需要在具体实践中注意

## 4、项目中的实践

**a href/ location.href**
```html
<a href="myapp://"></a>
```
- 微信、微博无论装没装 `App` 都打不开

- 没有装 App，某些浏览器会跳到错误页，某些则是点击没有反应（符合前面描述的 `deeplink` 的问题）

一种解决方法是设置一个延时：
```js
location.href = appUrl;
setTimeout(function () {
    location.href = h5Url;
}, 1000);
```
如果能打开 `app`，那就是成功；如果没有打开点击没有反应，会在 `1s` 后自动跳转到 `h5Url`

这么写还是有些问题，如果用户切到 `App` 之后，迅速的又回到了 `h5` 页面，我们认为用户的行为就是想在 `h5` 页面浏览，那么这时候还进行 `h5Url` 的跳转是不合理的，所以可以改进一下：
```js
var t = +new Date();
location.href = appUrl;
setTimeout(function () {
    if (+new Date() - t < 600) {
        location.href = h5url;
    }
}, 500);
```
PS：这个方法只解决了点击没有反应这个问题，跳到错误页的问题没有解决

**iframe**

另一种解决方法是通过 `iframe` 跳转
```js
callIframe: function () {
    var iframe = document.createElement('iframe');
    iframe.style.display = 'none';
    iframe.style.width = 0;
    iframe.style.height = 0;

    iframe.src = this.appUrl;
    document.body.appendChild(iframe);
}
```
经过测试：
- 没有装 `App`，大多数浏览器都会跳转到 `h5Url`；safari 会弹出 “页面未找到” 的框再跳转到 `h5Url`

- 装了 `App`，基本上都可以跳到 `App`；`Android Chrome` 调起不成功，符合前面所描述的 `Android Chrome Intent` 的情况，`maybe` 改成 `intent` 形式就好了；

- 已装 `App`，调起 `App` 成功后，浏览器也会同时加载一个 `h5` 页面

`iframe` 的方法算是解决了没有装 `App` 会跳到错误页的问题。

**如何避免同时加载 h5**

- `pageshow`：页面显示时触发，在 `load` 事件之后触发。需要将该事件绑定到 `window` 上才会触发

- `pagehide`：页面隐藏时触发

- `visibilitychange`：页面隐藏没有在当前显示时触发，比如切换 `tab`，也会触发该事件

- `document.hidden`： 当页面隐藏时，该值为 `true`，显示时为 `false`

```js
document.addEventListener('visibilitychange', function () {
    var tag = document.hidden || document.webkitHidden;
    if (tag) {
    }
}, false);

window.addEventListener('pagehide', function () {
}, false);
```

然而不幸的是，不是所有浏览器都会支持这些事件，所以这个恐怕暂时避免不了。

**IOS9的升级**

`IOS 9` 开始不支持 `iframe` 调起，点击没有反应，需要改为 `location.href` 调起，同时还做了下面的升级：

> What happens now is that in iOS 9, Apple changed the 'Open App' modal from a Javascript blocking modal to a non-blocking modal. 

所以和 `IOS9-` 表现不一样的是 `XX 想打开 XX/页面未找到` 的模态框不会阻塞 `js` 执行了。

- 如果没有安装 `App`，就会弹出 “页面未找到” 的框；无效的弹窗提示在用户体验上是不允许出现的，但是线上产品普遍存在这个弹框的问题无法解决；但是在 `IOS9` 中会发现如果用户不进行操作，会自动跳转到 `h5Url` 了

- 安装了 `App`，也会弹框提示 “XX 想打开 XX” （感觉 `IOS9` 就是把 `iframe` 换成了 `location.href`，之所以这样做是为了支持 `universal link`）如果用户不点弹窗的确认按钮，会发现页面会自动跳转到 h5Url 了

**微信的坑**

微信默认屏蔽了 `App` 的 `Scheme` 跳转，但是有些 `App` 是可以跳转的：

- 大众点评，小红书，BiliBili，京东等

- 豆瓣等(`universal link`)

参考 [微信API－－WXAppExtendObject](http://www.jianshu.com/p/3c2a0ba09ac8)

## 5、测试 demo

[Demo](http://jsbin.com/tujeyugoxe/5)

![](/assets/img/universal-qrcode.png)

## Refer
- [iOS 9.2 Update: The Fall of URI Schemes and the Rise of Universal Links](https://blog.branch.io/ios-9.2-redirection-update-uri-scheme-and-universal-links)
- [iOS/Android 浏览器(h5)及微信中唤起本地APP](http://www.magicwindow.cn/blog/posts/000035.html)

End.