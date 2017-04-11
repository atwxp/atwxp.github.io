---
title: 弹性布局的实践
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

如下图所示的布局，红球区域有33个球，篮球区域有16个球，要求自适应页面且最边上的球必须紧靠区域边缘

![Alt text](/assets/img/flex-study-001.png)

`HTML` 结构

```html
<div class="ball-list">
<i class="ball"></i>
<i class="ball"></i>
...
</div>
```

<!-- more -->

## 1、`float/inline-block` 布局

```css
.ball {
    display: inline-block;
    width: 28px;
    height: 28px;
    margin-right: 8px;
}
```

![Alt text](/assets/img/flex-study-002.png)

右侧会存在空隙，因为一行放不下的时候，浏览器就会折行显示

## 2、`flex` 布局

这种复杂又要求自适应宽度的布局，用 `flex` 再好不过了

```less
.ball-list {
	display: flex;
	flex-wrap: wrap;
	justify-content: space-between;
	.ball {
		// 兼容旧浏览器，子元素必须是块元素
		display: block;
	}
}
```

效果如下：

![Alt text](/assets/img/flex-study-003.png)

看样子不错的说，但是最后一行不尽人意

## 3、`flex + margin: auto` 布局

```less
&.flexauto {
    .ball:last-child {
      margin-right: auto;
    }
  }
```

![Alt text](/assets/img/flex-study-004.png)

效果不错，但是不够好

## 4、`flex + js`

尝试了前面几种方法，都不尽人意，看来纯粹使用 `css` 是行不通的，那只能使用 `js` 计算空出的占位符个数，手动填满这个区域

```js
var width = parseFloat(window.getComputedStyle(ballList).width);

// 33 是球的个数
// 8 是 margin-right
// 38 是球的宽度
var cols = Math.floor((width + 8) / 38);

var remainCols = cols - 33 % cols;

var fragment = document.createDocumentFragment();

for (var i = 0; i < remainCols; i++) {
    var itag = document.createElement('i');

    itag.className = 'ball';

    fragment.appendChild(itag);
}
  
ballList.appendChild(fragment);
```
  
![Alt text](/assets/img/flex-study-005.png)

大功告成

## 5、Demo

戳 [这里](http://output.jsbin.com/vexaseloku)

End.