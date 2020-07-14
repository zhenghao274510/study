## 前言

高阶函数是对其他函数进行操作的函数，可以将它们作为参数或通过返回它们。简单来说，高阶函数是一个函数，它接收函数作为参数或将函数作为输出返回。

例如`Array.prototype.map`，`Array.prototype.filter`，`Array.prototype.reduce` 都是一些高阶函数。

## 尾调用和尾递归

尾调用（Tail Call）是函数式编程的一个重要概念，本身非常简单，一句话就能说清楚。就是指某个函数的最后一步是调用另一个函数。

```js
function g(x) {
  console.log(x)
}
function f(x) {
  return g(x)
}
console.log(f(1))
//上面代码中，函数f的最后一步是调用函数g，这就是尾调用。
```

上面代码中，函数 f 的最后一步是调用函数 g，这就是尾调用。尾调用不一定出现在函数尾部，只要是最后一步操作即可。

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生栈溢出错误。但是队伍尾递归来说，由于只存在一个调用帧，所以永远不会发生栈溢出错误。

```js
function factorial(n) {
  if (n === 1) {
    return 1
  }
  return n * factorial(n - 1)
}
```

上面代码是一个阶乘函数，计算 n 的阶乘，最多需要保存 n 个调用数据，复杂度为 O（n），如果改写成尾调用，只保留一个调用记录，复杂度为 O（1）。

```js
function factor(n, total) {
  if (n === 1) {
    return total
  }
  return factor(n - 1, n * total)
}
```

斐波拉切数列也是可以用于尾调用。

```js
function Fibonacci(n) {
  if (n <= 1) {
    return 1
  }
  return Fibonacci(n - 1) + Fibonacci(n - 2)
}
//尾递归
function Fibona(n, ac1 = 1, ac2 = 1) {
  if (n <= 1) {
    return ac2
  }
  return Fibona(n - 1, ac2, ac1 + ac2)
}
```

## 柯理化函数

在数学和计算机科学中，柯里化是一种将使用多个参数的一个函数转换成一系列使用一个参数的函数的技术。**所谓柯里化就是把具有较多参数的函数转换成具有较少参数的函数的过程。**  
举个例子

```javascript
//普通函数
function fn(a, b, c, d, e) {
  console.log(a, b, c, d, e)
}
//生成的柯里化函数
let _fn = curry(fn)

_fn(1, 2, 3, 4, 5) // print: 1,2,3,4,5
_fn(1)(2)(3, 4, 5) // print: 1,2,3,4,5
_fn(1, 2)(3, 4)(5) // print: 1,2,3,4,5
_fn(1)(2)(3)(4)(5) // print: 1,2,3,4,5
```

柯理化函数的实现

```javascript
// 对求和函数做curry化
let f1 = curry(add, 1, 2, 3)
console.log('复杂版', f1()) // 6

// 对求和函数做curry化
let f2 = curry(add, 1, 2)
console.log('复杂版', f2(3)) // 6

// 对求和函数做curry化
let f3 = curry(add)
console.log('复杂版', f3(1, 2, 3)) // 6

// 复杂版curry函数可以多次调用，如下：
console.log('复杂版', f3(1)(2)(3)) // 6
console.log('复杂版', f3(1, 2)(3)) // 6
console.log('复杂版', f3(1)(2, 3)) // 6

// 复杂版(每次可传入不定数量的参数，当所传参数总数不少于函数的形参总数时，才会执行)
function curry(fn) {
  // 闭包
  // 缓存除函数fn之外的所有参数
  let args = Array.prototype.slice.call(arguments, 1)
  return function () {
    // 连接已缓存的老的参数和新传入的参数(即把每次传入的参数全部先保存下来，但是并不执行)
    let newArgs = args.concat(Array.from(arguments))
    if (newArgs.length < fn.length) {
      // 累积的参数总数少于fn形参总数
      // 递归传入fn和已累积的参数
      return curry.call(this, fn, ...newArgs)
    } else {
      // 调用
      return fn.apply(this, newArgs)
    }
  }
}
```

### 柯里化的用途

柯里化实际是把简答的问题复杂化了，但是复杂化的同时，我们在使用函数时拥有了更加多的自由度。 而这里对于函数参数的自由处理，正是柯里化的核心所在。 柯里化本质上是降低通用性，提高适用性。来看一个例子：

