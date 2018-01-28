---
title: Promise理解
date: 2018-01-28 23:12:13
tags: js
---

`Promise`是抽象异步处理对象以及对其进行各种操作的组件。 其详细内容在接下来我们还会进行介绍，`Promise`并不是从JavaScript中发祥的概念。

### 创建一个promise对象

要想创建一个`Promise`对象、可以使用new来调用`Promise`的构造器来进行实例化。

``` js
const fs = require('fs')
function readFile(filename) {
    return new Promise(function (resolve, reject) {
	// 异步处理
    	// 处理结束后、调用resolve 或 reject
        fs.readFile(filename, {encoding: 'utf8'}, (err, contents) => {
            if (err) {
                reject(err)
                return
            }
            resolve(contents)
        })
    })
}
```

这段代码中new了一个Promise对象，在这个构造函数中传入的参数是一个执行器，当触发这个执行器的异步操作时，并不会阻塞，而是继续向下执行，直到完成promise对象的创建。然后将这个创建的promise对象return，此时这个promise的状态为pending。

当执行器中的异步操作完成，就会将回调函数放入任务队列中，等到执行该任务时，调用resolve会将promise对象变为resolve（Fulfilled）状态,调用reject会将promise对象变为Rejected状态，此时用then方法来设置resolve后的回调函数， catch 方法来设置发生错误时的回调函数，并将回调函数加入到任务队列末尾：`promise.then(onFulfilled, onRejected)`。

在resolve(成功)时，onFulfilled 会被调用；reject(失败)时，onRejected 会被调用。resolve和reject会将参数传给onFulfilled和onRejected。onFulfilled、onRejected 两个都为可选参数。

### Promise.resolve()和Promise.reject()

静态方法`Promise.resolve(value)` 可以认为是 `new Promise()` 方法的快捷方式。比如` Promise.resolve(42)`; 可以认为是以下代码的语法糖。

``` js
new Promise(function(resolve){
    resolve(42);
});
```

在这段代码中的 resolve(42); 会让这个promise对象立即进入确定（即resolved）状态，并将 42 传递给后面then里所指定的 onFulfilled 函数。

方法 `Promise.resolve(value)`; 的返回值也是一个promise对象，所以我们可以像下面那样接着对其返回值进行 .then 调用。

``` js
Promise.resolve(42).then(function(value){
    console.log(value);
});
```

在使用`Promise.resolve(value)` 等方法的时候，如果promise对象立刻就能进入resolve状态的话，那么你是不是觉得 .then 里面指定的方法就是同步调用的呢？实际上， .then 中指定的方法调用是异步进行的。

``` js
var promise = new Promise(function (resolve){
    console.log("inner promise"); // 1
    resolve(42);
});
promise.then(function(value){
    console.log(value); // 3
});
console.log("outer promise"); // 2
```

### promise chain 中传递参数

前面例子中的Task都是相互独立的，只是被简单调用而已。这时候如果 Task A 想给 Task B 传递一个参数该怎么办呢？
答案非常简单，那就是在 Task A 中 return 的返回值，会在 Task B 执行时传给它。
我们还是先来看一个具体例子吧。

``` js
function doubleUp(value) {
    return value * 2;
}
function increment(value) {
    return value + 1;
}
function output(value) {
    console.log(value);// => (1 + 1) * 2
}
var promise = Promise.resolve(1);
promise
    .then(increment)
    .then(doubleUp)
    .then(output)
    .catch(function(error){
        // promise chain中出现异常的时候会被调用
        console.error(error);
    });
```

每个方法中 `return` 的值不仅只局限于字符串或者数值类型，也可以是对象或者promise对象等复杂类型。

`return`的值会由 `Promise.resolve(return的返回值)`; 进行相应的包装处理，因此不管回调函数中会返回一个什么样的值，最终 then的结果都是返回一个新创建的`promise`对象。也就是说， `Promise.then` 不仅仅是注册一个回调函数那么简单，它还会将回调函数的返回值进行变换，创建并返回一个`promise`对象。
