---
title: High Performance Mobile Web 101
tags: [移动web开发, 性能优化]
categories: [FrontEnd]
---

最近读了一本关于移动web开发性能优化方面的书 [《High Performance Mobile Web》](https://book.douban.com/subject/26584301/)，这本书系统全面的介绍了性能优化的知识点，对我来说至少以后再回答 “你是怎么做性能优化的？” 这个问题，不会再简单罗列诸如懒加载图片、缓存 js/css、按需加载这些了，而是会从不同的方面系统的给出优化点。接下来的系列会对这本书做个简单的总结。

首先谈谈移动设备和PC之间的不同，包括硬件、网络、操作系统、引擎、webview等方面。

<!-- more -->

## 1、硬件条件

现在市场上有很多种类型的移动设备，诸如平板（ipad...)、智能手机(iPhone, HTC...)、可穿戴设备（iwatch...)，对于web开发来说，这些移动设备和PC存在着很大的差别如分辨率，最直观的感受就是屏幕大小不一样，从性能优化的角度来说，不同点在于：

- CPU：核心工厂，解析下载执行的地方
- 内存：存储DOM树、图片、数据等
- GPU：开启硬件加速渲染
- GPU 内存：存储GPU渲染的层、图片、数据等
- ...

比如我自己的 `Mac Pro` 的内存是 `8GB`，`iPhone 6s` 的内存是 `2GB`

## 2、网络情况

上网的途径无非就是 WiFi 或者数据流量，WiFi 可以是稳定的如家庭、工作网络，也可以是不稳定的如热点，流量的话更不用说了，偏僻的地方手机都没有信号。

平时我们谈到流量都是说 2G、3G、4G，这些名字是什么意思呢？你可能会猜到 `G` 就是 `Generation` 就是 `代` 的意思，所谓 `2G` 就是第二代XXX，`3G` 就是第三代XXX。嗯，事实也就是这样，除了这些名字之外，还有其他名字表示：

- GPRS (2G)
- EDGE (2.5G, 3G)
- UMTS (3G)
- WCDMA (3G)
- LTE (4G)
- ...

虽然身边的人大都是 `4G` 用户，但是移动网络是不稳定的，看到是 `4G` 信号也不表示用的是 `4G`，很可能降到了 `3G`，更何况我这种之前一直使用 `3G` 的用户，在地铁上就没打开过网页。。。

