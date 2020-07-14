javascript语言的执行环境是**单线程**（single thread），就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。

这种模式的好处是实现起来比较简单，执行环境相对单纯；但是只要耗时比较多，假如有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。为了解决这个问题，Javascript语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）。  


* **同步模式**: 就是一个任务先执行，后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的；
* **异步模式**: 每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。  

Javascript处理异步的方法有以下几种：

### 回调函数
回调是一个函数被作为一个参数传递到另一个函数里，在那个函数执行完后再执行。回调函数是异步编程最基本的方法，其优点是简单、容易理解和部署；缺点是容易产生回调地狱。
```js
ajax('XXX1', () => {
  // callback 函数体
  ajax('XXX2', () => {
    // callback 函数体
    ajax('XXX3', () => {
      // callback 函数体
    })
  })
})
```

这就是所谓的回调地狱，回调地狱带来的负面作用有以下几点：

- 代码臃肿，可读性差，可维护性差。
- 代码复用性差。
- 容易滋生 bug。
- 只能在回调里处理异常。

### 事件监听
这种方式，异步任务的执行不取决于代码的顺序，而取决于某个事件是否发生。  

1. 普通方式
```js
f1.on('done', f2);
```
上面这行代码的意思是，当f1发生done事件，就执行f2。

2. onclick方法
```js
element.onclick=function(){
   //处理函数
}

element.onclick=handler1;
element.onclick=handler2;
element.onclick=handler3;
// 只有handler3会被添加执行
```
* 优点：写法兼容到主流浏览器;
* 缺点：当同一个element元素绑定多个事件时，只有最后一个事件会被添加

3. addEvenListener
```js
elment.addEvenListener("click",handler1,false);
elment.addEvenListener("click",handler2,false);
elment.addEvenListener("click",handler3,false);
```
该方法的第三个参数是一个布尔值：当为false时表示由里向外，true表示由外向里。

### 发布/订阅模式
我们假定，存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做"发布/订阅模式"（publish-subscribe pattern）  

首先，f2向信号中心jQuery订阅done信号。
```js
jQuery.subscribe('done', f2);
```

然后，f1进行如下改写：
```js
function f1() {
  setTimeout(function () {
    jQuery.publish('done');
  }, 1000);
}
```
f1执行完成后，向信号中心jQuery发布done信号，从而引发f2的执行。f2完成执行后，可以取消订阅（unsubscribe）
```js
jQuery.unsubscribe('done', f2);
```
这种方式的优点：可以通过查看“消息中心”，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

### promise
以上都是ES6之前的异步处理方式。ES6之后出现了promise。它是异步编程的一种解决方案，比传统的解决方案(回调函数)——更合理和更强大。  

Promise 对象有以下两个特点。

1. 对象的状态不受外界影响。Promise 对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果

#### 基本用法

1. ES6 规定，Promise 对象是一个构造函数，用来生成 Promise 实例。

```js
const promise = new Promise((resolve, reject) => {
  if (/* 异步操作成功 */){
    resolve(success)
  } else {
    reject(error)
  }
})
```
Promise接收一个函数作为参数，函数里有resolve和reject两个参数:  
* ``resolve``方法的作用是将``Promise``的``pending``状态变为``fullfilled``，在异步操作成功之后调用，可以将异步返回的结果作为参数传递出去。
* ``reject``方法的作用是将``Promise``的``pending``状态变为``rejected``，在异步操作失败之后调用，可以将异步返回的结果作为参数传递出去。
* 他们之间只能有一个被执行，不会同时被执行，因为Promise只能保持一种状态。


2. Promise 实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。

```js
promise.then((success) => {
  // 对应于上面的resolve(success)方法
}, (error) => {
  // 对应于上面的reject(error)方法
}


// 还可以写成这样 (推荐使用这种写法)
promise.then((success) => {
  // 对应于上面的resolve(success)方法
}).catch((error) => {
  // 对应于上面的reject(error)方法
})
```
``then(onfulfilled,onrejected)``方法中有两个参数，两个参数都是函数：
* 第一个参数执行的是``resolve()``方法(即异步成功后的回调方法)
* 第二参数执行的是``reject()``方法(即异步失败后的回调方法)(第二个参数可选)。
* 它返回的是一个新的Promise对象。  

3. promise构造函数是同步执行的，then方法是异步执行的
```js
const promise = new Promise((resolve, reject) => {
  console.log(1)
  resolve()
  console.log(2)
})

promise.then(() => {
  console.log(3)
})

console.log(4)
// 1  2  4   3
```

#### Promise.finally()
`Promise.finally()`用于指定不管 Promise 对象最后状态如何，都会执行的操作。
```js
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```

#### Promise.all()
`Promise.all()`用于处理多个异步处理，比如说一个页面上需要等多个 ajax 的数据回来才执行相关逻辑。
```js
const p = Promise.all([p1, p2, p3]);
```
p的状态由p1、p2、p3决定，分成两种情况。
* 只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。
* 只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。  

#### Promse.race()
`Promse.race()`就是赛跑的意思，Promise.race([p1, p2, p3])里面哪个结果获得的快，就返回那个结果，不管结果本身是成功状态还是失败状态。
```js
const p = Promise.race([p1, p2, p3])
```
上面代码中，只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。

### async/await
async/await是JavaScript为了解决异步问题而提出的一种解决方案，许多人将其称为异步的终极解决方案。**async 函数，就是 Generator 函数的语法糖。**

相较于 Generator，Async 函数的改进在于下面四点：

* 内置执行器。Generator 函数的执行必须依靠执行器，而 Aysnc 函数自带执行器，调用方式跟普通函数的调用一样。
* 更好的语义。async 和 await 相较于 * 和 yield 更加语义化。
* 更广的适用性。co 模块约定，yield 命令后面只能是 Thunk 函数或 Promise对象。而 async 函数的 await 命令后面可以是 Promise 或者原始类型（Number，string，boolean，但这时等同于同步）。
* 返回值是 Promise。async 函数返回值是 Promise 对象，比 Generator 函数返回的 Iterator 对象方便，可以直接使用 then() 方法进行调用。  

#### async/await使用规则
1. 凡是在前面添加了async的函数在执行后都会自动返回一个Promise对象
```js
async function test() {
    
}

let result = test()
console.log(result)  //即便代码里test函数什么都没返回，我们依然打出了Promise对象
```

2. await必须在async函数里使用，不能单独使用
```js
function test() {
  let result = await Promise.resolve('success')
  console.log(result)
}

test()   //执行以后会报错
```

#### await 在等什么
* 如果await等到的不是一个promise对象，那跟着的表达式的运算结果就是它等到的东西；
* 如果是一个promise对象，await会阻塞后面的代码，等promise对象resolve，得到resolve的值作为await表达式的运算结果
* 虽然await阻塞了，但await在async中，async不会阻塞，它内部所有的阻塞都被封装在一个promise对象中异步执行