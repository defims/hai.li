---
layout: post
title: "黑科技：CSS定制多行省略"
date: 2017-03-08 19:54:10
tags: frontend wrap line-clamp 多行省略 css 黑科技 mobile
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

![按需显示...更多](/assets/2017-03-08-css-multiline-overflow-ellipsis/show-on-demand.gif)

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

![按需显示...更多2](/assets/2017-03-08-css-multiline-overflow-ellipsis/show-on-demand2.gif)

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

<br/>
<br/>

### 文字溢出截断

![文字溢出截断](/assets/2017-03-08-css-multiline-overflow-ellipsis/overflow-clip.gif)

```html
<!DOCTYPE html><html><body>
<style>@-webkit-keyframes width-change {0%,100%{width: 320px} 50%{width:260px}}/*测试*/</style>
<div style="font-size: 12px;display: -webkit-box;-webkit-box-orient: vertical;-webkit-line-clamp: 6;color: red;line-height: 18px;position: relative;-webkit-animation: width-change 8s ease infinite;background: rgb(230, 230, 230);">
    <div style="color:#000;display: inline;vertical-align: top;background: rgb(204, 204, 204);">腾讯成立于1998年11月，是目前中国领先的互联网增值服务提供商之一。成立10多年来，腾讯一直秉承“一切以用户价值为依归”的经营理念，为亿级海量用户提供稳定优质的各类服务，始终处于稳健发展状态。2004年6月16日，腾讯控股有限公司在香港联交所主板公开上市(股票代号700)。</div>
</div>
</body></html>
```

![文字溢出截断](/assets/2017-03-08-css-multiline-overflow-ellipsis/overflow-clip2.gif)

```html
<!DOCTYPE html><html><body>
<style>@-webkit-keyframes width-change {0%,100%{width: 320px} 50%{width:260px}}/*测试*/</style>
<div style="font-size: 36px;letter-spacing: 28px;display: -webkit-box;-webkit-box-orient: vertical;-webkit-line-clamp: 6;color: red;line-height: 18px;position: relative;-webkit-animation: width-change 8s ease infinite;background: rgb(230, 230, 230);">
    <div style="color:#000;display: inline;font-size: 12px;vertical-align: top;letter-spacing: 0;background: rgb(204, 204, 204);">腾讯成立于1998年11月，是目前中国领先的互联网增值服务提供商之一。成立10多年来，腾讯一直秉承“一切以用户价值为依归”的经营理念，为亿级海量用户提供稳定优质的各类服务，始终处于稳健发展状态。2004年6月16日，腾讯控股有限公司在香港联交所主板公开上市(股票代号700)。</div>
</div>
</body></html>
```

设置外容器的`font-size`、`letter-spacing`、`color`，并在子容器里恢复就可以单独设置省略号。这里外容器设置font-size的值等于2倍行高（余下要撑开的宽度可用`letter-spacing`补足，也可仅用`font-size`撑开全部的宽度），`color`为`transparent`就可以让line-clamp既挤出文字又不截断容器高度，外容器高度达到7行而不是默认表现的6行，从而达到需要的溢出截断效果

<br/>
<br/>

![合体](/assets/2017-03-08-css-multiline-overflow-ellipsis/merge.png)

<br/>
<br/>

### 合体！定制多行省略

![合体2](/assets/2017-03-08-css-multiline-overflow-ellipsis/merge2.gif)

```html
<!DOCTYPE html><html><body>
<style>@-webkit-keyframes width-change {0%,100%{width: 320px} 50%{width:260px}}/*测试*/</style>
<div style="position: relative;line-height:18px;-webkit-animation: width-change 8s ease infinite;max-height: 108px;">
    <div style="font-size: 36px;letter-spacing: 28px;display: -webkit-box;-webkit-box-orient: vertical;-webkit-line-clamp: 6;color: transparent;line-height: 18px;position: relative;">
        <div style="font-size:12px;color: #000;display: inline;vertical-align: top;letter-spacing: 0;">
        腾讯成立于1998年11月，是目前中国领先的互联网增值服务提供商之一。成立10多年来，腾讯一直秉承“一切以用户价值为依归”的经营理念，为亿级海量用户提供稳定优质的各类服务，始终处于稳健发展状态。2004年6月16日，腾讯控股有限公司在香港联交所主板公开上市(股票代号700)。
        </div>
        <div style="position:absolute;top: 0;left: 50%;width: 100%;height: 100%;letter-spacing: 0;color: #000;font-size: 12px;background: rgba(173, 216, 230, 0.5);">
            <div style="float: right;width: 50%;height: 100%;background: rgba(255, 192, 203, 0.5);"></div>
            <div style="float: right;width: 50%;height: 108px;background: hsla(223, 100%, 50%, 0.19);"></div>
            <div style="float: right;width: 50px;height: 18px;position: relative;background: rgba(255, 165, 0, 0.5);" class="">... 更多</div>
        </div>
    </div>
</div>   
</body></html>
```

