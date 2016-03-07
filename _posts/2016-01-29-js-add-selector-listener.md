---
layout: post
title: "css选择器监听器addSelectorListener"
date: 2016-01-29 18:13:00
tags: frontend javascript css
---

[davidwalsh](https://davidwalsh.name/detect-node-insertion)的文章提到利用animationStart事件来监测新元素，但只提供了思路，在实际应用中可以做以下优化：

- 使用空keyframes内容减少属性污染
- 增加触发标志位避免animation属性被长时间占用，触发后立即销毁
- 改进的animation-name的uid机制
- js动态生成css，避免还要另写css
- 用了!important来增大权重，有可能导致更高权重的css覆盖导致事件失效

代码如下：

```javascript
function addSelectorListener(selector, handler) {
    var uid = "js-live-",
        $style = document.createElement("style");

    function animationStart(e) {
        if(e.animationName == uid) {
            var element = e.target;
            element.classList.add(uid);//增加触发标志，去除animation占用
            handler(element, selector);
        }
    }

    //选择器有变动则触发animationstart，然后调用回调
    document.getElementsByTagName("head")[0].appendChild($style);
    for(var i=0,el=$style; el = el.previousElementSibling; i++) ;//利用style标签的位置做uid
    uid += i;
    $style.appendChild(document.createTextNode("@-webkit-keyframes "+uid+" {0%{}}@keyframes "+uid+" {0%{}}"+selector+":not(."+uid+"){-webkit-animation:"+uid+" 1ms;animation:"+uid+" 1ms}"));
    document.addEventListener("webkitAnimationStart", animationStart, false);
    document.addEventListener("animationstart", animationStart, false);
}
```
