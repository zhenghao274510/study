# JS原型和原型链

### 构造函数

> 每个构造函数(constructor)都有一个原型对象(prototype), 原型对象都包含一个指向构造函数的指针, 而实例(instance)都包含一个指向原型对象的内部指针.

我们先来看一个例子

```js
function Person(name, age, job) {
  this.name = name
  this.age = age
  this.job = job
  this.sayName = function() {
    alert(this.name)
  }
}
var person1 = new Person('Zaxlct', 28, 'Engineer')
var person2 = new Person('Mick', 23, 'Doctor')
```

上面的例子中 person1 和 person2 都是 Person 的实例。这两个实例都有一个 constructor （构造函数）属性，该属性（是一个指针）指向 Person。 即：

```js
console.log(person1.constructor == Person) //true
console.log(person2.constructor == Person) //true
```

### prototype

每个构造函数都有一个 prototype 属性，指向调用该构造函数而创建的实例的原型，也就是这个例子中的 person1 和 person2 的原型。

```js
function Person() {}

Person.prototype.name = 'Zaxlct'
Person.prototype.age = 28
Person.prototype.job = 'Engineer'
Person.prototype.sayName = function() {
  alert(this.name)
}

var person1 = new Person()
person1.sayName() // 'Zaxlct'

var person2 = new Person()
person2.sayName() // 'Zaxlct'

console.log(person1.sayName == person2.sayName) //true
```

### **proto**

这是每一个 JavaScript 对象(除了 null )都具有的一个属性，叫**proto**，这个属性会指向该对象的原型。

```js
function Person() {}
var person1 = new Person()
console.log(person1.__proto__ === Person.prototype) // true
```

### constructor

每个原型都有一个 constructor 属性指向关联的构造函数

```js
function Person() {}
var person1 = new Person()
console.log(Person === Person.prototype.constructor) // true
console.log(person1.__proto__ === Person.prototype) // true
```

### 实例与原型

当读取实例的属性时，如果找不到，就会查找与对象关联的原型中的属性，如果还查不到，就去找原型的原型，一直找到最顶层为止。

```js
function Person() {}

Person.prototype.name = 'Kevin'

var person = new Person()

person.name = 'Daisy'
console.log(person.name) // Daisy

delete person.name
console.log(person.name) // Kevin
```

在这个例子中，我们给实例对象 person 添加了 name 属性，当我们打印 person.name 的时候，结果自然为 Daisy。

但是当我们删除了 person 的 name 属性时，读取 person.name，从 person 对象中找不到 name 属性就会从 person 的原型也就是 person.**proto** ，也就是 Person.prototype 中查找，幸运的是我们找到了 name 属性，结果为 Kevin。

### Object.create()

语法：`Object.create(proto, [propertiesObject])`

方法创建一个新对象，使用现有的对象来提供新创建的对象的 proto。

- new Object() 通过构造函数来创建对象, 添加的属性是在自身实例下。
- Object.create() es6 创建对象的另一种方式，可以理解为继承一个对象, 添加的属性是在原型下。

```js
// new Object() 方式创建
var a = { rep: 'apple' }
var b = new Object(a)
console.log(b) // {rep: "apple"}
console.log(b.__proto__) // {}
console.log(b.rep) // {rep: "apple"}

// Object.create() 方式创建
var a = { rep: 'apple' }
var b = Object.create(a)
console.log(b) // {}
console.log(b.__proto__) // {rep: "apple"}
console.log(b.rep) // {rep: "apple"}
```

经典面试题

```js
var obj1 = { name: 'one' }
obj2 = Object.create(obj1)
obj2.name = 'two'
console.log(obj1.name)
//one

var obj1 = { prop: { name: 'one' } }
obj2 = Object.create(obj1)
obj2.prop.name = 'two'
console.log(obj1.prop.name)
//two

var obj1 = { list: ['one', 'one', 'one'] }
obj2 = Object.create(obj1)
obj2.list[0] = 'two'
console.log(obj1.list[0])
//two
```

