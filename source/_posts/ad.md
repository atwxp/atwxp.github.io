---
title: 反广告过滤
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

 `dom`  的类名中存在某些关键词会被一些浏览器（猎豹，qq，UC...）的广告过滤机制干掉

已知的会被过滤掉的关键字：

- banner
- ad
- poster
- flag
- block
- download
- na-download

包括 `HTML` 中的 `<img />` 的资源路径可能也会被过滤了，有次我把图片命名为 `ad.jpg` 就被过滤了。。

End.
