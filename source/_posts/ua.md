---
title: UA检测
tags: [移动web开发, 基础知识]
categories: [FrontEnd]
---

## 1、平台检测

安卓
```js
    var android = ua.match(/(Android)\s([\d.]+)/);
```
IOS
```js
    var ios = ua.match(/iPhone|iPad|iPod/);
    var ipad = ua.match(/(iPad).*OS\s([\d_]+)/);
    var ipod = ua.match(/(iPod).*OS\s([\d_]+)/);
    var iphone = !ipad && ua.match(/(iPhone)\sOS\s([\d_]+)/);
```

<!-- more -->

## 2、浏览器检测

Chrome
```js
    // chrome for ios use CriOS/ instead of Version/
    var chrome = ua.match(/(Chrome)\/([\d.]+)/) || ua.match(/(CriOS)\/([\d.]+)/);
```
Safari
```js
    var ios = ua.match(/iPhone|iPad|iPod/);
    var safari = ios && ua.match(/Version\/([\d.]+)([^S]*(Safari)|[^M]*Mobile[^S]*(Safari))/);
```
UC
```js
    var uc = ua.match(/(UCBrowser)\/([\d.]+)/);
```
QQ
```js
    var qq = ua.match(/(MQQBrowser)\/([\d.]+)/)
```
微信
```js
    var wechat = ua.match(/(MicroMessenger)\/([\d._]+)/);
```
QQIM
```js
    // QQ聊天工具内置的浏览器, 安卓既有 MQQBrowser又有 QQ/5.8.0，IOS只有 QQ/5.8.0
    var qqim = ua.match(/(QQ)\/([\d.]+)/) || ua.match(/(MQQBrowser)\/([\d.]+)/);
```
百度浏览器
```js
    var bdbox = ua.match(/baiduboxapp/);
```
手百
```js
    var baidubrowser = ua.match(/baidubrowser\/([\d\.]*)/);
```

End.