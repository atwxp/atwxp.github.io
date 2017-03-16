---
title: window.getComputedStyle
tags: [开发经验]
categories: [FrontEnd]
---

获取元素的样式通常使用 `window.getComputedStyle`，有一天我写成了 `document.getComputedStyle`，总感觉不对劲，网上一查还真这回事o(╯□╰)o，只不过不是这样写的，而是 `document.defaultView.getComputedStyle`

## window.getComputedStyle

    window.getComputedStyle(elem, [pseudoElt])

1、使用这个方法返回的结果和 `elem.style` 返回的是一样的，都是 `CSSStyleDeclaration` 对象

![](/assets/img/getComputedStyle.png)

![](/assets/img/style-declaration.png)

但是二者也有一些区别：

<!-- more -->

- 前者是只读的，可以获取元素所有的样式信息，在 `chrome` 下测试返回 `293` 个属性
- 后者不仅可读还可以写，只能获取元素 `style` 属性中的样式

2、获取伪元素

![](/assets/img/pseudo.png)

3、值的类型

通过 `getComputedStyle()` 返回的值是 `resolved value`，这个值通常和 `CSS2.1 computed value` 相同，但是对于一些属性如 `width, height`可能就和 `used value` 相同。

[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle)上说有些浏览器获取不到元素过渡时候的样式，自测 `chomre, FF` 是可以的，不过使用的时候还是注意一下为好

> During a CSS transition, getComputedStyle returns the original property value in Firefox, but the final property value in WebKit.

4、兼容性

移动端：`IOS safari 4.3, android 3-` 不支持伪元素

PC：`IE8-` 完全不支持

所以在移动端，只要不涉及伪元素和低版本，可以放心使用

## getPropertyValue()

返回指定样式属性的值，是 `CSSStyleDeclaration` 对象的方法，用这个的好处：

1、对于驼峰式的样式名，如 `background-color`，就可以直接写

    window.getComputedStyle(div, null).backgroundColor;
    // equal
    div.style.getPropertyValue('background-color');

2、对于 `float` 这些特殊样式名，不用考虑浏览器使用 `cssFloat, styleFloat` 等

    div.style.getPropertyValue('float');

综上，推荐使用这个获取样式

3、兼容性

`IE9+`，现代浏览器 支持

## document.defaultView.getComputedStyle

不必要的，因为 `window.getComputedStyle()` 已经存在了，干嘛还再多访问一层属性呢
