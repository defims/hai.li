---
layout: post
title:  "多行省略"
date:   2016-03-05 15:54:10
tags: mobile frontend lines wrap getClientRects getRowRects line-clamp
---

## 一个多行省略需求

文字默认多行展示，当超过n行后，在第n行最后显示`...更多`，原有第三行`...更多`处的文字自动隐藏

## line-clamp

首先的反应是`-webkit-line-clamp`，当只显示省略号，且在webkit内核的时候非常完美，css代码也相对容易理解：

```css
.line-clamp {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
}
```

## 经典总结

然而当要增加一个`更多`链接时就变得棘手了，`-webkit-line-clamp`是webkit的私有属性，且无法自定义省略时的`...`，[css-tricks](https://css-tricks.com/line-clampin/)里进行了经典总结，里面提供了六种方法（哇！），来筛选一下：

- Weird WebKit Flexbox Way 就是`-webkit-line-clamp`，简单，但不能自定义省略时的`...`
- Fade Out Way 直接将省略显示的元素增加渐变背景色，遮挡在主文字想要的位置，在设计能妥协的情况下最佳
- Opera Overflow Way 这个只针对旧版Opera有效，忽略
- [Clamp.js Way](https://github.com/josephschmitt/Clamp.js/) 这个通过js动态计算字符个数来判断文字省略时机，强依赖文字宽度，当中英文混排或者有子元素设置了边距就非常不准了，不推荐
- [ftellipsis Way](https://github.com/ftlabs/ftellipsis) 这个和我的思路核心都是利用`Element.getClientRects`获取客户矩形数组来处理，但[强依赖手动设定px的line-height](https://github.com/ftlabs/ftellipsis/blob/master/lib/index.js#L541)，且省略显示内容是动态插入的，需要改动js才能自定义，可参考
- [TextOverflowClamp.js Way]()，这个思路比较特别，直接将多行变成多个单行来处理，无法自定义省略时的`...`，忽略

## 算法

- 通过[getClientRects](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getClientRects)获取客户矩形数组
- 不同`top`不同的客户矩形数量就是行数

## 原理

这里用客户矩形(client rect)而没用行框(line box)，内联框或者块级框，因为几者没有非常一致的对应，[示例](http://codepen.io/defims/pen/XdmPmV)可以看出，容器内联，不管子元素内联还是块级，都可以通过判断`getClientRects`返回的客户矩形数组的不同高度个数得到行数

## getRowRects

进一步扩展为[getRowRects](http://codepen.io/defims/pen/YqwqGq)，返回行矩形数组，每个行矩形区域包含该行所有客户矩形区域

```javascript
function getRowRects(element) {
    var rects = [],
        clientRects = element.getClientRects(),
        len = clientRects.length,
        clientRect, top, rectsLen, rect, i;

    for(i=0; i<len; i++) {
        has = false;
        rectsLen = rects.length;
        clientRect = clientRects[i];
        top = clientRect.top;
        while(rectsLen--) {
            rect = rects[rectsLen];
            if (rect.top == top) {
                has = true;
                break;
            }
        }
        if(has) {
            rect.right = rect.right > clientRect.right ? rect.right : clientRect.right;
            rect.width = rect.right - rect.left;
        }
        else {
            rects.push({
                top: clientRect.top,
                right: clientRect.right,
                bottom: clientRect.bottom,
                left: clientRect.left,
                width: clientRect.width,
                height: clientRect.height
            });
        }
    }
    return rects;
}
```

## 文字下挤

得到了行数也就得到了是否显示`...更多`的控制权，再看显示更多问题。直接遮盖会导致遮盖不全，达到挤出文字的效果就该浮动了。只要设定`...更多`浮动到指定位置即可。利用一个右浮动元素占位，将`...更多`右浮动同时清除右浮动即可定位。得到[示例](http://codepen.io/defims/pen/EKPgGO)

## 实际应用

结合两者得到实际应用方式

### 前端手动检测（推荐）

借助js检测，最简单的是在js插入内容后调用`getRowRects`判断是否过长，动态添加过长省略类，在过长省略类里定义相应样式，当文字不嵌套子标签时，直接用`getClientRects`，[示例](http://codepen.io/defims/pen/YqwGEb/)，当可能嵌套子标签时用`getClientRects`[示例](http://codepen.io/defims/pen/aNdmrX)

### 动态监测选择器

绑定一段多行省略js到一个选择器上，当有增删改符合选择器的元素时，自动处理，为适应子元素动态计算`...更多`位置，实际[示例](http://codepen.io/defims/pen/jqWMJG)


