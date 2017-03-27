---
layout: post
title: "js continuation monad简明推导"
date: 2017-03-27 19:51:02
tags: frontend javascript promise fp monad
---

## 背景

上篇文章 [函数式JS: 一种continuation monad推导](http://hai.li/2015/06/29/js-continuation-monad-derivation.html) 得到了一个类似promise的链式调用，引发了这样的思考：难道promise是monad？如果是的话又是怎样的monad呢？来来来，哥哥带你推倒，哦，不，是推导一下！

## Monad

[Monad](https://www.zhihu.com/question/19635359)是haskell里很重要的概念，作为一种类型，有着固定的操作方法，简单的可以类比面向对象的接口。

### 定义

```haskell
unit :: a -> Monad a
flatMap :: Monad a -> (a -> Monad b) -> Monad b
```

这是[类型签名](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/ch7.html)的表述。`unit`的作用可以理解为将`a`放入容器中变成`Monad a`。而当`flatMap`转为`(a -> Monad b) -> (Monad a -> Monad b)`时，它的作用就可以理解为将`a -> Monad b`函数转换成`Monad a -> Monad b`函数。

### 法则

```javascript
flatMap(unit(x), f) ==== f(x) //左单位元
flatMap(monad, unit) ==== monad //右单位元
flatMap(flatMap(monad, f), g) ==== flatMap(monad, function(x) { flatMap(f(x), g) }) //关联性
```
这里`x`是一般的值，`f`和`g`是一般的函数，`monad`是一个`Monad`类型的值，可以这么理解:

1. 左单位元法则就是将包裹`unit(x)`和函数`f`传给`flatMap`执行等价于将包裹中的值`x`抽出传给函数`f`执行
2. 右单位元法则就是将包裹`monad`和函数`unit`传给`flatMap`执行等价于包裹`monad`本身（有点像`1*1=1`）
3. 关联性法则就是将包裹`monad`和函数`f`传给`flatMap`执行，再将执行的结果和函数`g`传给`flatMap`执行等价于将包裹`monad`中的值`x`抽出传给`f`执行（执行结果依然是`Monad`类型），再将执行结果中的值`x`抽出传给`g`执行

## Promise

### 链式调用

```javascript
new Promise(function(resolve) { setTimeout(function() { resolve("0") }, 100) })
.then(function(v) { 
	console.log(v);
	return new Promise(function(resolve) { resolve(v + "->1") }) 
})
.then(function(v) { 
	console.log(v);
return new Promise(function(resolve) { setTimeout(function() { resolve(v + "->2") }, 1000) })
})
.then(console.log)
```

### 分析

先将`Promise`链式调用整理一下，将关注点集中在链式调用上

```javascript
function f0(resolve) { setTimeout(function() { resolve("0") }, 100) }
function f1(v) { console.log(v); return new Promise(function(resolve) { resolve(v + "->1") }) }
function f2(v) { console.log(v); return new Promise(function(resolve) { setTimeout(function() { resolve(v + "->2") }, 1000) }) }
function f3(v) { console.log(v) }
new Promise(f0).then(f1).then(f2).then(f3) //0 0->1 0->1->2
```

从`unit`和`flatMap`的特性可以直观地对应为

```javascript
function g0(resolve) { setTimeout(function() { resolve("0") }, 100) }
function g1(v) { console.log(v); return unit(function(resolve) { resolve(v + "->1") }) }
function g2(v) { console.log(v); return unit(function(resolve) { setTimeout(function() { resolve(v + "->2") }, 1000) }) }
function g3(v) { console.log(v) }
unit(g0).flatMap(g1).flatMap(g2).flatMap(f3)
```

而对象的方法可以通过将`this`作为参数传入方便地转为直接的函数，比如

```javascript
var a = {method: function f(v){ console.log(this, v) }}
var a_method = function(t, v){ console.log(t, v) }
a.method("a") === a_method(a, "a")
```

这样将链式调用转为嵌套函数调用变成

```javascript
function g0(resolve) { setTimeout(function() { resolve("0") }, 100) }
function g1(v) { console.log(v); return unit(function(resolve) { resolve(v + "->1") }) }
function g2(v) { console.log(v); return unit(function(resolve) { setTimeout(function() { resolve(v + "->2") }, 1000) }) }
function g3(v) { console.log(v) }
flatMap(
	flatMap(
		flatMap(
			unit(g0)
		)(g1)
	)(g2)
)(g3)
```

这样如果`unit`和`flatMap`这两个直接函数可以构造推导出来，就可以窥探`Promise`的真面目了。同学们！这道题！必考题！头两年不考，今年肯定考！

### 构造推导`unit`

```javascript
function unit(f){ return f}
```

1. 由`flatMap :: Monad a -> (a -> Monad b) -> Monad b`和`flatMap(unit(g0))(g1)`可知传入`g1`的参数就是`a`，对应着`"0"`。
2. 但由`unit :: a -> Monad a`和`unit(g0)`得到的`a`却对应着`g0`。实际上`a`对应着`"0"`，只是`a`在`g0`里作为立即量传入，在`g1`和`g2`的返回值中作为闭包引用传入。
3. `Monad`可看作容器，那用什么做的容器呢？既然作为参数传入`unit`的函数`f`已经包裹了`a`，那试试直接作为`Monad a`返回。同时根据`g0`看出返回值`f`是用回调返回值的。也就是*将一个用回调返回结果的函数作为容器*。

###  构造推导`flatMap`

```javascript
function flatMap(ma){
	return function(g) {
		var b, h=[];
		ma(function(a) { //1. 解包`ma`取出`a`
			g = g(a); //2. 将`a`传到`g`中执行
			(g && g.flatMap ? g : unit(function(c) { c(g) })) //处理g没返回unit情况
				(function(v) {
					b = v; // 1.1—1.2—1.3
					while(h.length) h.pop()(v); //1.1—1.3—1.2
				})
		});
		return unit(function(c) { //3. 将执行结果`b`包裹成`mb`返回
			b 
			? c(b)  // 1.1—1.2—1.3—2.1
			: h.push(c) //1.1—1.3—1.2—2.1
		})
	}
}
```

1. 由`flatMap :: Monad a -> (a -> Monad b) -> Monad b`知道`flatMap`传入`Monad a`返回函数，这个函数接收`(a -> Monad b)`返回`Monad b`，而`(a -> Monad b)`对应`g1`。可以构造`flatMap`如下
```javascript
function flatMap(ma){return function(g1) { /*b = g1(a);*/ return mb }}
```
2. 实际`flatMap`做了3步工作

    1. 解包`ma`取出`a`
    2. 将`a`传到`g1`中执行
    3. 将执行结果`b`包裹成`mb`返回

3. 这里`ma`和`g1`都是容器，通过回调得到输出结果，所以在`ma`的回调中执行`g1(a)`，再在`g1(a)`的回调中得到执行结果`v`，再将执行结果`v`赋值给外部变量`b`，最后将`b`用`unit`包裹成`Monad b`返回。
```javascript
function flatMap(ma){
	return function(g1) {
		var b;
		ma(function(a) {
			g1(a)(function(v) {
				b = v
			})
		});
		return unit(function(c) {c(b)})
	}
}
```
4. 如果`g1`是立即执行的话，第`flatMap`的执行步骤是1--2--3，但如果2延迟执行步骤就变成了1--3--2，算上下一个`flatMap`就是1.1--1.3--1.2--2.1。2.1的`ma`就是1.2的`mb`，2.1的`ma`的参数`c`中执行了2.2和2.3，也就是1.3的`c`决定着2.1之后的步骤。如果将`c`赋值给`b`就可以在1.2执行完后才继续2.1之后的步骤，也就是：
```text
        +-----------+
1.1—1.2—1.3—2.1    2.2—2.3
```
```javascript
function flatMap(ma){
	return function(g1) {
		var b;
		ma(function(a) {
			g1(a)(function(v) {
				b(v)
			})
		});
		return unit(function(c) { b = c })
	}
}
```
5. 为了`flatMap`可以链接多个`flatMap`，也就是一个1.3被多个2.1消化，需要保存所有在2.1后的执行链 `c`，用数组`h`解决。
```javascript
function flatMap(ma){
	return function(g1) {
		var h=[];
		ma(function(a) {
			g1(a)(function(v) {
				while(h.length) h.pop()(v)
			})
		});
		return unit(function(c) { h.push(c) })
	}
}
```
5. 整合1.2立即执行和延迟执行情况，代码如下：
```javascript
function flatMap(ma){
	return function(g1) {
	var b, h=[];
	ma(function(a) {
		g1(a)(function(v) {
			b = v; // 1.1—1.2—1.3
			while(h.length) h.pop()(v); //1.1—1.3—1.2
		})
	});
	return unit(function(c) { 
		b 
		? c(b)  // 1.1—1.2—1.3—2.1
		: h.push(c) //1.1—1.3—1.2—2.1
	})
	}
}
```
6. 由于`g3`没有返回`mb`，所以还要加上对`g1`返回的不是容器的处理，代码如下：
```javascript
function flatMap(ma){
	return function(g1) {
		var b, h=[];
		ma(function(a) {
			g1 = g1(a);
			(g1 && typeof(g1) == "function" ? g1 : unit(function(c) { c(g1) }))
				(function(v) {
					b = v; // 1.1—1.2—1.3
					while(h.length) h.pop()(v); //1.1—1.3—1.2
				})
		});
		return unit(function(c) { 
			b 
			? c(b)  // 1.1—1.2—1.3—2.1
			: h.push(c) //1.1—1.3—1.2—2.1
		})
	}
}
```
7.现在可以测试下代码了
```javascript
function unit(f){ return f }
function flatMap(ma) {
	return function(g) {
		var b, h=[];
		ma(function(a) { //1. 解包`ma`取出`a`
			g = g(a); //2. 将`a`传到`g`中执行
			(g && typeof(g ) == "function"? g : unit(function(c) { c(g) })) //处理g没返回unit情况
				(function(v) {
					b = v; // 1.1—1.2—1.3
					while(h.length) h.pop()(v); //1.1—1.3—1.2
				})
		});
		return unit(function(c) { //3. 将执行结果`b`包裹成`mb`返回
			b 
			? c(b)  // 1.1—1.2—1.3—2.1
			: h.push(c) //1.1—1.3—1.2—2.1
		})
	}
}
function g0(resolve) { setTimeout(function() { resolve("0") }, 100) }
function g1(v) { console.log(v); return unit(function(resolve) { resolve(v + "->1") }) }
function g2(v) { console.log(v); return unit(function(resolve) { setTimeout(function() { resolve(v + "->2") }, 1000) }) }
function g3(v) { console.log(v) }
flatMap(
	flatMap(
		flatMap(
			unit(g0)
		)(g1)
	)(g2)
)(g3)
```

### 整合代码

现在将嵌套函数变回链式调用，这里也可以用是否有`flatMap`方法来判断`g1`是否返回容器

```javascript
function unit(ma) {
	ma.flatMap = function(g){
		var b, h=[];
		ma(function(a) { //1. 解包`ma`取出`a`
			g = g(a); //2. 将`a`传到`g`中执行
			(g && g.flatMap ? g : unit(function(c) { c(g) })) //处理g没返回unit情况
				(function(v) {
					b = v; // 1.1—1.2—1.3
					while(h.length) h.pop()(v); //1.1—1.3—1.2
				})
		});
		return unit(function(c) { //3. 将执行结果`b`包裹成`mb`返回
			b 
			? c(b)  // 1.1—1.2—1.3—2.1
			: h.push(c) //1.1—1.3—1.2—2.1
		})
	}
	return ma
}
function g0(resolve) { setTimeout(function() { resolve("0") }, 100) }
function g1(v) { console.log(v); return unit(function(resolve) { resolve(v + "->1") }) }
function g2(v) { console.log(v); return unit(function(resolve) { setTimeout(function() { resolve(v + "->2") }, 1000) }) }
function g3(v) { console.log(v) }
unit(g0).flatMap(g1).flatMap(g2).flatMap(g3)
```

## Promise是Monad吗？

将整合代码中`unit`改成`newPromise`，`flatMap`改成`then`，哇塞，除了`new Promise`中的空格请问哪里有差？虽然改成构造函数使得`newPromise`改成`new Promise`也是分分钟的事情，但重点不是这个，重点是Promise是Monad吗？粗看是！

```javascript
function newPromise(ma) {
	ma.then = function(g){
		var b, h=[];
		ma(function(a) {
			g = g(a);
			(g && g.then ? g : newPromise(function(c) { c(g) }))
				(function(v) { b = v; while(h.length) h.pop()(v) })
		});
		return newPromise(function(c) { b ? c(b) : h.push(c) })
	}
	return ma
}
newPromise(function(resolve) { setTimeout(function() { resolve("0") }, 100) })
.then(function(v) { 
	console.log(v);
	return newPromise(function(resolve) { resolve(v + "->1") }) 
})
.then(function(v) { 
	console.log(v);
return newPromise(function(resolve) { setTimeout(function() { resolve(v + "->2") }, 1000) })
})
.then(console.log)
```

### 符合定义

```haskell
unit :: a -> Monad a
flatMap :: Monad a -> (a -> Monad b) -> Monad b
```

将定义改下名

```haskell
newPromise :: a -> Monad a
then :: Monad a -> (a -> Monad b) -> Monad b
```

1. `newPromise`的输入是一个函数，但在推导构造`unit`里解释过，这里借助了立即量和闭包引用来额外增加了输入`a`，`newPromise`的参数则作为构造`unit`的补充逻辑。
2. `newPromise`的输出是`a`的包裹`Monad a`。
3. `newPromise`的方法`then`借助了闭包引用额外输入了`Monad a`，而输入的`g`函数输入是`a`输出则是借助`newPromise`实现的`Monad b`。
4. `newPromise`的方法`then`输出的是借助`newPromise`实现的`Monad b`，这里和`g`的输出`Monad b`不是同一个`Monad`但是`b`确实相同的。


### 符合法则

```javascript
flatMap(unit(x), f) === f(x) //左单位元
flatMap(monad, unit) === monad //右单位元
flatMap(flatMap(monad, f), g) === flatMap(monad, function(x) { flatMap(f(x), g) }) //关联性
```

将法则改下名，同时改为链式调用

```javascript
newPromise(x).then(f) ==== f(x) //左单位元
monad.then(newPromise) ==== monad //右单位元
monad.then(f).then(g) ==== monad.then(function(x) { f(x).then(g) }) //关联性
```

1. 左单位元法则验证代码如下：
```javascript
function newPromise(ma) {
	ma.then = function(g){
		var b, h=[];
		ma(function(a) {
			g = g(a);
			(g && g.then ? g : newPromise(function(c) { c(g) }))
				(function(v) { b = v; while(h.length) h.pop()(v) })
		});
		return newPromise(function(c) { b ? c(b) : h.push(c) })
	}
	return ma
}
var x = 1;
var f = function(v){ return v + 2 }
newPromise(function(resolve) { resolve(x) }).then(f).then(console.log) //3
console.log(f(x)) //3
```
2.右单位元法则验证代码如下：
```javascript
function newPromise(ma) {
	ma.then = function(g){
		var b, h=[];
		ma(function(a) {
			g = g(a);
			(g && g.then ? g : newPromise(function(c) { c(g) }))
				(function(v) { b = v; while(h.length) h.pop()(v) })
		});
		return newPromise(function(c) { b ? c(b) : h.push(c) })
	}
	return ma
}
newPromise(function(resolve) { resolve(1) }).then(newPromise).then(console.log) //1
newPromise(function(resolve) { resolve(1) }).then(console.log)  //1
```
3.关联性法则验证代码如下：
```javascript
function newPromise(ma) {
	ma.then = function(g){
		var b, h=[];
		ma(function(a) {
			g = g(a);
			(g && g.then ? g : newPromise(function(c) { c(g) }))
				(function(v) { b = v; while(h.length) h.pop()(v) })
		});
		return newPromise(function(c) { b ? c(b) : h.push(c) })
	}
	return ma
}
var f = function(v) { return newPromise(function(resolve) { resolve(v+2) }) }
var g = function(v) { return newPromise(function(resolve) { resolve(v+3) }) }
newPromise(function(resolve) { resolve(1) }).then(f).then(g).then(console.log) //6
newPromise(function(resolve) { resolve(1) }).then(function(x) { return f(x).then(g) }).then(console.log)  //6
```

### 如此，原来Promise是这样的Monad！

## 参考

- [Monads by Diagram](http://planspace.org/20150125-monads_by_diagram/)
- [Monads and Gonads](https://gist.github.com/newswim/4668aef8a1f1bc0dabe8)
- [Monad laws](https://wiki.haskell.org/Monad_laws)
- [A Fistful of Monads](http://learnyouahaskell.com/a-fistful-of-monads)
- [Javascript Functor, Applicative, Monads in pictures](https://medium.com/%40tzehsiang/javascript-functor-applicative-monads-in-pictures-b567c6415221#.rl7zfo6ty)
- [Functors and Applicatives](https://buzzdecafe.github.io/code/2014/10/26/functors-and-applicatives)
- [JS函数式编程指南](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/ch8.html)
- [Translation from Haskell to JavaScript of selected portions of the best introduction to monads I’ve ever read](https://blog.jcoglan.com/2011/03/05/translation-from-haskell-to-javascript-of-selected-portions-of-the-best-introduction-to-monads-ive-ever-read/)
- [Flipping arrows in coBurger King](http://blog.ezyang.com/2010/07/flipping-arrows-in-coburger-king/)
- [My angle on MonadsBackground](https://gilmoretj.wordpress.com/other-subjects/my-angle-on-monads/)
- [Awesomely descriptive JavaScript with monads](https://www.slideshare.net/wicherrr/awesomely-descriptive-javascript-with-monads)
- [Monads in JavaScript](https://curiosity-driven.org/monads-in-javascript)
- [怎样用简单的语言解释 monad？](https://www.zhihu.com/question/24972880)
- [如何解释 Haskell 中的单子？](https://www.zhihu.com/question/22291305)
- [How do I return the response from an asynchronous call?](http://stackoverflow.com/questions/14220321/how-do-i-return-the-response-from-an-asynchronous-call)
- [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
