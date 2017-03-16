---
title: CSS Selector
tags: [基础知识]
categories: [FrontEnd]
---

## 什么是选择符

选择符决定它和文档树中的哪个元素相匹配，其实是模式匹配，这样它所定义的样式就能应用到那个元素上。

## 怎么写

选择符可以简单到只有一个元素名，如 `p {}`，也可以复杂到很多选择符组合在一起，如 `.header .title {}`。那它是怎么定义的呢？[CSS3](https://www.w3.org/TR/2011/REC-css3-selectors-20110929/#selector-syntax) 中定义：

- 选择符是一组或多组**简单选择符**通过连接符组合起来的
- 一组简单选择符就是多个**简单选择符**不是用连接符组合起来的
- 简单选择符就是最最基本的选择符，包含 通用选择符(`*`)、类型选择符(`p, div...`)、类(`.header...`)、伪类(`:first-child...`)、ID(`#footer`...)、属性选择符(`[attr]...`)
- 连接符包含 `space, +, > , ~`

<!-- more -->

## CSS3 和 CSS 2.1 有什么不同

1、对于简单选择符的定义不同
`CSS 2.1` 定义**简单选择符**为一个类型或通用选择符后跟着0个或多个属性、ID、伪类选择符。

也就是说 `p.name` 在 `CSS2` 中是一个简单选择符，而在 `CSS3` 中则是两个简单选择符连接在一起的一个组

2、引入了更多的伪类选择符
`CSS3` 引入了如 `nth-child(), :nth-of-type(), :target(), :not()...`等丰富的伪类选择符

3、伪元素的写法改为 `::`
把 `CSS2` 中 `:first-line` 的写法改为了 `::first-line`，即把伪元素的 `:` 改为了 `::`，应该是要和伪类进行区分

4、引入了新的连接符 `~`
这是一个通用兄弟选择符，可能是觉得 `+` 还不够用吧，只能选择直接相邻的兄弟

## 连接符

分为三种：后代(`space`)、子元素(`>`)、相邻兄弟(`+`)、通用兄弟(`~`)，例如下面的[demo](http://output.jsbin.com/gaxosubebi/1)：

```css
    p + em {
      color: #f00;
    }

    p ~ p {
      color: #00f;
    }

    div span {
      background: #ccc;
    }

    div > span {
      color: #0a0;
    }
    <div>
        <p>hello paragraph <span>div span {} select me</span></p>
        <em>p + em {} select me</em>
        <p>p ~ p {} select me</p>
        <span>div span {} and div > span {} select me</span>
    </div>
```

相邻兄弟我常用来设置重复区块之间的间隔：

```css
    .group + .group {
        margin-top: 20px;
    }
```

## 分类

- 类：`.class`
- ID：`#id`
- 通用选择符：`*`
- 类型选择符：`p, div, section...`
- 属性选择符：`[attr], [attr=val], [attr=|val], [attr=~val], [attr=^val], [attr=$val], [attr=*val]`
- 伪类
  - 动态伪类
  - 链接伪类：`:link, :visited`
  - 用户交互：`:hover, :active, :focus`
  - 目标：`:target`
  - UI：`:enabled, :disabled, :checked`
  - 结构化：`:nth-child(), :nth-last-child(), :nth-of-type(), nth-last-of-type(), :only-child, :only-of-type, :empty...`
  - :not()
- 伪元素
  - `::first-line`
  - `::first-letter`
  - `::before/::after`

## ::before/::after

这两个伪元素功能非常强大，从 [a single div](http://a.singlediv.com/)这个页面就可见一斑。日常中用它画个三角形、放置一个背景图片都是很常见的：

```css
    p::before {
        content: '';
        width: 40px;
        height: 40px;
        background: url('');
    }
```

个人很喜欢这俩，既不占用额外的 `HTML` 标签，又能写丰富的样式和动画

## 权重

我们有这么多的选择符，那如果多个选择符都匹配同一个元素时，该应用哪个呢？规范定义了一组数值 `abc`

- 计算 `ID` 选择符 的个数，赋值给 `a`
- 计算 类、属性、伪类 选择符 的个数，赋值给 `b`
- 计算 类型、伪元素选择符 的个数，赋值给 `c`
- 忽略通用选择符
- `:not()` 不计算，但它里面的选择符还是正常计算

下面的例子中 `p#footer` 的权重最大：

    * => 000
    p => 001
    p::before => 002
    p#footer => 101
    p.name => 011

这些是从选择符本身来看的，如果结合样式的来源（开发者、用户、浏览器）以及重要性 (`!important`)来看，那么一个元素最终的样式规则定义如下：

- 根据来源、重要性排序
    - 浏览器默认样式
    - 开发者定义的普通样式
    - 用户定义的普通样式
    - 用户定义的重要样式(`!important`)
    - 开发者定义的重要样式(`!important`)
- 相同重要性、来源的再根据选择符本身的权重抉择，越具体权重越大
- 前面计算的结果都一样的话，后来者居上

从上面的定义可以看到虽然用户可以覆盖开发者样式，但是 `!important` 赋予了开发者更高的权利，从而可以提高页面易访问性，避免用户瞎写导致页面布局错乱

## 属性值的计算

一旦用户代理解析完文档并构建了一个文档树，那么就需要为每个元素赋予其相应的属性值如 `border, color` 等。那么这个属性值是如何计算的呢？根据规范，最终的属性是通过下面4个步骤计算得到的：

- 规范规定的默认值或者在样式文件指定的值，`specified value`
- 转换为可以被用来继承的值，`computed value`
- 转换为绝对值如果必要的时候，`used value`
- 根据实际环境转换为实际值，`actual value`

**specified value**

首先 `UA` 必须为每一个属性赋予一个值，这个值：

1. 如果在样式文件中指定了，那么使用它
2. 否则，如果这个属性是可以继承的且不是根元素，那么使用父元素的 `computed
3. 否则，使用这个属性的初始值（由规范指定）

可以认为这一步是先要获取一个默认值（因为每个属性必须有值）

**computed value**

由第一步得到的属性值，可能是绝对值也可能是相对值。

绝对值即不依赖其他值的值，例如 `p{color: red，font-size: 14px;}`，这些值不需要计算转换，所以在第二步成为为 `computed value`

对于相对值，例如 `width: 20%`，必须要有一个参考值(依赖布局才能决定)才可以被计算出来。

还有一些相对值em，如 `{font-size: 16px;padding-top: 2em}`，就直接转换为 `{padding-top: 32px;}`，不需要等到第三步

**used value**

`css` 最终使用的值。`computed value` 是在不渲染文档最大程度处理的值。

但是有一些值只能在文档被确定布局之后才能决定。例如子元素的宽度设置为其包含块的 `20%` 等，那么子元素的宽度必须要等到包含块的宽度确定了才能确定。

所以 `computed value` 与 `used value` 的区别是：
- 前者是在页面展现之前，仅处理样式时就能得出的尽可能接近绝对的结果；
- 后者则是页面展示时，得出的绝对值。

**actual value**

原则上，`used value` 是可以被使用的，但是 `UA` 可能在一些环境中无法使用这个值，例如 `UA` 可能只允许渲染整数值的 `border`，这个时候就需要使用和 `used value` 接近的值。

这些有什么用？有一个例子是行高的继承，我们知道行高的设置有 数字、百分比、具体值，如果我们要设置一个页面所有内容都是行高为字体的2倍大小，比如：
```css
  body {
    font-size: 12px;
    line-height: 2em;
    line-height: 200%;
    line-height: 2;
  }
  <body>
    <p>font-size: 15px</p>
  </body>
```
哪个是正确的呢？答案是 `line-height: 2`，根据 [规范](https://www.w3.org/TR/2011/REC-CSS2-20110607/visudet.html#line-height)：

> `<length>`
> The specified length is used in the calculation of the line box height. Negative values are illegal.
> `<number>`
> The used value of the property is this number multiplied by the element's font size. Negative values are illegal. **The computed value is the same as the specified value.**
> `<percentage>`
**The computed value of the property is this percentage multiplied by the element's computed font size.** Negative values are illegal.

# Refer

- [CSS2.1 Selector REC - W3C](https://www.w3.org/TR/2011/REC-CSS2-20110607/selector.html)
- [CSS3 Selector REC - W3C](https://www.w3.org/TR/2011/REC-css3-selectors-20110929/ )
- [cascade REC - W3C](https://www.w3.org/TR/2011/REC-CSS2-20110607/cascade.html)
- [generate text - W3C](https://www.w3.org/TR/2011/REC-CSS2-20110607/generate.html)
- [Visual formatting model details - W3C](https://www.w3.org/TR/2011/REC-CSS2-20110607/visudet.html#line-height)

End.
