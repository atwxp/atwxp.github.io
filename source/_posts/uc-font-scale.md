---
title: UC下字体突然变大
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

项目中遇到了页面的字体在 `UC` 变大的问题，后来排查是一个地方的 `font-size: 12px` 导致的，临时方案就是换了字体大小解决了问题

后来其他同事又遇到了这个问题，这不得不重视起来，于是在 [知乎](https://www.zhihu.com/question/29769089) 上找到了答案

> uc浏览器判断到页面上文字居多时，会自动放大字体优化移动用户体验

解决方法是在头部加上
```html
<meta name="wap-font-scale" content="no">
```
尝试了其他方法也是可以的，如改变字体大小，把样式表放在最后加载等

End.
