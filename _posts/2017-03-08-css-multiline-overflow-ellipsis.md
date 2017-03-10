---
layout: post
title:  "CSS定制多行省略"
date:   2017-03-08 19:54:10
tags: frontend lines wrap line-clamp 多行省略 css
---

## 什么是多行省略？

![什么是多行省略](/assets/2017-03-08-css-multiline-overflow-ellipsis/what.jpg)

当字数多到一定程度就显示省略号点点点。最初只是简单的点点点，之后花样越来越多，点点点加下箭头，点点点加更多，点点点加更多加箭头...。多行省略就是大段文字后面的花式点点点。

<br/>
<br/>
<br/>

## 同行这么做：

![同行这么做](/assets/2017-03-08-css-multiline-overflow-ellipsis/competitor.jpg)

1. Google Plus用透明到白色的渐变遮罩，渐变遮罩在文字超出的时候才显示，但无法挤出文字，且背景只能纯色，不理想。
2. 豌豆荚则更简单粗暴换行显示，换行显示则文字未超出时依然显示 ...xxx，更不理想！

<br/>
<br/>
<br/>

## 我这样做：

![我这么做](/assets/2017-03-08-css-multiline-overflow-ellipsis/my-solution.jpg)

在QQ浏览器的页面用了一个<span style="color:red">原创</span>的mod-more UI组件，实现了<span style="color:red">定制</span>的多行省略，还是<span style="color:red">纯CSS</span>的，领先同行一大截，赞！赞！赞！只可惜，mod-more组件的高度是固定的。对mod-more进一步进化，完美自适应高度，而且代码简化易用。

<br/>
<br/>
<br/>

## 怎么做到的？

![怎么做到的](/assets/2017-03-08-css-multiline-overflow-ellipsis/how.gif)

<span>定制多行省略 <span style="color:red">=</span> 按需显示...更多 <span style="color:red">+</span> 文字溢出截断，按需显示...更多是用浮动特性实现，文字溢出阶段可以用前置浮动和line-clamp实现，QQ浏览器现有方案就是前置浮动，但缺点是高度固定，使用line-clamp则自适应高度，完美！限于篇幅，这里只提line-clamp的实现方案，QQ浏览器将在下阶段升级至该方案。

## 原理详解！

<br/>
<br/>


### 按需显示...更多

![怎么做到的](/assets/2017-03-08-css-multiline-overflow-ellipsis/show-on-demand.gif)

```html
<!doctype html><html><body>
<style>@-webkit-keyframes width-change {0%,100%{width: 320px} 50%{width:260px}}/*测试*/</style>
<div style="font-size:12px;line-height: 18px;-webkit-animation: width-change 8s ease infinite;background: rgb(230, 230, 230);">
    <div style="float:right;margin-left: -50px;width:100%;position:relative;background: hsla(229, 100%, 75%, 0.5);">腾讯成立于1998年11月，是目前中国领先的互联网增值服务提供商之一。成立10多年来，腾讯一直秉承“一切以用户价值为依归”的经营理念，为亿级海量用户提供稳定优质的各类服务，始终处于稳健发展状态。2004年6月16日，腾讯控股有限公司在香港联交所主板公开上市(股票代号700)。</div>
    <div style="float:right;position:relative;width:50px;height: 108px;color:transparent;background: hsla(334, 100%, 75%, 0.5);">placeholder</div>
    <div style="float:right;width:50px;height:18px;position: relative;background: hsla(27, 100%, 75%, 0.5);">...更多</div>
</div>
</body></html>
```

利用右浮动原理——右浮动元素从右到左依次排列，不够空间则换行。蓝色块、粉色块、橙色块依次右浮动，蓝色块高度小于6行文字时，橙色块在右边，蓝色块高度大于6行文字时，左下角刚好够橙色块排列的空间，于是橙色块就到左边了

![怎么做到的2](/assets/2017-03-08-css-multiline-overflow-ellipsis/show-on-demand2.gif)

```html
<!doctype html><html><body>
<style>@-webkit-keyframes width-change {0%,100%{width: 320px} 50%{width:260px}}/*测试*/</style>
<div style="font-size:12px;line-height: 18px;-webkit-animation: width-change 8s ease infinite;background: rgb(230, 230, 230);">
    <div style="float:right;margin-left: -50px;width:100%;position:relative;background: hsla(229, 100%, 75%, 0.5);">腾讯成立于1998年11月，是目前中国领先的互联网增值服务提供商之一。成立10多年来，腾讯一直秉承“一切以用户价值为依归”的经营理念，为亿级海量用户提供稳定优质的各类服务，始终处于稳健发展状态。2004年6月16日，腾讯控股有限公司在香港联交所主板公开上市(股票代号700)。</div>
    <div style="float:right;position:relative;width:50px;height: 108px;color:transparent;background: hsla(334, 100%, 75%, 0.5);">placeholder</div>
    <div style="float:right;width:50px;height:18px;position: relative;background: hsla(27, 100%, 75%, 0.5);left: 100%;-webkit-transform: translate(-100%,-100%);">...更多</div>
</div>
</body></html>
```

进一步将橙色块偏移到正确位置就大功告成了！细心的同学会发现，将橙色块加上渐变底就是Google Plus在用的方案。


