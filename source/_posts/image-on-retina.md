---
title: Image On Retina
tags: [移动web开发, 基础知识]
categories: [FrontEnd]
---

> 所谓“Retina”是一种显示技术，可以将把更多的像素点压缩至一块屏幕里，从而达到更高的分辨率并提高屏幕显示的细腻程度。          —— 百度百科
 
## 1、device pixel

`device pixel/physical pixel`，即 **设备像素/物理像素**。我们近距离观察显示器屏幕，可以看到很多个点紧紧排在一起，这些点就是物理像素点，每个点都有自己的颜色和亮度。
 
一般可以通过 `screen.width/screen.height` 获取设备像素（在 `PC` 上就是电脑屏幕大小，当然这个值我们不 care）

屏幕密度，即屏幕上存在的像素数量，通常用 `PPI` 来衡量，即 `pixels per inch`，每英寸的像素数。
 
拿 `MX3` 来说，分辨率 `1800 * 1080`，尺寸 `5.1寸`，它的 `PPI`：

```js
 	// 411ppi
	PPI = Math.sqrt(1080 * 1080 + 1800 * 1800) / 5.1;
```

<!-- more -->

## 2、 css pixel

css 像素是一个相对单位，浏览器用它来渲染页面内容。通常情况下，css像素被称为 `device-independent pixels(DIPs)`。
 
在一个标准显示密度下，1个css像素对应1个屏幕像素。这对应于 `PC` 端的开发，设计图是多少像素，开发的时候就写多少像素。但是在移动端，事情变得有些复杂，涉及到 `devicePixelRatio` 的概念。
 
## 3、 关系
 
`css pixel` 用来实现页面布局，定义每个元素的位置大小，然后设备则把 `css pixel` 转为 `device pixel`。
 
如定义一个 `200*300` 的盒子，在普通屏下，显示大小为 `200 * 300`；但是在 `retina` 屏幕下(假设设备像素比是2)，保证同样的尺寸大小，`retina` 渲染的像素点就是普通屏的4倍即 `400*600`

## 4、devicePixelRatio
 
前面提到了一个概念 **设备像素比**，定义如下：
 
> devicePixelRatio is the ratio between physical pixels and device-independent pixels (dips) on the device.      - quirksmode

`window.devicePixelRatio` 可以获取设备像素比，一般都比较靠谱。其他两个值 `physical pixel、dips` 就比较复杂了。
 
## 5、dips
 
- 给页面设置 `<meta name="viewport" content="width=device-width">`
- 读取 `document.documentElement.clientWidth`
 
大多浏览器获取到的是 `layout viewport` 的宽度，也是 `dips` 的大小
 
PS：`IOS` 下，`screen.width` 返回 `dips`

## 6、physic pixel
 
- 设置页面 `<meta name="viewport" content="width=device-width">`
- 分别读取 `screen.width, window.innerWidth, document.documentElement.clientWidth`
 
在 `MX3` 下测试，只有 `screen.width` 返回的是 `physical pixel`。

[More about devicePixelRatio](http://www.quirksmode.org/blog/archives/2012/07/more_about_devi.html) 这篇文章总结到：
 
- `window.devicePixelRatio` 基本上浏览器都支持，数据可靠
- 对于 `IOS` 设备，获取 `scree.width` 得到 `dips`， 然后乘上 `window.devicePixelRatio` 得到 `physical pixel`
- 对于安卓设备，读取 `screen.width` 得到 `physical pixel`，然后除以 `window.devicePixelRatio` 得到 `dips`
 
关于 `devicePixelRatio` 的用法，参考这篇 [wiki](http://wiki.baidu.com/display/nuomiWap/Media++Query)

## 7、retina 屏幕下的图片适配

### Bitmap Pixels
 
位图像素是栅格图像中最小的数据单位，每个像素包含着它的位置信息、颜色、透明度。
 
和分辨率无关，web上的图像都有一个抽象的大小，是通过 css 定义的，浏览器会根据 css 定义的大小渲染图片。
 
其实和前面的盒子说到的问题一样，如果我们定义一张图 `200*300` 大小，在 `retina` 下为了保持同等尺寸大小，因为 `retina` 相比普通屏多了一倍像素，就是说现在要用4倍的像素量填充图片的大小，但是因为位图像素不能被分割，这样就会取就近色进行填充，导致图片失真。
 
### 图片适配

为了适应 retina 高清屏，我们需要一份 `@2x` 的图片，在 css 这样写：
```css
.icon {
	width: 20px;
	heigth: 20px;
	background: url(example@2x.png) no-repeat 0 0;
	background-size: 20px 20px;
}
 ```
有时候我们会需要适配不同设备像素比，就要用到 [media query](http://wiki.baidu.com/display/nuomiWap/Media++Query) 了
 ```css
.icon {
	width: 20px;
	heigth: 20px;
	background: url(example.png) no-repeat 0 0;
	background-size: 20px 20px;
}

@media screen and (-webkit-min-device-pixel-ratio: 3){
	background-image: url(example@3x.png);
}
 ```
## Refer
- [Towards A Retina Web](http://www.smashingmagazine.com/2012/08/towards-retina-web/) 
- [More about devicePixelRatio](http://www.quirksmode.org/blog/archives/2012/07/more_about_devi.html)
- [A tale of two viewports](http://www.quirksmode.org/mobile/viewports2.html)
- [visual-viewport-vs-layout-viewport-on-mobile-devices](http://stackoverflow.com/questions/7344886/visual-viewport-vs-layout-viewport-on-mobile-devices)

End.