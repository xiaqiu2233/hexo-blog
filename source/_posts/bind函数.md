---
title: bind函数
date: 2017-12-28 22:34:08
tags: js
---

bind函数

### 简介

存在一种情况比如你想要调用某个对象中的方法，并且该方法中会调用这个对象的某个属性，如果直接调用比如：

``` js
this.x = 9; 
var module = {
  x: 81,
  getX: function() { return this.x; }
};
module.getX(); // 返回 81
var retrieveX = module.getX;
retrieveX(); // 返回 9, 在这种情况下，"this"指向全局作用域
```

可以看到retrieveX中this指向全局，因此如果需要这个this指向module则需要使用bind函数将module.getX的this变成module。bind函数可以实现创建一个新函数，并且制定这个新函数的this值。

``` js
var retrieveX = module.getX;
// 创建一个新函数，将"this"绑定到module对象
// 新手可能会被全局的x变量和module里的属性x所迷惑
var boundGetX = retrieveX.bind(module);
boundGetX(); // 返回 81
```

### 用法

bind的第一个参数是要指定的this对象，其它都是需要传入的参数：

``` js
fun.bind(thisArg[, arg1[, arg2[, …]]])
```

当绑定函数被调用时，thisArg参数会作为原函数运行时的 this 指向。当使用new 操作符调用绑定函数时，该参数无效。

### 解释

为了解释’当使用new 操作符调用绑定函数时，该参数无效‘这句话，咱们先来归纳一下当使用new操作符来调用函数时，会执行的操作：

1. 创建（或构造）一个全新的对象。
2. 这个新对象会执行[[Prototype]]连接
3. 这个新对象会绑定到函数调用的this
4. 如果这个函数没有返回其它对象，那么new表达式中的函数调用会自动返回这个新对象。

因为使用new会将新对象绑定到函数调用的this，所以指定的this将不起作用，所以这个参数将无效。

bind函数返回由指定的this值和初始化参数改造的原函数拷贝。

bind 函数在 ECMA-262 第五版才被加入；它可能无法在所有浏览器上运行。你可以部份地在脚本开头加入以下代码，就能使它运作，让不支持的浏览器也能使用 bind() 功能。

### polyfill

``` js
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) { //oThis是thisArg
    if (typeof this !== 'function') { //this是调用bind的函数
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }
    var aArgs   = Array.prototype.slice.call(arguments, 1), //aArgs是[arg1,arg2,….]
        fToBind = this, //this是调用bind的函数(可以想象为slice)
        fNOP    = function() {},
        fBound  = function() { // bind函数返回一个函数所以fBound
          return fToBind.apply(this instanceof fNOP ? this: oThis,
                 aArgs.concat(Array.prototype.slice.call(arguments)));// 获取调用时用户向fBound传递的参数.bind 返回的函数入参往往是这么传递的
        };
    // 维护原型关系
    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype; 
    }
    fBound.prototype = new fNOP();
    return fBound;
  };
}
```
