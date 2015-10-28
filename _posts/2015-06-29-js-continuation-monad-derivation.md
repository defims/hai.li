---
layout: post
title: "移动端webview点击延迟处理"
date: 2015-06-29 11:52:49
categories: frontend
tags: frontend javascript continuation monad
---

## 背景

js(javascript)揉合了[面向对象](https://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1)和[面向函数](https://zh.wikipedia.org/zh-cn/%E5%87%BD%E6%95%B8%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80)的特性，使用js解释如何从面向对象迁移到面向函数非常适合，这部分介绍js continuation monad的简明推导。

## continuation monad

[monad](https://en.wikipedia.org/wiki/Monad_(functional_programming))的一种，用于模式化[cps](https://en.wikipedia.org/wiki/Continuation-passing_style)(也就是回调风格)，monad是函数型语言处理副作用的其中一种方式，可以理解为容器（见末尾参考）

## 定义

```haskell
unit :: a -> monad a
bind :: monad a -> (a -> monad b) -> b
```

两个函数，unit函数输入一个参数a返回monad a，bind函数输入monad a和输入a返回monad b的函数，返回monad b

## 应用

这里使用一个callback hell的例子

```javascript
//四个cps的异步函数
function getFoo(param ,next) {
    setTimeout(function() {
        console.log("getFoo: "+param);
        next("foo<-"+param)
    },100)
}
function getBar(param ,next) {
    setTimeout(function() {
        console.log("getBar: "+param);
        next("bar<-"+param)
    },100)
}
function getBaz(param ,next) {
    setTimeout(function() {
        console.log("getBaz: "+param);
        next("baz<-"+param)
    },100)
}
function getSna(param ,next) {
    setTimeout(function() {
        console.log("getSna: "+param);
        next("sna<-"+param)
    },100)
}

//通常按getFoo->getBar->getBaz->getSna的顺序调用，参数通过回调函数传入
function call(param ,next) {
    getFoo(param ,function(foo) {
        getBar(foo ,function(bar) {
            getBaz(bar ,function(baz) {
                getSna(baz ,next)
            })
        })
    })
}
console.log("\ncall");
call("o",function(param){console.log("end: "+param)});

//使用unit和bind结合Array.prototype.reduce优雅解决callback hell
function call12(param ,next) {
    function unit(arg) {
        return function(f) {
            f(arg)
        }
    }
    function bind(M ,f) {
        return function(next) {
            M(function(arg) {
                f(arg ,next)
            })
        }
    }
    [getFoo ,getBar ,getBaz ,getSna].reduce(bind ,unit(param))(next)
}
console.log("\ncall12");
call12("o",function(param){console.log("end: "+param)});
```

## 推导

### 0、最初

```javascript
//按getFoo->getBar->getBaz->getSna的顺序调用，参数通过回调函数传入
function call(param ,next) {
    getFoo(param ,function(foo) {//闭包引用getFoo,param
        getBar(foo ,function(bar) {//闭包引用getBar
            getBaz(bar ,function(baz) {//闭包引用getBaz
                getSna(baz ,next)//闭包引用getSna,next
            })
        })
    })
}
console.log("\ncall");
call("o",function(param){console.log("end: "+param)});
```

### 1、扁平

```javascript
//将匿名函数转为引用，压扁嵌套回调，根据引用依赖倒置顺序
function call1(param ,next) {
    var bazNext = function(baz) {//闭包引用getSna,next
        getSna(baz ,next)
    }
    var barNext = function(bar) {//闭包引用getBaz,bazNext
        getBaz(bar ,bazNext)
    }
    var fooNext = function(foo) {//闭包引用getBar,barNext
        getBar(foo ,barNext)
    }
    getFoo(param ,fooNext)//闭包引用getFoo,param,fooNext
}
console.log("\ncall 1");
call1("o",function(param){console.log("end: "+param)});
```

### 2、fooNext闭包引用转参数引用

```javascript
//fooNext闭包引用转为参数引用，隔离依赖，抽离调用
function call2(param ,next) {
    var bazNext = function(baz) {//闭包引用getSna,next
        getSna(baz ,next)
    }
    var barNext = function(bar) {//闭包引用getBaz,bazNext
        getBaz(bar ,bazNext)
    }
    var fooNext = function(foo) {//闭包引用getBar,barNext
        getBar(foo ,barNext)
    }
    var fooM = function(next) {//闭包引用getFoo,param
        getFoo(param ,next)
    }
    fooM(fooNext)//闭包引用fooNext
}
console.log("\ncall 2");
call2("o",function(param){console.log("end: "+param)});
```

### 3、fooNext代入

```javascript
//fooNext代入，调整调用顺序
function call3(param ,next) {
    var bazNext = function(baz) {//闭包引用getSna,next
        getSna(baz ,next)
    }
    var barNext = function(bar) {//闭包引用getBaz,bazNext
        getBaz(bar ,bazNext)
    }
    var fooM = function(next) {//闭包引用getFoo,param
        getFoo(param ,next)
    }
    fooM(function(foo) {//闭包引用getBar,barNext
        getBar(foo ,barNext)
    })
}
console.log("\ncall 3");
call3("o",function(param){console.log("end: "+param)});
```

### 4、barNext闭包引用转为参数引用

```javascript
//barNext闭包引用转为参数引用，隔离依赖，抽离调用
function call4(param ,next) {
    var bazNext = function(baz) {//闭包引用getSna,next
        getSna(baz ,next)
    }
    var barNext = function(bar) {//闭包引用getBaz,bazNext
        getBaz(bar ,bazNext)
    }
    var fooM = function(next) {//闭包引用getFoo,param
        getFoo(param ,next)
    }
    var barM = function(next) {//闭包引用fooM,getBar
        fooM(function(foo) {
            getBar(foo ,next)
        })
    }
    barM(barNext)//闭包引用barNext
}
console.log("\ncall 4");
call4("o",function(param){console.log("end: "+param)});
```

### 5、barNext代入

```javascript
//barNext代入，调整调用顺序
function call5(param ,next) {
    var bazNext = function(baz) {//闭包引用getSna,next
        getSna(baz ,next)
    }
    var fooM = function(next) {//闭包引用getFoo,param
        getFoo(param ,next)
    }
    var barM = function(next) {//闭包引用fooM,getBar
        fooM(function(foo) {
            getBar(foo ,next)
        })
    }
    barM(function(bar) {//闭包引用getBaz,bazNext
        getBaz(bar ,bazNext)
    })
}
console.log("\ncall 5");
call5("o",function(param){console.log("end: "+param)});
```

### 6、bazNext闭包引用转参数引用

```javascript
//bazNext闭包引用转为参数引用，隔离依赖，抽离调用
function call6(param ,next) {
    var bazNext = function(baz) {//闭包引用getSna,next
        getSna(baz ,next)
    }
    var fooM = function(next) {//闭包引用getFoo,param
        getFoo(param ,next)
    }
    var barM = function(next) {//闭包引用fooM,getBar
        fooM(function(foo) {
            getBar(foo ,next)
        })
    }
    var bazM = function(next) {//闭包引用barM,getBaz
        barM(function(bar) {
            getBaz(bar ,next)
        })
    }
    bazM(bazNext)//闭包引用bazNext
}
console.log("\ncall 6");
call6("o",function(param){console.log("end: "+param)});
```

### 7、bazNext代入

```javascript
//bazNext代入，调整调用顺序
function call7(param ,next) {
    var fooM = function(next) {//闭包引用getFoo,param
        getFoo(param ,next)
    }
    var barM = function(next) {//闭包引用fooM,getBar
        fooM(function(foo) {
            getBar(foo ,next)
        })
    }
    var bazM = function(next) {//闭包引用barM,getBaz
        barM(function(bar) {
            getBaz(bar ,next)
        })
    }
    bazM(function(baz) {//闭包引用getSna,next
        getSna(baz ,next)
    })
}
console.log("\ncall 7");
call7("o",function(param){console.log("end: "+param)});
```

### 8、bazNext闭包引用转参数引用

```javascript
//bazNext闭包引用转为参数引用，隔离依赖，抽离调用
//现在getFoo,getBar,getBaz,getSna的依赖顺序调反过来了，且各自的next闭包引用转为参数引用
function call8(param ,next) {
    var fooM = function(next) {//闭包引用getFoo,param
        getFoo(param ,next)
    }
    var barM = function(next) {//闭包引用fooM,getBar
        fooM(function(foo) {
            getBar(foo ,next)
        })
    }
    var bazM = function(next) {//闭包引用barM,getBaz
        barM(function(bar) {
            getBaz(bar ,next)
        })
    }
    var snaM = function(next) {//闭包引用bazM,getSna
        bazM(function(baz) {
            getSna(baz ,next)
        })
    }
    snaM(next)//闭包引用next
}
console.log("\ncall 8");
call8("o",function(param){console.log("end: "+param)});
```

### 9、规整化fooM

```javascript
//根据barM，bazM，snaM用新增容器函数initM规整化fooM
function call9(param ,next) {
    var initM = function(f) {//闭包引用param
        f(param)
    }
    var fooM = function(next) {//闭包引用initM,getFoo
        initM(function(arg) {
            getFoo(arg ,next)
        })
    }
    var barM = function(next) {//闭包引用fooM,getBar
        fooM(function(foo) {
            getBar(foo ,next)
        })
    }
    var bazM = function(next) {//闭包引用barM,getBaz
        barM(function(bar) {
            getBaz(bar ,next)
        })
    }
    var snaM = function(next) {//闭包引用bazM,getSna
        bazM(function(baz) {
            getSna(baz ,next)
        })
    }
    snaM(next)//闭包引用next
}
console.log("\ncall 9");
call9("o",function(param){console.log("end: "+param)});
```

### 10、构造函数unit生成initM

```javascript
//构造函数unit生成initM，initM闭包引用转为参数引用，隔离依赖，抽离调用
function call10(param ,next) {
    function unit(arg) {
        return function(f) {
            f(arg);
        }
    }
    var initM = unit(param);//闭包引用unit,param
    var fooM = function(next) {//闭包引用initM,getFoo
        initM(function(p) {
            getFoo(p ,next)
        })
    }
    var barM = function(next) {//闭包引用fooM,getBar
        fooM(function(foo) {
            getBar(foo ,next)
        })
    }
    var bazM = function(next) {//闭包引用barM,getBaz
        barM(function(bar) {
            getBaz(bar ,next)
        })
    }
    var snaM = function(next) {//闭包引用bazM,getSna
        bazM(function(baz) {
            getSna(baz ,next)
        })
    }
    snaM(next);//闭包引用next
}
console.log("\ncall 10");
call10("o",function(param){console.log("end: "+param)});
```

### 11、构造函数bind生成fooM,barM,bazM,snaM

```javascript
//构造函数bind生成fooM,barM,bazM,snaM，bind内闭包引用变成参数引用
function call11(param ,next) {
    function unit(arg) {
        return function(f) {
            f(arg)
        }
    }
    function bind(M ,f) {
        return function(next) {
            M(function(arg) {
                f(arg ,next)
            })
        }
    }
    var initM = unit(param);//闭包引用unit,param
    var fooM = bind(initM ,getFoo);//闭包引用initM,getFoo
    var barM = bind(fooM ,getBar);//闭包引用fooM,getBar
    var bazM = bind(barM ,getBaz);//闭包引用barM,getBaz
    var snaM = bind(bazM ,getSna);//闭包引用bazM,getSna
    snaM(next);//闭包引用next
}
console.log("\ncall 11");
call11("o",function(param){console.log("end: "+param)});
```

### 12、结合reduce简洁调用bind

```javascript
//用Array.prototype.reduce使构造函数更简洁
function call12(param ,next) {
    function unit(arg) {
        return function(f) {
            f(arg)
        }
    }
    function bind(M ,f) {
        return function(next) {
            M(function(arg) {
                f(arg ,next)
            })
        }
    }
    [getFoo ,getBar ,getBaz ,getSna].reduce(bind ,unit(param))(next)
}
console.log("\ncall 12");
call12("o",function(param){console.log("end: "+param)});
```

### 13、改造unit链式调用bind

```javascript
//改造将bind合并为unit的方法，构成链式调用
function call13(param ,next) {
    function unit(arg) {
        var ret = function(f) {
            f(arg)
        }
        function bind(f) {
            var  M = this//this代替M参数
            	,ret = function(next) {
                    M(function(arg) {
                        f(arg ,next)
                    })
                }
                ;
            ret.bind = bind;//.bind().bind
            return ret
        }
        ret.bind = bind;//unit().bind
        return ret
    }
    unit(param).bind(getFoo).bind(getBar).bind(getBaz).bind(getSna)(next)
}
console.log("\ncall 13");
call13("o",function(param){console.log("end: "+param)});
```

## 总结

### 闭包引用转参数引用

上面推导过程重复性很高，其中可以看到用了很多次闭包引用转参数引用，函数型语言处理副作用的方式很多，但多数要求隔离副作用，闭包引用使得对闭包环境的依赖深，转为参数引用可以将引用显式传入利于隔离。

### 规整化

函数型语言计算函数，抽离可重用部分，利用将未知函数转换为已知函数的思想，已知函数变得可重用，规整化可以将可重用部分规律化地显现出来，利于抽离可重用部分。

## 下一步
链式调用让我们联想到了promise，难道就是monad？这两者之间有什么关系，下回分解！

## 参考

- [deriving a useful monad in javascript](http://www.reddit.com/r/javascript/comments/1fv3w5/deriving_a_useful_monad_in_javascript/)
- [你造Promise 就是 Monad 吗](http://segmentfault.com/a/1190000000613420)
- [monads in javascript](https://curiosity-driven.org/monads-in-javascript)
- [Functor, Applicative, 以及 Monad 的图片阐释](http://jiyinyiyong.github.io/monads-in-pictures/)
