---
title: RSS Reader Chrome Extension
tags: [chrome extension]
categories: [茶余饭后]
---

写了个简单的 `rss reader` 扩展，用来订阅一些博客了解最新的技术进展。

## 1、实现的功能
- 支持添加订阅、导入已有的 OPML，导出 OPML，可以分享到分享微博、印象笔记、（微信todo）
- 可以配置每页文章数量，几天更新一次
- 使用客户端存储 `localStorage`，分为两种类型存储：一种是 `rss feeds` 列表数组，一种是每个 `rss feed` 的文章数组
- 每次切换到新的 `rss feed`，都会先判断本地存储是否存在且是否过期，不存在或者过期了都会重新更新
- 导入 `OPML` 不会立刻请求所有文章更新，而添加单个 `feed` 会立刻请求文章存储在本地

## 2、实现效果
![](/assets/img/rss.png)

![](/assets/img/rss-detail.png)

<!-- more -->

## 3、怎么做的

主要是用 `vue + vuex + vue-router + vue-resource + webpack` 搭建，`chrome` 扩展以及 `rss` 相关的知识

### 3.1 chrome extension

首先每个扩展都必须在有个 `manifest.json` 文件，它是扩展的入口，数据格式是 `json`，如：
```js
{
    "manifest_version": 2,
    "name": "RSS",
    "version": "2.0",
    "browser_action": {
        // "default_icon": {
        //     "19": "",
        //     "38": ""
        // }
    },
    "background": {
        "scripts": ["background.js"],
        "persistent": false
    },
    "permissions": ["*://*/*", "tabs"],
    "content_security_policy": "script-src 'self' 'unsafe-eval'; object-src 'self'"
}
```

`name, version, manifest_version` 是必须的，`manifest_version` 目前来说是 2

当我们点击扩展图标的时候，希望打开一个新的 `tab`，可以通过 `background` 配置

```js
// background.js
chrome.browserAction.onClicked.addListener(function (tab) {
    chrome.tabs.create({url: url})
})
```

因为我们需要请求 `feed url`，所以会涉及到跨域，`chrome` 扩展允许指定 `permissions` 声明跨域请求，比如我们尝试请求 `m.nuomi.com` 可以配置 `manifest.json` 如下：

```js
"permissions": ["*://*.nuomi.com/*"]
```

### 3.2 atom/rss xml

这俩的规范在 [这里](https://validator.w3.org/feed/docs/)

### 3.3 vue

#### 路由
路由设计很简单就三个：
- `/add`: 添加、导出、导入 feed
- `/setting`：设置每页文章数目、每几天更新
- `/feed/:id`：feed的详情页

#### 数据
数据有两个：
```js
const store = {
    feedList: [],
    config: {
        perPage: 5,
        expired: 1
    }
}
```

`action` 有三个：`ADD_FEED, DELETE_FEED, UPDATE_CONFIG`
```js
[types.ADD_FEED] (state, feed) {
    feed.forEach(f => state.feedList.push(f))

    ls.set('feeds', state.feedList)
},

[types.DELETE_FEED] (state, id) {
    Vue.set(state, 'feedList', state.feedList.filter(f => f.id !== id))

    ls.set('feeds', state.feedList)
},

[types.UPDATE_CONFIG] (state, cfg, save = true) {
    Object.assign(state.config, cfg)

    save && ls.set('config', state.config)
}
```

#### 分页

拿到了某个 `feed` 下的所有文章列表后，会有两个变量 `allPost`(所有文章列表) 以及 `post`(要渲染的文章列表)，会在组件 `v-pager` 中触发 `pagechange` 事件获取当前页码
```js
pagechange(curPage) {
    this.post = this.allPost.slice((curPage - 1) * this.perPage, curPage * this.perPage)
}
```

#### 解析 xml

解析一段字符串为 `XML` 主要用到了 `new DOMParser()`，然后根据文档到特点解析相应的信息即可

#### 其他

`webpack2` 有很多规则都不太一样了，比如 `loader` 的配置，`loaders` 改为了 `rules`，`loader`名字不能简写要写上 `xx-loader`

```js
module: {
    rules: [
        {
            test: /\.css$/,
            use: ['style-loader', 'css-loader']
        }
    ]
}
```
比如 `resolve` 中 `extensions` 第一个元素不再是 `''` 了，默认是 `['.js', 'json'`，模块的查找路径写成了 `modules: [src_path, 'node_modules']`
```js
resolve: {
    modules: [SRC_PARH, 'node_modules'],
    alias: {
        'vue$': 'vue/dist/vue.common.js'
    },
    extensions: ['.js', '.json', '.vue']
},
```

学习了 `vue` 全家桶的是用，不懂的地方其文档也写的非常清楚，以后还要更深入多理解
