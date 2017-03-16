---
title: touchend不触发bug
tags: [移动web开发, 开发经验]
categories: [FrontEnd]
---

给一个元素绑定 `touchstart, touchmove, touchend, click`

如果只是触摸一下这个元素，按理是只触发 `touchstart touchend click`，如果滚动的话，应该就是多了一个 `touchmove` 而已

但是安卓机的某些浏览器（如 `UC`）在滚动的时候不会触发 `touchend`（偶尔也会触发），`touchmove` 可能也只是触发一次，见 [demo](http://output.jsbin.com/cuqedenofa)

解决方法：
- 在 `touchstart` 设置 `e.preventDefault()`，页面不能滚动，链接不能跳转，不会触发 `click` 事件
- 在 `touchmove` 设置 `e.preventDefault()`，页面不能滚动以及不能 `pinch-room`

参考 [Touch_events - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events#Additional_tips)
> Since calling preventDefault() on a touchstart or the first touchmove event of a series prevents the corresponding mouse events from firing, it’s common to call preventDefault() ontouchmove rather than touchstart. That way, mouse events can still fire and things like links will continue to work.

参考 [Preventing the touch default](http://www.quirksmode.org/mobile/default.html)
> The default of all actions is prevented when you return false (or call preventDefault()) ontouchstart. The touchmove event is trickier: only scroll and pinch-zoom are prevented when you return false on that event.

## Refer
- [如何修复移动浏览器上 touchend 事件不触发的bug](https://www.douban.com/note/425950082/)

End.