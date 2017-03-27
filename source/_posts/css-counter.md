---
title: CSS计数器的使用
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

## 使用场景

类似这样的列表，前面的序号可以使用 `CSS Counter` 来实现：

![](/assets/img/list-counter.png)

```css
.list {
	counter-reset: index;
	
	.item::before {
		counter-increment: index;
		content: counter(index);
	}
}
```

如果不想对某个元素生成计数器，有两个方法：

- display: none

- 无法正常生成内容的伪元素

## 好处

1、减少 `HTML` 结构，不用多写一个 `span` 等标签来包裹它
2、在某些情况下，涉及动态删除、添加列表等情况下，那么列表顺序会被打乱，不能自动排序，需要在js中动态实现；如果使用 `css` 的话，我们只需要专心实现`dom` 的增删即可
3、一般这种序号都属于样式修饰，能做到与 `HTML` 分离自然是很好的

End.