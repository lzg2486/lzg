## 手写apply、call、bind函数
`apply`、`call`、`bind`是JavaScript中的三个重要函数，主要用于改变函数的执行上下文(`this`指向)和传递参数。

`apply`的使用方式如下，其中`context`是指将要执行的上下文对象，`argsArray`是要传递给函数的参数。
```js
function.apply(context, [argsArray])
```
`call`的使用方式如下，其中`context`是指将要执行的上下文对象，不同于`apply`是以数组的形式传递参数,`call`的参数是一个个传递的，`arg1`、`arg2`、...是要传递给函数的参数。
```js
function.call(context, arg1, arg2, ...)
```
`bind`的使用方式如下，不同于`apply`和`call`，`bind`方法用于创建一个新的函数，将指定的上下文绑定到这个新函数上，并可以预先设置一些参数，也可以在函数被调用时接收新参数，因此通过`apply`和`call`方法会立刻执行相应的函数，而`bind`方法需要手动调用
```js
function.bind(context, arg1, arg2, ...)
```
?> `JavaScript`中的函数在执行时会默认接收一个隐式参数`this`, 这个`this`就是函数被调用是的上下文**对象**。this 关键字的值取决于函数的调用方式，它可以指向不同的对象或值，或者在一些情况下指向全局对象（在浏览器中通常是 window 对象）。此外，在函数原型上定义的函数，其内部this则代表被定义的函数本身`this: function(){}`。是因此`apply`、`call`、`bind`则可以通过强行改变函数执行时的上下文**对象**来改变this指向。

### 模拟实现apply
?> 在函数原型上定义
* 接收两部分参数，一个部分是上下文对象`context`，一个部分是形参`args`。
* `context`应为可选参数，不传的情况下默认上下文是`window`。
* 判断当前this是否为函数，防止`Function.prototype.myCall()`自身调用。
* 为传递的上下文`context`创建一个`Symbol`（保证不重名）属性，该属性指向当前`this`（函数自身），即通过改变当前函数的调用环境改变上下文。
* 判断参数是否是一个数组，如果是并传递参数。
* 调用后删除`Symbol`属性不更改传递的对象。

```js
Function.prototype.myApply = function(context = window, argsArray = []) {
  // this 指定义的函数自身
  if ( this === Function.property ) {
    return undefined
  }
  // 唯一key
  const symbol = Symbol()
  // 改变函数（this）的调用环境
  context[symbol] = this
  // 在context内容调用这个函数（this）即这个函数内部的（this）执行这个context
  let result
  if ( Array.isArray(argsArray) ) {
    result = context[symbol](...argsArray);
  } else {
    result = context[symbol]();
  }
  // 删除这个多余的属性
  delete context[symbol]
  return result
}

```
*示例：通过lzg.fn()调用时，fn是在lzg内部执行，所以上下文（this）是lzg对象，通过lzg.fn().myApply调用时，fn的执行环境被改变，上下文变成test对象*
```js
const lzg = {
  name: 'lzg',
  fn() {
    console.log(`我是${this.name}`)
  }
}

lzg.fn() // 输出lzg
const test = {
  name: 'test'
}

lzg.fn.myApply(test) // 输出test
```
?> 定义为常规函数
```js
function myApply(fn, context = window, argsArray = []) {
  if ( typeof  fn !== 'function' ) {
    throw new TypeError(`${fn}不是一个函数`)
  }
  const symbol = Symbol()
  context[symbol] = fn
  let result
  if ( Array.isArray(argsArray) ) {
    result = context[symbol](...argsArray)
  } else {
    result = context[symbol]()
  }
  delete context[symbol]
  return result
}
```
!> 为什么上述this 指定义的函数自身？
```js
// 上述通过在Function原型链上定义一个myApply 方法
// 那么对于一个普通函数，我们这个可以通过其__proto__属性，找到其原型链上的方法，这就意味着每个普通函数都是拥有这个myApply方法
// 所以一个普通函数通过test.myApply()调用myApply方法时等同于下面test对象调用test.myApply()方法，因此myApply方法内部的this就是test自身
function test() {}

// 等同于
var test = {
  myApply: function () { console.log(this)}
}
```


### 模拟实现call
?> 实现类似apply，区别在于参数是通过形参的方式一个一个传递
* 接收两部分参数，一个部分是上下文对象`context`，一个部分是形参`args`。
* `context`应为可选参数，不传的情况下默认上下文是`window`。
* 判断当前this是否为函数，防止`Function.prototype.myCall()`自身调用。
* 为传递的上下文`context`创建一个`Symbol`（保证不重名）属性，该属性指向当前`this`（函数自身），即通过改变当前函数的调用环境改变上下文。
* 传递参数。
* 调用后删除`Symbol`属性不更改传递的对象。

```js
Function.prototype.myCall = function(context = window, ...args) {
  // this 指定义的函数自身
  if ( this === Function.prototype ) {
    return undefined
  }
  // 唯一key
  const symbol = Symbol()

  // 改变函数（this）的调用环境
  context[symbol] = this
  // 在context内容调用这个函数（this）即这个函数内部的（this）执行这个context
  const result = context[symbol](...args)
  // 删除这个多余的属性
  delete context[symbol]
  return result
}
```

