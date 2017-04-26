---
title: 微信小程序的尝试
tags: [小程序]
categories: [FrontEnd]
---

因工作需求尝试开发了微信小程序，这里总结一下开发经验。

## 准备工作

根据[官方文档](https://mp.weixin.qq.com/debug/wxadoc/dev/)，我们首先要申请 `AppId`，跟着流程走就好了，因为是个人开发所以**主体类型**就选择**个人**。

接下来我们要下载 [微信开发者工具](https://mp.weixin.qq.com/debug/wxadoc/dev/devtools/download.html)，它用来创建、调试、发布小程序。

浏览一下官方文档，我们大概知道：

- 小程序的文件类型有四种： `.wxml, .wxss, .js, .json`

- 小程序的每个页面都要写在 `app.json` 中的 `pages` 字段内，且 `pages` 中的第一个页面是小程序的首页，格式为 **路径＋页面**，如
	
```
// app.json
"pages":[
    "pages/welcome/welcome",
    "pages/index/index",
    "pages/logs/logs"
],
```
- 每个页面都是由同路径下同名的四个不同后缀文件的组成，如 `index` 下有 `index.js, index.wxml, index.wxss, index.json`

- 整个程序不是跑在浏览器下，所以和 `DOM` 相关的事件没法使用

- 我们不能加载非自定义的页面，如跳转到 `m.baidu.com`

<!-- more -->

## 创建项目

看完官方文档介绍后，就可以着手写个 [Todo](https://github.com/atwxp/weapp-todos) 了（Fork 别人修改的）

![](/assets/img/wechat-001.png)

项目结构很简单，就只有一个页面：

![Alt text](/assets/img/wechat-002.png)

这里把 `todos` 放在 `localstorage` 保存：

```js
save: function (data) {
    wx.setStorageSync(STORAGE_TODO_KEY, data)
},
```

不管是添加、删除、更改状态，操作的对象都是 `Todos` 这个数组，但是已经完成的 `todo`、没有完成的 `todo` 的个数都是和 `Todos` 相关联的

```js
filterDone: function (data = this.data.todos) {
    return data.filter(todo => todo.done)
},

filterRemain: function (data = this.data.todos) {
    return data.filter(todo => !todo.done)
},
```

所以我们可以写个类似 `vue` 中 `computed` 的计算属性：

```js
computedTodoData: function (data) {
    return {
        todos: data,
        doneCount: this.filterDone(data).length,
        remainCount: this.filterRemain(data).length
    }
},
```

这样在执行如删除操作的时候，手动调用它就好了

```js
// 删除 todo item
removeTodo: function (e) {
    const index = e.currentTarget.dataset.index

    this.data.todos.splice(index, 1)

    this.setData(this.computedTodoData())
},
```

## 发布

项目完成后点击发布，就可以在开发者中心的 **开发管理** 找到 **开发版本**：

![Alt text](/assets/img/wechat-003.png)

右侧点击 **体验版** 就可以发布为体验版，点击左下角就可以下载二维码进行扫码体验（即使后面再次修改再次发布，这个二维码也不会变）

然后在 **用户身份** 这里绑定需要的体验者：

![Alt text](/assets/img/wechat-004.png)

## 不同

#### 1. rpx

`wxss` 中的像素单位有 `px, rpx, em...` 其中 `rpx` 是在微信中定义的，即把手机屏幕分为750份

```js
	1rpx = screen.width / 750
```

拿 `iphone6` 来看， `1rpx = 0.5px`

所以按照 `iphone6` 的设计稿，前端直接量取间距就可以使用，无需再进行换算

#### 2. 组件

微信定义了一系列组件，不能使用 `HTML` ，就使用来看

`<view>` 相当于 `<div>`

`<p><h1-h6><span><em>...` 相当于  `<text>`

`<img>` 相当于 `<image>`

...

#### 3. 图片

微信的 `<image>` 组件很好用，当我们要把一个图片按比例居中显示，`HTML` 的写法有点复杂：

```html
<div class="wrapper">
    <div class="image" style="background-image:url();">
</div>
.wrapper {
    width: 100px;
    height: 0;
    padding-top: 100%;
}
.image {
    height: 0;
    padding-bottom: 100%;
    background-size: cover;
    background: top center;
}
```

在小程序中非常简单，魔法就是 `mode` 属性，设置为 `aspectFill` 就可以实现 `background-size: cover` 的效果：

```html
<image mode="aspectFill" src="{{img}}"></image>
```
## 有坑

#### 1. 开启环境不校验请求域名

涉及到 `wx.request()` 的请求，开发的时候要勾选 **开发环境不校验请求域名** 不然 会请求失败

![Alt text](/assets/img/wechat-005.png)

同时在手机预览调试的时候，需要打开手机上的调试工具 [vConsole](https://github.com/WechatFE/vConsole)

#### 2. 选择符

官方给的选择器有类、ID、元素、`::before/::after`，但是在项目中 `:nth-child, div + div` 这些也是能用的，不知道会不有坑。。。

#### 3. flex布局的坑

写了个 [例子](http://output.jsbin.com/yumoceqohu) 想实现这样的布局，使用了 `flex`，在微信上实现如下：

```less
<view class="item">
        <image src="http://gw.alicdn.com/bao/uploaded/T1tKJKXqlcXXb1upjX.jpg" mode="aspectFill" class="image"/>      
        <view class="content">
        <text class="title">helllohelllo,helllo,helllohelllo, helllohelllo,helllohelllo</text>
        <text class="desc">fsfsfhelllo, helllohelllo, helllosdfs</text>
        </view>
</view>

.item {
    display: flex;
    padding: 12px 0;
}
.image {
    display: block;
    width: 25%;
    height: 160rpx;
    margin-right: 24rpx;
}
.content {
    flex: 1;
    line-height: 44rpx;
    font-size: 28rpx;
    // width: 20rpx;
    // overflow: hidden;
}
.title {
    display: block;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    color: #00c;
}

.desc {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
    color: #333;
}
```

显示如下：

![Alt text](/assets/img/wechat-006.png)

左侧图片移出视口之外了，给 `.content` 加上宽度或者`overflow: hidden` 就好了

![Alt text](/assets/img/wechat-007.png)

类似的问题在华为 `UC` 上也遇到过，需要给 `flex item` 定个宽度才可以

## 总结

- 移动端常见的 `swiper, scroll` 组件都已经内置了，填一下数据就好
- 数据必须使用 `setData()` 才能更新视图
- 页面布局基本使用 `flex` 搞定
- 一个列表的图片都是一次性加载的，没有性能优化？？
- 还没有在多个真机实践到底还有多少坑

## refer

- [小程序 - 官方文档](https://mp.weixin.qq.com/debug/wxadoc/dev/)

- [有用！关于微信小程序，那些开发文档没有告诉你的](https://zhuanlan.zhihu.com/p/23536784)