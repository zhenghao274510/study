# JS 继承

### 1、属性拷贝

如果继承过来的成员是引用类型的话, 那么这个引用类型的成员在父对象和子对象之间是共享的, 也就是说修改了之后, 父子对象都会受到影响.

```javascript
// 创建父对象
var superObj = {
  age: 25,
  friends: { age: 33 },
  showName: function() {
    console.log(this.name)
  }
}

// 创建需要继承的子对象
var subObj = {}

// 开始拷贝属性(使用for...in...循环)
for (var i in superObj) {
  subObj[i] = superObj[i]
}

subObj.age = 20
subObj.friends.age = 44

console.log(subObj) //  {age:20,friends: { age: 44 }}
console.log(superObj) //  {age:25,friends: { age: 44 }}
```

### 2、原型式继承

- 父构造函数的原型对象和子构造函数的原型对象上的成员有共享问题
- 只能继承父构造函数的原型对象上的成员, 不能继承父构造函数的实例对象的成员

```javascript
// 创建父构造函数
function SuperClass(name) {
  this.name = name
  this.showName = function() {
    alert(this.name)
  }
}

// 设置父构造器的原型对象
SuperClass.prototype.showAge = function() {
  console.log(this.age)
}

// 创建子构造函数
function SubClass() {}

// 设置子构造函数的原型对象实现继承
SubClass.prototype = SuperClass.prototype

var child = new SubClass()
```

### 3、原型链继承

不能给父构造函数传递参数，父子构造函数的原型对象之间有共享问题

```javascript
// 创建父构造函数
function SuperClass() {
  this.name = 'liyajie'
  this.age = 25
  this.showName = function() {
    console.log(this.name)
  }
}
// 设置父构造函数的原型
SuperClass.prototype.friends = ['小名', '小强']
SuperClass.prototype.showAge = function() {
  console.log(this.age)
}
// 创建子构造函数
function SubClass() {}
// 实现继承
SubClass.prototype = new SuperClass()
// 修改子构造函数的原型的构造器属性
SubClass.prototype.constructor = SubClass

var child = new SubClass()
console.log(child.name) // liyajie
console.log(child.age) // 25
child.showName() // liyajie
child.showAge() // 25
console.log(child.friends) // ['小名','小强']

// 当我们改变friends的时候, 父构造函数的原型对象的也会变化
child.friends.push('小王八')
console.log(child.friends) //[('小名', '小强', '小王八')]
var father = new SuperClass()
console.log(father.friends) //[('小名', '小强', '小王八')]
```

### 4、借用构造函数

使用 call 和 apply 借用其他构造函数的成员, 可以解决给父构造函数传递参数的问题, 但是获取不到父构造函数原型上的成员.也不存在共享问题

```javascript
// 创建父构造函数
function Person(name) {
  this.name = name
  this.friends = ['小王', '小强']
  this.showName = function() {
    console.log(this.name)
  }
}
Person.prototype.showAge = function() {
  console.log(this.age)
}

// 创建子构造函数
function Student(name) {
  // 使用call借用Person的构造函数
  Person.call(this, name)
}

// 测试是否有了 Person 的成员
var stu = new Student('Li')
stu.showName() // Li
console.log(stu.friends) // ['小王','小强']
stu.showAge()  // stu.showAge is not a function
```

### 5、组合继承 (借用构造函数 + 原型式继承)

- 解决了父构造函数的属性继承到了子构造函数的实例对象上了,
- 并且继承了父构造函数原型对象上的成员
- 解决了给父构造函数传递参数问题
- 存在共享的问题

```javascript
// 创建父构造函数
function Person(name, age) {
  this.name = name
  this.age = age
  this.showName = function() {
    console.log(this.name)
  }
}
// 设置父构造函数的原型对象
Person.prototype.showAge = function() {
  console.log(this.age)
}
// 创建子构造函数
function Student(name) {
  Person.call(this, name)
}
// 设置继承
Student.prototype = Person.prototype
Student.prototype.constructor = Student
```

### 6、借用构造函数 + 深拷贝

这样就将 Person 的原型对象上的成员拷贝到了 Student 的原型上了, 这种方式没有属性共享的问题.

```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
    this.showName = function(){
        console.log(this.name);
    }
}
Person.prototype.friends = ['小王','小强','小王八'];

function Student(name,25){
    // 借用构造函数(Person)
    Person.call(this,name,25);
}
// 使用深拷贝实现继承
deepCopy(Student.prototype,Person.prototype);
Student.prototype.constructor = Student;
```
