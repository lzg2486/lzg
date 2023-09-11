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

