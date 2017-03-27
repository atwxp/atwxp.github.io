---
title: 响应式图片精灵实现
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

# 背景

在优化图片的时候，有一个分享功能，点击会弹出分享浮层，里面有各类社交 `logo` 如 `QQ`，微信，微博等，这些 `logo` 图片都是各自加载的，现在希望能做一张精灵图减少 `HTTP` 请求，样式如下图：

![Alt text](/assets/img/sms.png)

遇到的问题就是怎么能自适应的设置图片大小以及位置

<!-- more -->

# 实现原理

首先我们有一个放置背景图片的容器，通过 `CSS` 把它写成**正方形**
```
    <div class="sprite"></div>

    div.sprite {
        width: 20%;
        height: 0;
        padding-bottom: 20%;
    }
```
假设我们的精灵图是 `1000*500px`，左边是 `500px` 宽的第一张图，右边是第二张图，假设我们这样写：

```less
    div.sprite {
        background-image: url(./sprite.png);
        background-size: 100%;
    }
```
我们会发现这张图会自动缩放在容器里，不是我们想要的容器内只显示一张图的效果。

实际上我们希望显示这张精灵图的一半大小，那么我们可以设置 `background-size: 200%`，这句话会设置精灵图的宽度为两个容器宽度的大小，又因为精灵图本身的两张图宽度比例是 `1:1` 的，所以每个图的宽度都是容器宽度
```less
    div.sprite {
        width: 20%;
        height: 0;
        padding-bottom: 20%;
        background-image: url('./sprite.png');
        background-size: 200%;
        &:hover {
            background-position: 100% 0;
        }
    }
```
响应式的关键是 **使用百分比设置图片的位置**

拿项目本身来说，每个 `logo` 容器都是正方形，精灵图是 `4 * 2` 的，且每个都是 `198*198` 即也是正方形，怎么实现呢？
```less
    .logo {
        background-image: url(./sms.png);
        background-size: 400% 200%;
    }
```
水平宽度写 `400%` 即是容器宽度的 4 倍大小，精灵图本身宽度是4个logo，所以每个logo在宽度上刚好占满容器宽度；高度同理

微信、微博、空间、邮件这几个很好设置图片位置
```less
    .wx {
        background-position: 0 0;
    }
    .weibo {
        background-position: 0 100%;
    }
    .qzone {
        background-position: 100% 0;
    }
    .email {
        background-position: 100% 100%;
    }
```
那么中间的怎么设置位置呢？我是这么理解，既然每个 `logo` 和容器大小是一样的，位置又是通过百分比来设置，那么假设 `w` 是容器宽度，`m` 表示精灵图有 `m` 列，第一排第  `n` 列 `logo` 的位置：
```js
    percent * w * m - percent * w = (n - 1) * w
```
即设置第一个 `logo` 百分比是 0，第二个是  `1/3` 第三个是 `2/3` 第四个是  `3/3`

# Refer

- [responsive-background-image-sprites-css-tutorial](http://brianjohnsondesign.com/responsive-background-image-sprites-css-tutorial/)

End.