PS：[The State of LTE (November 2016)](https://opensignal.com/reports/2016/11/state-of-lte) 可以看到全球 4G 的使用率

网络有带宽、延迟的特性，在移动开发中，我们是否也需要关注这些呢？

### 带宽

带宽固然也大越好，但是相比较我们传输的文本文件的大小，似乎也没有很大的影响，除非你要求实时传输，做视频音频流这些对带宽具有高要求的需求

### 延时

计算机网络有一个概念：RTT(Round-Trip-Time)，表示从发送端发送数据开始，到发送端收到来自接收端的确认，总共经历的时延。不管怎么样延迟是避免不了的，这是因为手机连到网络上需要经过发射塔、运营商网关、服务器，然后取得数据之后再反向走一遍。

PS：书中提到 `2G` 下有 `1s` 的延时，`4G` 下有 `180ms` 的延时

### radio rate

手机使用无线电和发射塔通信，无线电的使用会消耗电量，所以操作系统会尽量减少它的使用，这意味着如果没有 `App` 使用网络，手机会把无线电状态从 `active` 切换为 `idel`，当有软件请求数据时，会重启无线电，这个过程也花费时间

PS：书中提到 `3G` 下这个时间可以达到 `2.5s`，`4G` 下通常少于 `100ms`

## 3、操作系统

市场上主流的手机操作系统有 `iOS, Android, Windows`，平时开发我们只会考虑前两者。此外还有一些其他操作系统，比如 `BlackBerry, Firefox OS, Symbian...`，一般不考虑这些。

`PC` 上的话，主流的有 `Windows, MacOS, Linux, Ubuntu...`。

PC：相比较移动设备，PC开发的时候一般不会去判断系统类型

## 4、引擎

引擎分为渲染引擎和 js 执行引擎。

### 4.1 渲染引擎

渲染引擎（布局引擎）是用来下载、解析、渲染 HTML、CSS的，包括相关的内容如SVG、图片。

目前主流的操作系统如 `iOS、Android...` 等都是使用 `webkit` 渲染引擎。除了 `webkit`，还有其他的渲染引擎可供选择如 `Blink, Trident, Gecko, Presto`：

- `webkit` 是 `Apple` 基于 `KHTML` 创建并使用在 `safari` 中的一个开源项目，引擎中和渲染布局相关的代码叫做 `WebCore`。

- `blink` 是 `Google` 基于 `webkit` 开发的，目前 `chrome, opera` 都在用 `blink` 引擎。

- `Trident` 是 `IE` 专有的引擎，自从 `windows 10` 以后，微软又基于 `Trident` 重写了一个引擎 `Edge`

- `Gecko` 是 `Mozilla` 维护的开源引擎

- `Presto` 是 `opera` 特有的引擎，自从 `opera 12` 后就切换为 `blink` 了

### 4.2 执行引擎

执行引擎是一个解释执行代码、管理内存的运行时环境。有些引擎会包含 `Just-In-Time(JIT)` 编译有些则不会，那他们有什么差异？

没有 `JIT` 编译的引擎会边执行 `JS` 代码边解释，而有 `JIT` 的会在执行之前编译为机器码，如果再次执行相同的代码无需再翻译直接执行机器码就好了。

#### javaScriptCore

`JavaScriptCore` 引擎是 `WebKit` 中默认的引擎，在早期阶段只有解释器来解释执行 `JavaScript` 代码，性能不是特别突出，后来经过优化产出了 `Nitro`，也正是 `safari` 浏览器所使用的

### V8

`Google` 开发的开源js引擎，V8在执行之前会把 `js` 代码编译为机器码从而提高效率，更进一步使用了内联缓存等优化性能。

`V8` 是 `Google Chrome` 浏览器内置的`JavaScript` 脚本引擎 同时也是 `Node.js` 支持的引擎

其他主流的 `JS` 引擎：`IE` 的 `Chakra`，`Mozilla` 的 `monkey` 系列 `JaegerMonkey, IonMonkey...`

## 5、web 平台

web平台指的是我们编写的web应用在哪里被解析执行，换句话说就是指它的宿主环境。

平时开发最经常打交道的环境有两种：浏览器和app，一个是直接在浏览器地址栏打开能访问的到，一个是嵌套了app的外壳，即 webview 的方式

### 5.1 browser

浏览器是最熟悉不过了，从PC开始就已经和它打交道了，Chrome、Firefox、Safari、IE。。。然而在移动设备上浏览器会更加五花八门

现在的手机都会自带一个浏览器出厂，称为 `stock browser/preinstalled browser`，这个浏览器的问题在于如果我们不更新手机系统，它就不会更新。

`iOS` 系统上的浏览器会比较简单点，它的预装浏览器是 safari，如果不是开发人员的话，很少会再去装其他的浏览器。如果装其他浏览器的话如 `chrome, firefox, uc...`，其实和 `safari` 都是使用一样的引擎，这是因为 `iOS` 系统的封闭性，第三方浏览器只能优化功能、定制UI等，所以整体上 `iOS` 上的 `bug` 会比较简单一致

`Android` 就不一样了，它是一个开源的操作系统，`4.3` 之前使用的是 `webkit` 的 `webview`，`4.4` 之后就转为基于 `chromium` 的 `webview` 了，也就是使用 `blink/v8` 渲染了。

因为 `Android` 自带的浏览器性能并不好，就导致各个厂商、浏览器厂商开发自己的浏览器，有的会使用 `webkit` 有的又使用 `chromium`，这就导致渲染会存在差异，修改了一个手机的问题，另一个手机可能还存在问题。

### 5.2 webview

`webview` 是一个类似于按钮、输入框等的原生控件，它允许原生 `App` 包含并执行 `web content`，`webview` 可以使用整个屏幕或者一部分。`webview` 能做些什么？

- 执行 js 代码
- 显示富文本内容
- 创建完整的用户界面
- 创建一个 in-app browser
- 创建一个 pseudo-browser
- ....

#### in-app browservs pseudi browser

什么是 `in-app browser` ？有些原生App想展示一个页面，但是并不希望你离开App，就在 app 内部提供了一个渲染环境。比如在 `evernote` 内打开一个链接，并没有调用 `safari` 浏览器

`pseudo-browser` 是一个被认为是浏览器的原生应用，但是它没有自己的渲染、执行引擎，使用的是系统原生的 `web view`。从用户的角度来说它是个浏览器，从开发者角度来说，它就是一个具有特殊 `UI` 的 `webview`。`iOS Chrome` 就是这样一个  `pseudo-browser`

他俩的区别在于 `in-app browser` 场景下的 app 并不是一个浏览器，你不能说evernote是个浏览器，即便它在app内部打开了一个页面

#### iOS

`iOS 8` 之后，有两个 `webview` 可以用：`UIWebView / WKWebView`

#### Android

`Android` 使用的是基于 `webkit` 的 `webview`，`Android 2.x` 性能非常慢，虽然在 `4.0 - 4.3` 有所好转，但是在性能兼容性上并没有显著的提升。所以很多厂商都决定使用新版本的 `webkit` 或者 `chromium` 替换默认的 webview ，这就导致每个设备、操作系统对应的 webkit/chromium 的版本可能不一致

在 `Android 4.4` 之后，`Google` 决定使用 `chromium`。在 `Android 5.0` 之后，基于 `chromium 39` 版本的 `webview` 可以自动从 `Google Play Store` 更新，意味着用户不必手动更新系统就能体验最新webview的特性了