我们工作中会遇到各种需要通过正则检验的需求，比如校验电话号码、校验邮箱、校验身份证号、校验密码等， 这时我们会封装一个通用函数 checkByRegExp ,接收两个参数，校验的正则对象和待校验的字符串

```javascript
function checkByRegExp(regExp, string) {
  return regExp.text(string)
}

checkByRegExp(/^1\d{10}$/, '18642838455') // 校验电话号码
checkByRegExp(/^(\w)+(\.\w+)*@(\w)+((\.\w+)+)$/, 'test@163.com') // 校验邮箱
```

我们每次进行校验的时候都需要输入一串正则，再校验同一类型的数据时，相同的正则我们需要写多次， 这就导致我们在使用的时候效率低下，并且由于 checkByRegExp 函数本身是一个工具函数并没有任何意义。此时，我们可以借助柯里化对 checkByRegExp 函数进行封装，以简化代码书写，提高代码可读性。

```javascript
//进行柯里化
let _check = curry(checkByRegExp)
//生成工具函数，验证电话号码
let checkCellPhone = _check(/^1\d{10}$/)
//生成工具函数，验证邮箱
let checkEmail = _check(/^(\w)+(\.\w+)*@(\w)+((\.\w+)+)$/)

checkCellPhone('18642838455') // 校验电话号码
checkCellPhone('13109840560') // 校验电话号码
checkCellPhone('13204061212') // 校验电话号码

checkEmail('test@163.com') // 校验邮箱
checkEmail('test@qq.com') // 校验邮箱
checkEmail('test@gmail.com') // 校验邮箱
```

### 柯里化函数参数 length

函数 currying 的实现中，使用了 fn.length 来表示函数参数的个数，那 fn.length 表示函数的所有参数个数吗？并不是。

函数的 length 属性获取的是形参的个数，但是形参的数量不包括剩余参数个数，而且仅包括第一个具有默认值之前的参数个数，看下面的例子。

```js
;((a, b, c) => {})
  .length(
    // 3

    (a, b, c = 3) => {}
  )
  .length(
    // 2

    (a, b = 2, c) => {}
  )
  .length(
    // 1

    (a = 1, b, c) => {}
  )
  .length(
    // 0

    (...args) => {}
  ).length
// 0

const fn = (...args) => {
  console.log(args.length)
}
fn(1, 2, 3)
// 3
```

## compose 函数

compose 就是组合函数，将子函数串联起来执行，一个函数的输出结果是另一个函数的输入参数，一旦第一个函数开始执行，会像多米诺骨牌一样推导执行后续函数。

```js
const greeting = (name) => `Hello ${name}`
const toUpper = (str) => str.toUpperCase()

toUpper(greeting('Onion')) // HELLO ONION
```

compose 函数的特点

- compose 接受函数作为参数，从右向左执行，返回类型函数
- fn()全部参数传给最右边的函数，得到结果后传给倒数第二个，依次传递

compose 的实现

```js
var compose = function (...args) {
  var len = args.length // args函数的个数
  var count = len - 1
  var result
  return function func(...args1) {
    // func函数的args1参数枚举
    result = args[count].call(this, args1)
    if (count > 0) {
      count--
      return func.call(null, result) // result 上一个函数的返回结果
    } else {
      //回复count初始状态
      count = len - 1
      return result
    }
  }
}
```

举个例子

```js
var greeting = (name) => `Hello ${name}`
var toUpper = (str) => str.toUpperCase()
var fn = compose(toUpper, greeting)
console.log(fn('jack'))
```

大家熟悉的 webpack 里面的 loader 执行顺序是从右到左，是因为 webpack 选择的是 compose 方式，从右到左依次执行 loader，每个 loader 是一个函数。

```js
rules: [{ test: /\.css$/, use: ['style-loader', 'css-loader'] }]
```

如上，webpack 使用了 style-loader 和 css-loader，它是先用 css-loader 加载.css 文件，然后 style-loader 将内部样式注入到我们的 html 页面。

webpack 里面的 compose 代码如下：

```js
const compose = (...fns) => {
  return fns.reduce(
    (prevFn, nextFn) => {
      return (value) => prevFn(nextFn(value))
    },
    (value) => value
  )
}
```
