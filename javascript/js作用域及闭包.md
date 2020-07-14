# JS 作用域与闭包

### JS 作用域

Javascript 变量的作用域无非就是两种：**全局变量和局部变量**。Javascript 语言的特殊之处，就在于函数内部可以直接读取全局变量。

#### 全局作用域(Global Scope)

在代码中任何地方都能访问到的对象拥有全局作用域，一般来说一下几种情形拥有全局作用域：

1. 最外层函数和在最外层函数外面定义的变量拥有全局作用域

```js
var name = 'Jack' // 全局定义
function foo() {
  var age = 23 // 局部定义
  function inner() {
    // 局部函数
    console.log(age) //age 23
  }
  inner()
}

console.log(name) // yuan
console.log(age) // Uncaught ReferenceError: age is not defined，在外部没有这个变量
foo() // 内嵌函数的打印23
inner() // Uncaught ReferenceError: inner is not defined 因为内嵌函数，找不到这个函数
```

2. 所有末定义直接赋值的变量自动声明为拥有全局作用域

```js
var name = 'yuan'
function foo() {
  age = 23 // 全局定义
  var sex = 'male' // 局部定义
}
foo()
console.log(age) //  23
console.log(sex) // sex is not defined
```

3. 所有 window 对象的属性拥有全局作用域
   一般情况下，window 对象的内置属性都都拥有全局作用域，例如 window.alert()、window.location、window.top 等等。

### 作用域链

当代码在一个环境中执行时，会创建变量对象的一个作用域链（作用域形成的链条）

1. 作用域链的前端，始终都是当前执行的代码所在环境的变量对象
2. 作用域链中的下一个对象来自于外部环境，而在下一个变量对象则来自下一个外部环境，一直到全局执行环境
3. 全局执行环境的变量对象始终都是作用域链上的最后一个对象

当在内部函数中，需要访问一个变量的时候，首先会访问函数本身的变量对象，是否有这个变量，如果没有，那么会继续沿作用域链往上查找，直到全局作用域。如果在某个变量对象中找到则使用该变量对象中的变量值。

#### 内部环境可以通过作用域链访问所有外部环境，但外部环境不能访问内部环境的任何变量和函数。

### 变量提升

ES6 之前我们一般使用 var 来声明变量，变量提升如下例子：

```js
function test() {
  console.log(a) //undefined
  var a = 123
}

// 它的实际执行顺序如下
function test() {
  var a
  console.log(a)
  a = 123
}
test()
```

### 函数提升

javascript 中不仅仅是变量声明有提升的现象，函数的声明也是一样。具名函数的声明有两种方式：

- 函数声明式
- 函数字面量式

```js
//函数声明式
function bar() {}
//函数字面量式
var foo = function() {}
```

函数提升是整个代码块提升到它所在的作用域的最开始执行

```js
console.log(bar)
function bar() {
  console.log(1) //ƒ bar () { console.log(1)}
}

// 实际执行顺序
function bar() {
  console.log(1)
}
console.log(bar)
```

### 闭包

闭包就是能够读取其他函数内部变量的函数，函数没有被释放，整条作用域链上的局部变量都将得到保留。由于在 javascript 语言中，只有函数内部的子函数才能读取局部变量，**因此可以把闭包简单理解成“定义在一个函数内部的函数”**。

#### 产生一个闭包

创建闭包最常见方式，就是在一个函数内部创建另一个函数。闭包的作用域链包含着它自己的作用域，以及包含它的函数的作用域和全局作用域。

```js
function fn() {
  var a = 1,
    b = 2

  function f1() {
    return a + b
  }
  return f1
}
```

上面例子中的 f1 就是一个闭包

#### 闭包的应用

1. 设计私有的方法和变量。
   任何在函数中定义的变量，都可以认为是私有变量，因为不能在函数外部访问这些变量。私有变量包括函数的参数、局部变量和函数内定义的其他函数。

   把有权访问私有变量的公有方法称为特权方法（privileged method）。

```js
function Animal() {
  // 私有变量
  var series = '哺乳动物'
  function run() {
    console.log('Run!!!')
  }

  // 特权方法
  this.getSeries = function() {
    return series
  }
}
```

2. 匿名函数最大的用途是创建闭包。减少全局变量的使用。从而使用闭包模块化代码，减少全局变量的污染。

```js
var objEvent = objEvent || {}
(function() {
  var addEvent = function() {
    // some code
  }
  function removeEvent() {
    // some code
  }

  objEvent.addEvent = addEvent
  objEvent.removeEvent = removeEvent
})()
```

addEvent 和 removeEvent 都是局部变量，但我们可以通过全局变量 objEvent 使用它，这就大大减少了全局变量的使用，增强了网页的安全性。

3. 定义模块，我们将操作函数暴露给外部，而细节隐藏在模块内部。
```js
function module() {
	var arr = [];
	function add(val) {
		if (typeof val == 'number') {
			arr.push(val);
		}
	}
	function get(index) {
		if (index < arr.length) {
			return arr[index]
		} else {
			return null;
		}
	}
	return {
		add: add,
		get: get
	}
}
var mod1 = module();
mod1.add(1);
mod1.add(2);
mod1.add('xxx');
console.log(mod1.get(2));
```

#### 使用闭包的注意点

1. 由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在 IE 中可能导致内存泄露。解决方法时，在退出函数之前，将不使用的局部变量全部删除。

```js
function makeAdder(x) {
  return function(y) {
    return x + y
  }
}

var add5 = makeAdder(5)
var add10 = makeAdder(10)

console.log(add5(2)) // 7
console.log(add10(2)) // 12

// 释放对闭包的引用
add5 = null
add10 = null
```

add5 和 add10 都是闭包。它们共享相同的函数定义，但是保存了不同的环境。在 add5 的环境中，x 为 5。而在 add10 中，x 则为 10。最后通过 null 释放了 add5 和 add10 对闭包的引用。

2. 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象(object)使用，把闭包当作它的公用方法，把内部变量当作它的私有属性，这时一定要小心，不要随便改变父函数内部变量的值。
