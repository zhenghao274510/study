# JS 数据类型

**JavaScript 是弱类型语言**，而且 JavaScript 声明变量的时候并没有预先确定的类型，变量的类型就是其值的类型，也就是说变量当前的类型由其值所决定,夸张点说上一秒种的 String，下一秒可能就是个 Number 类型了，这个过程可能就进行了某些操作发生了强制类型转换。

js 数据分为两种类型：原始数据类型和引用数据类型。

- 基本数据类型有：string、number、boolean、undefined、null 和 symbol（符号）
- 引用数据类型有：Object、Function、Date、RegExp 等。

### 栈和堆
堆和栈的概念存在于数据结构中和操作系统内存中。

在数据结构中，栈中数据的存取方式为先进后出。而堆是一个优先队列，是按优先级来进行排序的，优先级可以按照大小来规定。完全
二叉树是堆的一种实现方式。

在操作系统中，内存被分为栈区和堆区。
栈区内存由编译器自动分配释放，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。
堆区内存一般由程序员分配释放，若程序员不释放，程序结束时可能由垃圾回收机制回收。

基本数据类型直接存储在栈（stack）中的简单数据段，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储。

引用数据类型存储在堆（heap）中的对象，占据空间大、大小不固定。如果存储在栈中，将会影响程序运行的性能；引用数据类型在
栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实
体。

### 基本数据类型的特点

1. 基本数据类型是按值访问的，就是说我们可以操作保存在变量中的实际的值；
2. 基本数据类型的值是不可变的，任何方法都无法改变一个基本数据类型的值

```JS
  let name = 'zhangsan'
  name.substr()
  console.log(name) // 输出：zhangsan
  let age = 'firstblood'
  age.toUpperCase()
  console.log(age) // 输出：firstblood
```

substr()和 toUpperCase()方法后返回的是一个新的字符串，跟原来定义的变量 name 并没有什么关系。

3. 基本数据类型不可以添加属性和方法

```js
let user = 'zhangsan'
user.age = 18
user.method = function() {
  console.log('12345')
}
console.log(user.age) // 输出：undefined
console.log(user.method) // 输出：undefined
```

4. 基本数据类型的赋值是简单的赋值(不影响原变量的值)

```js
let a = 18
let b = a
a++
console.log(a) // 输出：19
console.log(b) // 输出：18
```

5. 基本数据类型的比较是值的比较

```js
var a = '{}'
var b = '{}'
console.log(a === b) // 输出：true
```

6. 基本类型的值在内存中占据固定大小的空间，被保存在栈内存中

### 引用数据类型

引用类型是存放在堆内存中的对象，变量其实是保存的在栈内存中的一个指针（保存的是堆内存中的引用地址），这个指针指向堆内存。

```js
var obj1 = new Object()
var obj2 = obj1
obj2.name = '我有名字了'

console.log(obj1.name) // 我有名字了
```

### 数据类型转换

#### 转为字符串

1、toString()方法：注意，不可以转 null 和 underfined
2、String()方法：都能转

```js
let ab = 'zhangsan'
let bc = null
let cd = undefined
console.log(ab.toString()) // 输出：zhangsan
console.log(bc.toString()) // error 报错
console.log(cd.toString()) // error 报错
console.log(String(ab)) // 输出：zhangsan
console.log(String(bc)) // 输出：null
console.log(String(cd)) // 输出：undefined
```

3、隐式转换：num + ""，当 + 两边一个操作符是字符串类型，一个操作符是其它类型的时候，会先把其它类型转换成字符串再进行字符串拼接，返回字符串

```js
var a = true
var str = a + ''
console.log('str')
```

#### 转为数值类型

1、Number()：可以把任意值转换成数值，如果要转换的字符串中有一个不是数值的字符，返回 NaN

2、parseInt()/parseFloat():parseFloat()把字符串转换成浮点数,parseFloat()和 parseInt 非常相似，不同之处在与 parseFloat 会解析第一个. 遇到第二个.或者非数字结束如果解析的内容里只有整数，解析成整数。

```js
var a = '12.3px'
console.log(parseInt(a)) // 12
console.log(parseFloat(a)) // 12.3
let b = 'abc2.3'
console.log(parseInt(b)) //NAN
console.log(parseFloat(b)) //NAN
```

3、隐式转换

```js
var str = '123'
var num = str - 1
console.log(num) // 122
```

4、isNaN()函数用于判断是否是一个非数字类型，如果传入的参数是一个非数字类型，那么返回 true，否则返回 false

#### 转换为 Boolean()

除了 0 ''(空字符串) null undefined NaN 会转换成 false 其它都会转换成 true

### 判断 JS 数据类型

#### 1、typeof()函数

对于原始数据类型，我们可以使用 typeof()函数来判断他的数据类型。但他是没法用来区分引用数据类型的，因为所有的引用数据类型都会返回"object"。

```js
typeof 'seymoe' // 'string'
typeof true // 'boolean'
typeof 10 // 'number'
typeof Symbol() // 'symbol'
typeof null // 'object' 无法判定是否为 null
typeof undefined // 'undefined'

typeof {} // 'object'
typeof [] // 'object'
typeof (() => {}) // 'function'
```

#### 2、instanceof

对于引用类型我们使用 instanceof 来进行类型判断。

```javascript
var obj = {}
obj instanceof Object //true
var arr = []
arr instanceof Array //true
```

#### 3、Object.prototype.toString.call()

在 javascript 高级程序设计中提供了另一种方法，可以通用的来判断原始数据类型和引用数据类型

```javascript
var arr = []
Object.prototype.toString.call(arr) == '[object Array]' //true

var func = function() {}
Object.prototype.toString.call(func) == '[object Function]' //true
```

#### 4、constructor

constructor 作用和 instanceof 非常相似。但 constructor 检测 Object 与 instanceof 不一样，还可以处理基本数据类型的检测。

```javascript
var aa = [1, 2]
console.log(aa.constructor === Array) //true
console.log(aa.constructor === RegExp) //false
console.log((1).constructor === Number) //true
var reg = /^$/
console.log(reg.constructor === RegExp) //true
console.log(reg.constructor === Object) //false
```

### javascript 的内置方法

- toString()方法返回一个表示该对象的字符串。
- valueOf()方法返回指定对象的原始值。

```js
var str = new String('123')
console.log(str.valueOf()) //123

var num = new Number(123)
console.log(num.valueOf()) //123

var date = new Date()
console.log(date.valueOf()) //1526990889729

var bool = new Boolean('123')
console.log(bool.valueOf()) //true

var obj = new Object({
  valueOf: () => {
    return 1
  }
})
console.log(obj.valueOf()) //1
```

**valueOf() 和 toString()在特定的场合下会自行调用。**

### 包装对象（wrapper object）

先来看一个例子

```js
let name = 'marko'
console.log(typeof name) // "string"
console.log(name.toUpperCase()) // "MARKO"
```

name 类型是 string，属于基本类型，所以它没有属性和方法，但是在这个例子中，我们调用了一个 toUpperCase()方法，它不会抛出错误，还返回了对象的变量值。 原因是基本类型的值被临时转换或强制转换为对象，因此 name 变量的行为类似于对象。name.toUpperCase()在幕后看起来如下：

```js
console.log(new String(name).toUpperCase()) // "MARKO"
```

除 null 和 undefined 之外的每个基本类型都有自己包装对象。也就是：String，Number，Boolean，Symbol 和 BigInt。
