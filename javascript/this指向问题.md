# this 指向

使用 JavaScript 开发的时候，很多开发者多多少少会被 this 的指向搞蒙圈，但是实际上，关于 this 的指向，记住最核心的一句话：**哪个对象调用函数，函数里面的 this 指向哪个对象。**

### 1、普通函数：谁调用指向谁

全局变量指向全局对象-window

```js
var username = 'cn'
function fn() {
  alert(this.username) //cn
}
fu()
```

有一点需要注意，let 声明的全局变量，不是指向 window 对象

```js
let username = 'cn'
function fn() {
  alert(this.username) //undefined
}
fn()
```

### 2、对象函数调用

就是那个函数调用，this 指向哪里

```js
window.b = 2222
let obj = {
  a: 111,
  fn: function() {
    alert(this.a) //111
    alert(this.b) //undefined
  }
}
obj.fn()
```

### 3.构造函数中调用
JS里的普通函数可以使用new操作符来创建一个对象，此时该函数就是一个构造函数，箭头函数不能作为构造函数。执行new操作符，其实JS内部完成了以下事情：

1. 创建一个空的简单JavaScript对象（即{}）；
1. 将构造函数的prototype绑定为新对象的原型对象 ；
1. 将步骤1新创建的对象作为this的上下文并执行函数 ；
1. 如果该函数没有返回对象，则返回this。

```js
function A () {
  this.a = 1
  this.func = () => {
    return this
  }
}

let obj = new A()
console.log(obj.a) // 1
console.log(obj.func() === obj) // true
```

### 4.箭头函数中调用
箭头函数的this指向，和箭头函数定义所在上下文的this相同。对于普通函数，this在函数调用时才确定；而对于箭头函数，this在箭头函数定义时就已经确定了，并且不能再被修改。
```js
let obj = {
  A () {
    return () => {
      return this
    }
  },
  B () {
    return function () {
      return this
    }
  }
}

let func = obj.A()
console.log(func() === obj) // true

func = obj.B()
console.log(func() === obj) // false
console.log(func() === window) // true
```


### apply、call、bind

在 javascript 中，call 和 apply 都是为了改变某个函数运行时的上下文（context）而存在的，换句话说，就是为了改变函数体内部 this 的指向。
举个例子

```javascript
function fruits() {}

fruits.prototype = {
  color: 'red',
  say: function() {
    console.log('My color is ' + this.color)
  }
}

var apple = new fruits()
apple.say() //My color is red

// 但是如果我们有一个对象banana= {color : "yellow"} ,我们不想对它重新定义 say 方法，
//那么我们可以通过 call 或 apply 用 apple 的 say 方法：
banana = {
  color: 'yellow'
}
apple.say.call(banana) //My color is yellow
apple.say.apply(banana) //My color is yellow
```

#### apply、call 区别

对于 apply、call 二者而言，作用完全一样，只是接受参数的方式不太一样

```javascript
func.call(this, arg1, arg2)
func.apply(this, [arg1, arg2])
```

apply、call 实例

```javascript
// 数组追加
var array1 = [12 , "foo" , {name:"Joe"} , -2458];
var array2 = ["Doe" , 555 , 100];
Array.prototype.push.apply(array1, array2);
// array1 值为  [12 , "foo" , {name:"Joe"} , -2458 , "Doe" , 555 , 100]

// 获取数组中的最大值和最小值
var  numbers = [5, 458 , 120 , -215 ];
var maxInNumbers = Math.max.apply(Math, numbers),   //458
var maxInNumbers = Math.max.call(Math,5, 458 , 120 , -215); //458

// 验证是否是数组
functionisArray(obj){
  return Object.prototype.toString.call(obj) === '[object Array]'
}
```

#### bind()

bind()最简单的用法是创建一个函数，使这个函数不论怎么调用都有同样的 this 值。
bind()方法会创建一个新函数，称为绑定函数，当调用这个绑定函数时，绑定函数会以创建它时传入 bind()方法的第一个参数作为 this，传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。

```javascript
this.num = 9
var mymodule = {
  num: 81,
  getNum: function() {
    console.log(this.num)
  }
}

mymodule.getNum() // 81

var getNum = mymodule.getNum
getNum() // 9, 因为在这个例子中，"this"指向全局对象

var boundGetNum = getNum.bind(mymodule)
boundGetNum() // 81
```

当调用 bind 函数后，bind 函数的第一个参数就是原函数作用域中 this 指向的值

```javascript
function func() {
  console.log(this)
}

let newFunc = func.bind({ a: 1 })
newFunc() // 打印：{a:1}

let newFunc2 = func.bind([1, 2, 3])
newFunc2() // 打印：[1,2,3]

let newFunc3 = func.bind(1)
newFunc3() // 打印：Number:{1}

let newFunc4 = func.bind(undefined / null)
newFunc4() // 打印：window
```

- 当传入为 null 或者 undefined 时，在非严格模式下，this 指向为 window。
- 当传入为简单值时，内部会将简单的值包装成对应类型的对象，数字就调用 Number 方法包装；字符串就调用 String 方法包装；true/false 就调用 Boolean 方法包装。要想取到原始值，可以调用 valueOf 方法。

传递的参数的顺序问题

```javascript
function func(a, b, c) {
  console.log(a, b, c) // 打印传入的实参
}

let newFunc = func.bind({}, 1, 2)

newFunc(3) //1,2,3
// 可以看到，在 bind 中传递的参数要先传入到原函数中。
```

返回的新函数被当成构造函数

```javascript
// 原函数
function func(name) {
  console.log(this) // 打印：通过{name:'wy'}
  this.name = name
}
func.prototype.hello = function() {
  console.log(this.name)
}
let obj = { a: 1 }
// 调用bind,返回新函数
let newFunc = func.bind(obj)

// 把新函数作为构造函数,创建实例

let o = new newFunc('seven')

console.log(o.hello()) // 打印：'seven'
console.log(obj) // 打印：{a:1}
```

新函数被当成了构造函数，原函数 func 中的 this 不再指向传入给 bind 的第一个参数，而是指向用 new 创建的实例。在通过实例 o 找原型上的方法 hello 时，能够找到原函数 func 原型上的方法。

#### apply、call、bind 比较

- apply 、call 、bind 三者都是用来改变函数的 this 对象的指向的；
- apply 、call 、bind 三者第一个参数都是 this 要指向的对象，也就是想指定的上下文；
- apply 、call 、bind 三者都可以利用后续参数传参；
- bind 是返回对应函数，便于稍后调用；apply 、call 则是立即调用 。

```javascript
var obj = {
  x: 81
}

var foo = {
  getX: function() {
    return this.x
  }
}

console.log(foo.getX.bind(obj)()) //81
console.log(foo.getX.call(obj)) //81
console.log(foo.getX.apply(obj)) //81
```
