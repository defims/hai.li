---
layout: post
title:  "移动端webview点击延迟处理"
date:   2013-09-05 22:21:49
categories: frontend
tags: android frontend fastclick
---

移动端webview在处理click事件的时候会有一定的延迟。我们可以用[fastclick](http://ftlabs.github.io/fastclick/)来解决，但[fastclick](http://ftlabs.github.io/fastclick/)未压缩23k，压缩后8k，比较重，这里有一个简易的解决方案：

```javascript
function onFastClick(element ,handler) {
    var delay = 300,
        offset = 10,
        still = true,
        startX = 0,
        startY = 0,
        touch;
    function longClick() {//排除长按
        still = false;
    }
    function start(e) {
        still = true;
        touch = e.touches[0];
        startX = touch.pageX;
        startY = touch.pageY;
        setTimeout(longClick ,delay);
    }
    function move(e) {
        touch = e.touches[0];
        if(touch.pageX - startX >= offset || touch.pageY - startY >= offset) {//排除滑动
            still = false;
        }
    }
    function end(e) {
        clearTimeout(longClick);
        still && handler();
    }
    element.addEventListener("touchstart" ,start);
    element.addEventListener("touchmove" ,move);
    element.addEventListener("touchend" ,end);
    element.addEventListener("touchcancel" ,end);
}
onFastClick(document ,function() {
    console.log("fastclick")
})
```