将`-webkit-line-clamp`实现的文字溢出截断代码为主体，叠加绝对定位同步的按需显示...更多结构。因为绝对定位，这里使用百分比简化代码。最外包一层结构限制最大高度。


![合体3](/assets/2017-03-08-css-multiline-overflow-ellipsis/merge3.gif)

```html
<!DOCTYPE html><html><body>
<style>
/*
 * 行高 h
 * 最大行数 n
 * ...更多容器的宽 w
 * 字号 f
 */

@-webkit-keyframes width-change {0%,100%{width: 320px} 50%{width:260px}}
.ellipsis {
    position: relative;
    background: rgb(230, 230, 230);
    width: 260px;
    max-height: 108px; /* h*n */
    line-height: 18px; /* h */
    overflow: hidden;
    -webkit-animation: width-change 8s ease infinite;
}
.ellipsis-container {
    position: relative;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 6; /* n */
    font-size: 50px; /* w */
    color: transparent;
}
.ellipsis-content {
    color: #000;
    display: inline;
    vertical-align: top;
    font-size: 12px; /* f */
}
.ellipsis-ghost {
    position:absolute;
    z-index: 1;
    top: 0;
    left: 50%;
    width: 100%;
    height: 100%;
    color: #000;
}
.ellipsis-ghost:before {
    content: "";
    display: block;
    float: right;
    width: 50%;
    height: 100%;
}
.ellipsis-placeholder {
    content: "";
    display: block;
    float: right;
    width: 50%;
    height: 108px; /* h*n */
}
.ellipsis-more {
    position: relative;
    float: right;
    font-size: 12px; /* f */
    width: 50px; /* w */
    height: 18px; /* h */
    margin-top: -18px; /* -h */
}
</style>
<div class="ellipsis">
    <div class="ellipsis-container">
        <div class="ellipsis-content">腾讯成立于1998年11月，是目前中国领先的互联网增值服务提供商之一。成立10多年来，腾讯一直秉承“一切以用户价值为依归”的经营理念，为亿级海量用户提供稳定优质的各类服务，始终处于稳健发展状态。2004年6月16日，腾讯控股有限公司在香港联交所主板公开上市(股票代号700)。</div>
        <div class="ellipsis-ghost">
            <div class="ellipsis-placeholder"></div>
            <div class="ellipsis-more">...更多</div>
        </div>
    </div>
</div>   
</body></html>
```

将橙色块偏移到正确位置，梳理了下代码，最终实现了自适应高度的定制多行省略，只需要[-webkit-line-clamp](http://caniuse.com/#feat=css-line-clamp)和float的支持，即是安卓、ios放心大胆用，完美！从此妈妈再也不担心我在省略号后面加东西改东西了！恭喜你击败99%的同行了！


<br/>
<br/>
<br/>

## 为什么这么做？

<br/>
<br/>

### line-clamp<span style="red">3</span>宗罪

![和text-align:justify一起用会使省略号和文字相叠](/assets/2017-03-08-css-multiline-overflow-ellipsis/why.gif)

和`text-align:justify`一起用会使省略号和文字相叠

![超出截断后会截掉部分行高](/assets/2017-03-08-css-multiline-overflow-ellipsis/why2.gif)

超出截断后会截掉部分行高

![省略号可能出现在单词中间](/assets/2017-03-08-css-multiline-overflow-ellipsis/why3.gif)

省略号可能出现在单词中间

<br/>
<br/>

### 定制省略当然某问题啦

![text-align:justify时如期所示，没问题！](/assets/2017-03-08-css-multiline-overflow-ellipsis/why4.gif)

`text-align:justify`时如期所示，没问题！

![截断时如期所示，也没问题！](/assets/2017-03-08-css-multiline-overflow-ellipsis/why5.gif)

截断时如期所示，也没问题！

![省略号在有单词时如期显示，依然没问题！](/assets/2017-03-08-css-multiline-overflow-ellipsis/why11.gif)

省略号在有单词时如期显示，依然没问题！

<br/>
<br/>

### 更别说点点点花样增改

![后退，我要开始装逼了](/assets/2017-03-08-css-multiline-overflow-ellipsis/why6.jpg)

![为什么7](/assets/2017-03-08-css-multiline-overflow-ellipsis/why7.gif)

简单增改文字加链接只是小case

![为什么8](/assets/2017-03-08-css-multiline-overflow-ellipsis/why8.gif)

用折角还是其他图片表示文本溢出可以增添趣味

![为什么9](/assets/2017-03-08-css-multiline-overflow-ellipsis/why9.gif)

溢出时显示溢出字数增加了实用用途

![这B装的beautiful](/assets/2017-03-08-css-multiline-overflow-ellipsis/why10.jpg)

<br/>
<br/>
<br/>

## 如此，你怕了吗？