- 第二题先计算 obj2.prop 的值，在原型链中被发现，然后再计算 obj2.prop 对应的对象(不检查原型链)中是否存在 name 属性。
- 第三题是先计算 obj2.list 属性的值，然后赋值给 obj2.list 属性下标为 0（属性名为“0”）的属性。

### 私有变量、函数

在函数内部定义的变量和函数如果不对外提供接口，那么外部将无法访问到，也就是变为私有变量和私有函数。

```js
function Obj() {
  var a = 0 //私有变量
  var fn = function() {
    //私有函数
  }
}

var o = new Obj()
console.log(o.a) //undefined
console.log(o.fn) //undefined
```

### 静态变量、函数

当定义一个函数后通过 “.”为其添加的属性和函数，通过对象本身仍然可以访问得到，但是其实例却访问不到，这样的变量和函数分别被称为静态变量和静态函数。

```js
function Obj() {}
  Obj.a = 0 //静态变量
  Obj.fn = function() {
    //静态函数
}

console.log(Obj.a) //0
console.log(typeof Obj.fn) //function

var o = new Obj()
console.log(o.a) //undefined
console.log(typeof o.fn) //undefined
```

### 实例变量、函数
在面向对象编程中除了一些库函数我们还是希望在对象定义的时候同时定义一些属性和方法，实例化后可以访问，JavaScript也能做到这样。
```js
function Obj(){
    this.a=[]; //实例变量
    this.fn=function(){ //实例方法    
    }
}
 
console.log(typeof Obj.a); //undefined
console.log(typeof Obj.fn); //undefined
 
var o=new Obj();
console.log(typeof o.a); //object
console.log(typeof o.fn); //function
```

### 一道综合面试题

题目如下

```js
function Foo() {
  getName = function() {
    alert(1)
  }
  return this
}
Foo.getName = function() {
  alert(2)
}
Foo.prototype.getName = function() {
  alert(3)
}
var getName = function() {
  alert(4)
}
function getName() {
  alert(5)
}

//请写出以下输出结果：
Foo.getName()
getName()
Foo().getName()
getName()
new Foo.getName()
new Foo().getName()
new new Foo().getName()
```

解读：首先定义了一个叫 Foo 的函数，之后为 Foo 创建了一个叫 getName 的静态属性存储了一个匿名函数，之后为 Foo 的原型对象新创建了一个叫 getName 的匿名函数。之后又通过函数变量表达式创建了一个 getName 的函数，最后再声明一个叫 getName 函数。

先来剧透一下答案，再来看看具体分析

```js
//答案：
Foo.getName() //2
getName() //4
Foo().getName() //1
getName() //1
new Foo.getName() //2
new Foo().getName() //3
new new Foo().getName() //3
```

1. 第一问:Foo.getName 自然是访问 Foo 函数上存储的静态属性，自然是 2
2. 第二问，直接调用 getName 函数。既然是直接调用那么就是访问当前上文作用域内的叫 getName 的函数，所以跟 1 2 3 都没什么关系。但是此处有两个坑，一是变量声明提升，二是函数表达式。  
   关于函数变量提示，此处省略一万字。。。。题中代码最终执行时的是

```js
function Foo() {
  getName = function() {
    alert(1)
  }
  return this
}
var getName //只提升变量声明
function getName() {
  alert(5)
} //提升函数声明，覆盖var的声明

Foo.getName = function() {
  alert(2)
}
Foo.prototype.getName = function() {
  alert(3)
}
getName = function() {
  alert(4)
} //最终的赋值再次覆盖function getName声明

getName() //最终输出4
```

3. 第三问的 Foo().getName(); 先执行了 Foo 函数，然后调用 Foo 函数的返回值对象的 getName 属性函数。这里 Foo 函数的返回值是 this，this 指向 window 对象。所以第三问相当于执行 window.getName()。 然而这里 Foo 函数将此变量的值赋值为`function(){alert(1)}`。
4. 第四问直接调用 getName 函数，相当于 window.getName()，答案和前面一样。
5. 后面三问都是考察 js 的运算符优先级问题。
