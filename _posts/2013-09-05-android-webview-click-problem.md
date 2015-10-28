---
layout: post
title:  "移动端webview点击延迟处理"
date:   2013-09-05 22:21:49
categories: frontend
tags: android frontend fastclick
---

移动端webview在处理click事件的时候会有一定的延迟。我们可以用[fastclick](http://ftlabs.github.io/fastclick/)来解决，但[fastclick](http://ftlabs.github.io/fastclick/)未压缩23k，压缩后8k，比较重，这里有一个简易的解决方案：

{% gist 9133056 %}
