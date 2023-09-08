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
* 接收两个参数，一个上下文对象`context`，一个参数数组`argsArray`。
* `context`应为可选参数，不传的情况下默认上下文是`window`。
* 判断当前this是否为函数，防止`Function.prototype.myCall()`自身调用。
* 为传递的上下文`context`创建一个`Symbol`（保证不重名）属性，该属性指向当前`this`（函数自身），即通过改变当前函数的调用环境改变上下文。
* 传递参数
* 调用后删除`Symbol`属性不更改传递的对象

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

## Initialize

If you want to write the documentation in the `./docs` subdirectory, you can use the `init` command.

```bash
docsify init ./docs
```

## Writing content

After the `init` is complete, you can see the file list in the `./docs` subdirectory.

- `index.html` as the entry file
- `README.md` as the home page
- `.nojekyll` prevents GitHub Pages from ignoring files that begin with an underscore

You can easily update the documentation in `./docs/README.md`, of course you can add [more pages](more-pages.md).

## Preview your site

Run the local server with `docsify serve`. You can preview your site in your browser on `http://localhost:3000`.

```bash
docsify serve docs
```

?> For more use cases of `docsify-cli`, head over to the [docsify-cli documentation](https://github.com/docsifyjs/docsify-cli).

## Manual initialization

If you don't like `npm` or have trouble installing the tool, you can manually create `index.html`:

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <meta charset="UTF-8" />
    <link
      rel="stylesheet"
      href="//cdn.jsdelivr.net/npm/docsify@4/themes/vue.css"
    />
  </head>
  <body>
    <div id="app"></div>
    <script>
      window.$docsify = {
        //...
      };
    </script>
    <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  </body>
</html>
```

### Specifying docsify versions

?> Note that in both of the examples below, docsify URLs will need to be manually updated when a new major version of docsify is released (e.g. `v4.x.x` => `v5.x.x`). Check the docsify website periodically to see if a new major version has been released.

Specifying a major version in the URL (`@4`) will allow your site will receive non-breaking enhancements (i.e. "minor" updates) and bug fixes (i.e. "patch" updates) automatically. This is the recommended way to load docsify resources.

```html
<link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/themes/vue.css" />
<script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
```

If you prefer to lock docsify to a specific version, specify the full version after the `@` symbol in the URL. This is the safest way to ensure your site will look and behave the same way regardless of any changes made to future versions of docsify.

```html
<link
  rel="stylesheet"
  href="//cdn.jsdelivr.net/npm/docsify@4.11.4/themes/vue.css"
/>
<script src="//cdn.jsdelivr.net/npm/docsify@4.11.4"></script>
```

### Manually preview your site

If you have Python installed on your system, you can easily use it to run a static server to preview your site.

```python2
cd docs && python -m SimpleHTTPServer 3000
```

```python3
cd docs && python -m http.server 3000
```

## Loading dialog

If you want, you can show a loading dialog before docsify starts to render your documentation:

```html
<!-- index.html -->

<div id="app">Please wait...</div>
```

You should set the `data-app` attribute if you changed `el`:

```html
<!-- index.html -->

<div data-app id="main">Please wait...</div>

<script>
  window.$docsify = {
    el: '#main',
  };
</script>
```

Compare [el configuration](configuration.md#el).