*示例：通过lzg.fn()调用时，fn是在lzg内部执行，所以上下文（this）是lzg对象，通过lzg.fn().myCall调用时，fn的执行环境被改变，上下文变成test对象*

```js
const lzg = {
  name: 'lzg',
  fn() {
    console.log(`我是${this.name}`)
  }
}

lzg.fn() // 输出lzg
const test = {
  name: 'test'
}

lzg.fn.myCall(test) // 输出test
```
### 模拟实现bind

?> 不同于apply和call改变函数执行上下文后，会立即执行并返回相应结果，bind方法，返回的是一个改变函数执行上线文的函数，因此bind不会立即执行，如需使用还需单独调用

* 接收两部分参数，一个部分是上下文对象`context`，一个部分是形参`args`。
* `context`应为可选参数，不传的情况下默认上下文是`window`。
* 判断当前this是否为函数，防止`Function.prototype.myCall()`自身调用。
* 返回一个闭包
* 判断是否为构造函数调用，如果是则通过`new`调用当前函数，否则使用`apply`或者`call`进行处理

```js
Function.prototype.myBind = function(context = window, ...args1) {
  if ( this === Function.prototype) {
    throw new Error('Error')
  }

  const _this = this
  return function F(...args2) {
    if (this instanceof F) {
      return new _this(...args1, ...args2)
    }
    return _this.apply(context, [...args1, ...args2])
  }
}
```

*示例：通过lzg.fn()调用时，fn是在lzg内部执行，所以上下文（this）是lzg对象，通过lzg.fn().bind调用时，只是返回了一个函数（闭包），当函数被调用时，fn的执行环境被改变，上下文变成test对象*

```js
const lzg = {
  name: 'lzg',
  fn() {
    console.log(`我是${this.name}`)
  }
}

lzg.fn() // 输出lzg
const test = {
  name: 'test'
}

const newfn = lzg.fn.bind(test) // 返回一个函数

newfn() // 输出test
```

## 手写Promise
?> `异步`、`Promise`、`微任务`、`宏任务`、`async/await`的前因后果

**异步：** 在`JavaScript`中按照代码顺序执行称为同步操作，由于`JavaScript`是单线程，在某些场景如定时器、网络请求，由于代码不能立即反馈，而依赖上述反馈的代码就需要等待上述任务执行完成才能继续执行（代码阻塞）形成异步操作。

在`JavaScript`早期，为了解决异步操作，主要通过回调函数进行处理。多个回调函数的嵌套容易形成回调地狱导致代码难以理解和维护。

**Promise：** 为了解决这些问题**ECMAScript 6（ES6）**设计了`Promise`，以更结构化、可读性更强的方式处理异步编程，同时规定了**宏任务（macrotask）**和**微任务（microtask）**概念更好的管理和控制异步操作。

Promsie引入了（`pending`、`fulfilled`、`rejected`）的概念，允许异步操作的结果在将来的某个时刻被处理，当`Promsie`被解决（`fulfilled`）或拒绝（`rejected`）时，**对应的处理函数**会被放入微任务队列中等待被执行（事件轮询）。同时，`Promise`允许链式调用`.then()`方法，也可以通过`.catch()`方法来捕获异步操作的错误。

**微任务（Microtask）：** 微任务是一个非常轻量级的任务队列，用于处理异步操作，它具有更高的优先级。在`ES6`中，`Promise`的处理函数会产生微任务。当一个`Promise`被解决（`fulfilled`）或拒绝（`rejected`）时，相关的处理函数会被放入微任务队列中。这使得`Promise`的异步操作能够在下一个事件循环迭代之前执行，以确保它们具有更高的优先级，从而避免了阻塞主线程。

**宏任务（Macrotask）：** 宏任务是一个较重的任务队列，用于处理其他类型的异步操作。宏任务包括定时器回调（如`setTimeout`、`setInterval`）、I/O操作、DOM事件处理器等。这些任务通常在事件循环的下一轮中执行，相对于微任务而言优先级较低。

**async/await：** 尽管Promise解决了回调地狱的问题，但是依然需要编写嵌套的`Promise`链式调用，使得代码看起来依然复杂。`async/await`是为了进一步改善异步编程体验而引入的。

`async/await`是`Promise`的语法糖。`async/await`是建立在`Promise`之上的，它提供了一种更像同步代码的写法，使异步代码看起来更像同步执行。
使用`async`关键字声明的函数会自动返回一个`Promise`对象，这个`Promise`对象会在函数内部的异步操作完成时解决（`fulfilled`）并返回结果，或者在出现异常时拒绝（`rejected`）并抛出异常。

`await`关键字可以在`async`函数内部用于等待一个`Promise`的解决，然后获取解决后的值，这使得代码看起来更像同步代码。另外`await`后面的代码则是当前`async`被解决后**对应处理的函数**，只有这部分才会被放入**微任务**队列中
```js
async function a() {
    console.log(1) // 我只是返回一个promise对象，我并没有进行等待，所以我不需要放入微任务队列
}
async function b() {
    console.log(2) // 我并没有进行等待，所以我不需要放入微任务队列
    await a() // 等待a完成
    console.log(3) // 我是等待后要处理的函数，所以只有我会在a解决后被放入到微任务队列中
}
function c() {
    console.log(4)
    a()
    b()
    console.log(5)
}
c() // 4 1 2 1 5 3

```

