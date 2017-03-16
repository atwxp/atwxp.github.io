---
title: background-clip 的 bug
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

同时设置 `border-radius` & `background` & `border-color` & `background-clip` 在 `Android 4.1.1` 上有 `bug`
```less
    position: absolute;
    top: 50%;
    left: 50%;
    margin-top: -37px;
    margin-left: -37px;
    width: 66px;
    height: 66px;

    // border: 4px solid rgba(255, 255, 255, 0.5);
    border: 4px solid #f00;
    background: #fafafa;
    border-radius: 50%;
    -webkit-background-clip: content-box;
    background-clip: content-box;

    text-align: center;
    line-height: 1;
    z-index: 1;
```
设置 `border` 不是透明色，表现的和预期一样：


![Alt text](/assets/img/bgclip-normal.png)

如果设置为透明色，就会这样：

![Alt text](/assets/img/bgclip-error.png)

## Refer

- [the-backgound-clip-property-and-use-cases](https://css-tricks.com/the-backgound-clip-property-and-use-cases/)

End.
