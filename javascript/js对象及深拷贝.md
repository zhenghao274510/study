# JS 对象创建与拷贝

### 五种创建对象的方法

1、对象字面量的方式

```javascript
person = { firstname: 'Mark', lastname: 'Yun', age: 25, eyecolor: 'black' }
```

2、用 function 来模拟无参的构造函数

```javascript
function Person() {}
var person = new Person() //定义一个function，如果使用new"实例化",该function可以看作是一个Class
person.name = 'Mark'
person.age = '25'
person.work = function() {
  alert(person.name + ' hello...')
}
person.work()
```

3、用 function 来模拟参构造函数来实现（用 this 关键字定义构造的上下文属性）

```javascript
function Pet(name, age, hobby) {
  this.name = name //this作用域：当前对象
  this.age = age
  this.hobby = hobby
  this.eat = function() {
    alert('我叫' + this.name + ',我喜欢' + this.hobby + ',是个程序员')
  }
}
var maidou = new Pet('麦兜', 25, 'coding') //实例化、创建对象
maidou.eat() //调用eat方法
```

4、用工厂方式来创建（内置对象）

```javascript
var wcDog = new Object()
wcDog.name = '旺财'
wcDog.age = 3
wcDog.work = function() {
  alert('我是' + wcDog.name + ',汪汪汪......')
}
wcDog.work()
```

5、用原型方式来创建

```javascript
function Dog() {}
Dog.prototype.name = '旺财'
Dog.prototype.eat = function() {
  alert(this.name + '是个吃货')
}
var wangcai = new Dog()
```

合并两个对象

```javascript
let object1 = { a: 1, b: 2, c: 3 }
let object2 = { b: 30, c: 40, d: 50 }
let merged = { ...object1, ...object2 } //spread and re-add into merged
console.log(merged) // {a:1, b:30, c:40, d:50}
```

### 深拷贝和浅拷贝

js 数据分为基本数据类型(String, Number, Boolean, Null, Undefined，Symbol)和引用数据类型。

- 基本数据类型：直接存储在栈(stack)中的数据
- 引用数据类型点：存储的是该对象在栈中引用，真实的数据存放在堆内存里

深拷贝和浅拷贝的主要区别就是在内存中的存储类型不同。堆和栈是内存中划分出来用来存储的区域。

- 栈（stack）为自动分配的内存空间，它由系统自动释放。
- 堆（heap）则是动态分配的内存，大小不定也不会自动释放。

#### 数据的比较
基本类型的比较是值的比较，只要它们的值相等就认为他们是相等的，例如：

```js
var a = 10
var b = 10
console.log(a === b) //true
```

而引用类型的比较是引用的比较，所以每次我们对 js 中的引用类型进行操作的时候，都是操作其对象的引用，所以比较两个引用类型，是看其的引用是否指向同一个对象。例如：

```js
var a = [1, 2, 3]
var b = [1, 2, 3]
console.log(a === b) // false
```

#### 传值与传址

基础数据类型的赋值都是属于传值，两个变量是两个独立相互不影响的变量。举个例子；

```js
var a = 10
var b = a

a++
console.log(a) // 11
console.log(b) // 10
```

但是引用类型的赋值是传址。只是改变指针的指向，例如，也就是说引用类型的赋值是对象保存在栈中的地址的赋值，这样的话两个变量就指向同一个对象，因此两者之间操作互相有影响。例如：

```js
var a = {} // a保存了一个空对象的实例
var b = a // a和b都指向了这个空对象

a.name = 'jozo'
console.log(a.name) // 'jozo'
console.log(b.name) // 'jozo'

b.age = 22
console.log(b.age) // 22
console.log(a.age) // 22

console.log(a == b) // true
```

#### 深拷贝

深拷贝复制变量值，对于非基本类型的变量，则递归至基本类型变量后，再复制。 深拷贝后的对象与原来的对象是完全隔离的，互不影响，对一个对象的修改并不会影响另一个对象。

#### 浅拷贝

浅拷贝是会将对象的每个属性进行依次复制，但是当对象的属性值是引用类型时，实质复制的是其引用，当引用指向的值改变时也会跟着变化。

#### 区别

浅拷贝和深拷贝都只针对于引用数据类型，浅拷贝只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存；但深拷贝会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象。

区别：浅拷贝只复制对象的第一层属性、深拷贝可以对对象的属性进行递归复制；

```javascript
let obj = {
  name: 'Tom',
  age: 18,
  hobbies: ['reading', 'photography']
}
let obj1 = obj
let obj2 = Object.assign({}, obj)
let obj3 = { ...obj }
let obj4 = JSON.parse(JSON.stringify(obj))

obj.name = 'Jack'
obj.hobbies.push('coding')
console.log(obj) //{ name: 'Jack', age: 18,hobbies: [ 'reading', 'photography', 'coding' ] }
console.log(obj1) //{ name: 'Jack', age: 18,hobbies: [ 'reading', 'photography', 'coding' ] }
console.log(obj2) //{ name: 'Tom', age: 18,hobbies: [ 'reading', 'photography', 'coding' ] }
console.log(obj3) //{ name: 'Tom', age: 18,hobbies: [ 'reading', 'photography', 'coding' ] }
console.log(obj4) //{ name: 'Tom', age: 18,hobbies: [ 'reading', 'photography' ] }
```

从以上例子可以看出，当数据为引用数据类型时

1. 直接赋值属于浅拷贝
2. Object.assign，当数据第一层为基本数据类型时，新的对象和原对象互不影响，这属于深拷贝，但是如果第一层的属性值是复杂数据类型，那么新对象和原对象的属性值其指向的是同一块内存地址，这是浅拷贝。
3. 扩展运算符和Object.assign原理一样。
3. JSON.parse(JSON.stringify(obj))可实现深拷贝

### 深拷贝的实现

1. JSON.parse(JSON.stringify(obj)) 但存在一些缺陷

- 对象的属性值是函数时，无法拷贝。
- 原型链上的属性无法拷贝
- 不能正确的处理 Date，RegExp 类型的数据
- 会忽略 undefined

2. 实现一个 deepClone 函数

- 如果是基本数据类型，直接返回
- 如果是 RegExp 或者 Date 类型，返回对应类型
- 如果是复杂数据类型，递归。
- 考虑循环引用的问题

```javascript
function deepClone(obj, hash = new WeakMap()) {
  //递归拷贝
  if (obj instanceof RegExp) return new RegExp(obj)
  if (obj instanceof Date) return new Date(obj)
  if (obj === null || typeof obj !== 'object') {
    //如果不是复杂数据类型，直接返回
    return obj
  }
  if (hash.has(obj)) {
    return hash.get(obj)
  }
  /**
   * 如果obj是数组，那么 obj.constructor 是 [Function: Array]
   * 如果obj是对象，那么 obj.constructor 是 [Function: Object]
   */
  let t = new obj.constructor()
  hash.set(obj, t)
  for (let key in obj) {
    //递归
    if (obj.hasOwnProperty(key)) {
      //是否是自身的属性
      t[key] = deepClone(obj[key], hash)
    }
  }
  return t
}
```
