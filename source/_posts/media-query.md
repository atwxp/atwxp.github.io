---
title: Media Query
tags: [移动web开发, 基础知识]
categories: [FrontEnd]
---

# 一、Media Type

`HTML4` 和 `CSS2` 支持针对不同媒体类型（`Media Type`）定制不同的媒体样式。

在 `HTML` 中可以这样使用
```html
<link rel="stylesheet" media="print" href="print.css" />
```
在样式表中可以这样写:
 ```css
@media screen {
    p {
        color: #f00;
    }
}
```
规范定义很多媒体类型，但是都支持的浏览器确是很少，不过大都会支持这两个类型：`screen & print`

<!-- more -->

# 二、Media Query

`Media Query` 在 `Media Type`的基础上添加了对CSS属性的判断，所以它实际上是一个逻辑表达式。

### OR (media query list)

媒体查询列表即包含着多个媒体查询条件，只要符合一条即返回 `true`，即**逻辑或**的处理逻辑。如果列表为空，相当于返回 `true`。
```css
@media screen, print { … }
@media all { … }
@media { … }
```
### all

`all` 关键字表示适用于所有媒体类型，如果不明确媒体类型的话默认就是 `all`
```css
@media all and (device-width: 320px) {}
@media (device-width: 320px) {}
```
### not/only

`not` 用来指定某个的媒体查询条件，而 `only` 用来指定某种媒体查询条件，用来排除不支持媒体查询的浏览器
```css
@media not screen {}
@media only print {}
```
### 支持的属性

- `width`：布局视口的宽度

- `height`：布局视口的高度

- `device-width`：设备的宽度

- `device-height`：设备的高度

- `device-aspect-ratio`：设备像素比

- `orientation`：横屏(lanscape) 或者 竖屏(portrait)

- ...

PS：这里只列出经常使用的属性

列出来这些属性之后，我发现少了我们最经常使用的 `device-pixel-ratio`，但是文档也没有提到它，后来在 [caniuse](http://caniuse.com/#search=media) 发现 `device-pixel-ratio` 竟然标记为 `older not standard device-pixel-ratio media query`。。。写了那么久以为是 `standard`，太孤陋寡闻了~

### Resolution

`resolution`描述设备的分辨率（像素密度），单位有 `dpi & dpcm`以及后来新增的 `dppi`

- `dpi`：`dot per inch`

- `dpcm`：`dot per centimeter`

- `dppi`：`dot per pixel`

简单的说就是 `resolution` 定义了 `1px || 1 inch || 1 cm CSS` 像素包含了多少个 `物理像素点`。

例如 `iphone retina` 的设备像素比是 `2`，那么完整的写法是：
```css
/* dpi */
@media screen and (min-device-pixel-ratio: 2),
    (min-resolution: 192dpi){
}

/* dppi */
@media screen and (min-device-pixel-ratio: 2),
    (min-resolution: 2dppi) {
}
```
PS：`1dppi = 96dpi`

至于为什么 `device-pixel-ratio` 不添加到规范中，可以看看 [这篇文章](http://www.w3.org/blog/CSS/2012/06/14/unprefix-webkit-device-pixel-ratio/)

# 三、兼容性

### Media Query Feature

| IE   | Chrome | FireFox | Safari | Opera  | IOS Safari | Android |
|:----:|:----:|:---|
| 9+   | 4+     | 3.5+    |  3.1+  |  10.1+ | 3.2+       | 2.1+    |


从 上表 以及 [caniuse](http://caniuse.com/#search=media) 可以看出，除了 `IE8-` 之外，其他浏览器都是支持 `Media Query`的，真是非常赞！

### Resolution Feature


- `IE`：`IE9+` 支持，但是只支持 `dpi`


- `Chrome`：`4 - 28` 支持 `device-pixel-ratio`，`28+`支持 `resolution`

    - 最小：`-min-webkit-device-pixel-ratio`

    - 最大：`-max-webkit-device-pixel-ratio`


- `Firefox`：`16-`  支持 `device-pixel-ratio`，对于 `resolution` 只支持 `dpi`；

    - 最小：`min--moz-device-pixel-ratio`

    - 最大：`max--moz-device-pixel-ratio`


- `safari`：目前只支持 `device-pixel-ratio`，且要加 `-webkit-`



- `opera`：`10.1 - 11.5` 支持 `device-pixel-ratio`，`12.1+`开始支持 `resolution`

    - `webkit` 下的 `2` 要写成 `2/1`，`1` 即要写成 `1/1`


- `android`：`2.3 - 4.3` 支持 `device-pixel-ratio`，`4.4+`开始支持 `resolution`

所以，针对高像素密度的屏幕适配如下：
```css
@media
    only screen and (-webkit-min-device-pixel-ratio: 2),   // webkit-base(safari, android)
    only screen and (  min--moz-device-pixel-ratio: 2),    // firefox 16-
    only screen and (    -o-min-device-pixel-ratio: 2/1),  // opera 11.5-
    only screen and (      min-device-pixel-ratio: 2),     // for standard
    only screen and (        min-resolution: 192dpi),      // IE9+
    only screen and (          min-resolution: 2dppx)      // dppi
    {
        /* Retina-specific stuff here */
    }
```
# Refer

- [Media Query W3C SPEC](http://www.w3.org/TR/css3-mediaqueries)
- [unprefix-webkit-device-pixel-ratio](http://www.w3.org/blog/CSS/2012/06/14/unprefix-webkit-device-pixel-ratio/)
- [resolution-units](http://www.w3.org/TR/css3-images/#resolution-units)
- [retina-display-media-query](https://css-tricks.com/snippets/css/retina-display-media-query/)
- [iPhone iPad Media Query](http://stephen.io/mediaqueries/)

End.
