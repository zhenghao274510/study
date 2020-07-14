# JS 执行机制

js 是一门**单线程语言**。所谓单线程，是指在 JavaScript 引擎中负责解释和执行 JavaScript 代码的线程唯一，同一时间上只能执行一件任务。     

js 引擎有一个主线程（main thread）用来解释和执行 js 程序，还存在其他的线程。例如：处理 ajax 请求的线程、处理 DOM 事件的线程、定时器线程、读写文件的线程(例如在 node.js 中)等等。这些线程可能存在于 js 引擎之内，也可能存在于 js 引擎之外，在此我们不做区分。不妨叫它们工作线程。  

为什么是 js 是单线程的呢？这是因为 JavaScript 可以修改 DOM 结构，如果 JavaScript 引擎线程不是单线程的，那么可以同时执行多段 JavaScript，如果这多段 JavaScript 都修改 DOM，那么就会出现 DOM 冲突。为了避免 DOM 渲染的冲突，可以采用单线程或者死锁，JavaScript 采用了单线程方案。  

但单线程有一个问题：如果任务队列里有一个任务耗时很长，导致这个任务后面的任务一直排队等待，就会发生页面卡死，严重影响用户体验。


## JS 执行上下文

当代码运行时，会产生一个对应的执行环境，在这个环境中，所有变量会被事先提出来（变量提升），有的直接赋值，有的为默认值 undefined，代码从上往下开始执行，就叫做执行上下文。

例子 1:变量提升

```js
foo // undefined
var foo = function() {
  console.log('foo1')
}

foo() // foo1，foo赋值

var foo = function() {
  console.log('foo2')
}

foo() // foo2，foo重新赋值
```

例子 2：函数提升

```js
foo() // foo2
function foo() {
  console.log('foo1')
}

foo() // foo2

function foo() {
  console.log('foo2')
}

foo() // foo2
```

例子 3：声明优先级，函数 > 变量

```js
foo() // foo2
var foo = function() {
  console.log('foo1')
}

foo() // foo1，foo重新赋值

function foo() {
  console.log('foo2')
}

foo() // foo1
```

#### 执行上下文类型。

- 全局执行上下文  

  这是默认或者说基础的上下文，任何不在函数内部的代码都在全局上下文中。它会执行两件事：创建一个全局的 window 对象（浏览器的情况下），并且设置 this 的值等于这个全局对象。一个程序中只会有一个全局执行上下文。
- 函数执行上下文  

  每当一个函数被调用时, 都会为该函数创建一个新的上下文。每个函数都有它自己的执行上下文，不过是在函数被调用时创建的。函数上下文可以有任意多个。每当一个新的执行上下文被创建，它会按定义的顺序（将在后文讨论）执行一系列步骤。
- Eval 函数执行上下文  

  执行在 eval 函数内部的代码也会有它属于自己的执行上下文，但由于 JavaScript 开发者并不经常使用 eval，所以在这里我不会讨论它。

#### 执行上下文特点

1. 单线程，在主进程上运行
2. 同步执行，从上往下按顺序执行
3. 全局上下文只有一个，浏览器关闭时会被弹出栈
4. 函数的执行上下文没有数目限制
5. 函数每被调用一次，都会产生一个新的执行上下文环境

## 执行栈
JavaScript 将任务的执行模式分为两种：同步和异步。同步任务都在主线程（这里的主线程就是 JavaScript 引擎线程）上执行，会形成一个 调用栈 ，又称**执行栈**。  

当JavaScript代码被运行的时候，会创建一个全局上下文，并push到当前执行栈。之后当发生函数调用的时候，引擎会为函数创建一个函数执行上下文并push到栈顶。引擎会先执行调用栈顶部的函数，当函数执行完成后，当前函数的执行上下文会被移除当前执行栈。并移动到下一个上下文。  

除了主线程外，还有一个任务队列（也称消息队列），用于管理异步任务的事件回调 ，在调用栈的任务执行完毕之后，系统会检查任务队列，看是否有可以执行的异步任务。

其实这是一个压栈出栈的过程——执行栈。

<img src="../img/js1.png">

```js
var // 1.进入全局上下文环境
  a = 10,
  fn,
  bar = function(x) {
    var b = 20
    fn(x + b) // 3.进入fn上下文环境
  }

fn = function(y) {
  var c = 20
  console.log(y + c)
}

bar(5) // 2.进入bar上下文环境
```

#### 执行上下文生命周期

<img src="../img/js2.png">

* 创建阶段
    - 生成变量对
    - 建立作用域
    - 确定 this 指向

* 执行阶段
    - 变量赋值
    - 函数引用
    - 执行其他代码

* 销毁阶段
    - 执行完毕出栈，等待回收被销毁


#### 栈溢出
在我们执行 JavaScript 代码的时候，有时会出现栈溢出的情况：
```js
Uncaught RangeError: Maximum call stack size exceeded
```
这是一个典型的栈溢出。调用栈是用来管理执行上下文的一种数据结构，它是有大小的，当入栈的上下文过多的时候，它就会报栈溢出。比如
```js
function add() {
  return 1 + add()
}

add()
```
add 函数不断的递归，不断的入栈，调用栈的容量有限，它就溢出了，所以，我们日常的开发中，一定要注意此类代码的出现。

## javascript 事件循环

- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入 `Event Table` 并注册函数。
- 当指定的事情完成时，`Event Table` 会将这个函数移入 `Event Queue`。
- 主线程内的任务执行完毕为空，会去 `Event Queue` 读取对应的函数，进入主线程执行。
- 上述过程会不断重复，也就是常说的 `Event Loop`(事件循环)。

<img src="../img/js3.png">

同步任务和异步任务，我们对任务有更精细的定义：

#### macro-task(宏任务)：

每次执行栈执行的代码就是一个宏任务(包括每次从事件队列中获取一个事件回调并放到执行栈中执行)。浏览器为了能够使得 js 内部`(macro)task`与 DOM 任务能够有序执行，会在一个`(macro)task`执行结束后，在下一个`(macro)task`执行开始前，对页面进行重新渲染。宏任务主要包含：

- script(整体代码)
- setTimeout / setInterval
- setImmediate(Node.js 环境)
- I/O
- UI render
- postMessage
- MessageChannel

#### micro-task(微任务)：

可以理解是在当前 task 执行结束后立即执行的任务。也就是说，在当前 task 任务后，下一个 task 之前，在渲染之前。所以它的响应速度相比 setTimeout（setTimeout 是 task）会更快，因为无需等渲染。也就是说，在某一个 macrotask 执行完后，就会将在它执行期间产生的所有 microtask 都执行完毕（在渲染前）。microtask 主要包含：

- process.nextTick(Node.js 环境)
- Promise
- Async/Await
- MutationObserver(html5 新特性)

<img src="../img/eventLoop.png" width="80%">

总的结论就是，执行宏任务，然后执行该宏任务产生的微任务，若微任务在执行过程中产生了新的微任务，则继续执行微任务，微任务执行完毕后，再回到宏任务中进行下一轮循环。

#### 举个例子

我们来分析一段较复杂的代码，看看你是否真的掌握了 js 的执行机制：

```js
console.log('1')

setTimeout(function() {
  console.log('2')
  process.nextTick(function() {
    console.log('3')
  })
  new Promise(function(resolve) {
    console.log('4')
    resolve()
  }).then(function() {
    console.log('5')
  })
})
process.nextTick(function() {
  console.log('6')
})
new Promise(function(resolve) {
  console.log('7')
  resolve()
}).then(function() {
  console.log('8')
})

setTimeout(function() {
  console.log('9')
  process.nextTick(function() {
    console.log('10')
  })
  new Promise(function(resolve) {
    console.log('11')
    resolve()
  }).then(function() {
    console.log('12')
  })
})

// 1,7,8,2,4,5,6,3,9,11,12,10
```

再来一段

```js
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2')
}
console.log('script start')
setTimeout(function() {
  console.log('setTimeout')
}, 0)
async1()
new Promise(function(resolve) {
  console.log('promise1')
  resolve()
}).then(function() {
  console.log('promise2')
})
console.log('script end')

// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```

### node 的事件循环

- timers 定时器：本阶段执行已经安排的 setTimeout() 和 setInterval() 的回调函数。
- pending callbacks 待定回调：执行延迟到下一个循环迭代的 I/O 回调。
- idle, prepare：仅系统内部使用。
- poll 轮询：检索新的 I/O 事件;执行与 I/O 相关的回调（几乎所有情况下，除了关闭的回调函数，它们由计时器和 setImmediate() 排定的之外），其余情况 node 将在此处阻塞。
- check 检测：setImmediate() 回调函数在这里执行。
- close callbacks 关闭的回调函数：一些准备关闭的回调函数，如：socket.on('close', ...)。

#### 微任务和宏任务在 Node 的执行顺序

Node 10 以前：

- 执行完一个阶段的所有任务
- 执行完 nextTick 队列里面的内容
- 然后执行完微任务队列的内容

Node 11 以后：

- 和浏览器的行为统一了，都是每执行一个宏任务就执行完微任务队列。

## node 环境和浏览器的区别

#### 1、全局环境下 this 的指向

- node 中 this 指向 global
- 浏览器中 this 指向 window
- 这就是为什么 underscore 中一上来就定义了一 root；

浏览器中的 window 下封装了不少的 API 比如 alert 、document、location、history 等等还有很多, 我们就不能在 node 环境中 xxx();或 window.xxx();了。因为这些 API 是浏览器级别的封装，存 javascript 中是没有的。当然 node 中也提供了不少 node 特有的 API。

#### 2、js 引擎

- 在浏览器中不同的浏览器厂商提供了不同的浏览器内核，浏览器依赖这些内核解释折我们编写的 js。但是考虑到不同内核的少量差异，我们需要对应兼容性好在有一些优秀的库帮助我们处理这个问题。比如 jquery、underscore 等等。

- nodejs 是基于 Chrome's JavaScript runtime，也就是说，实际上它是对 GoogleV8 引擎（应用于 Google Chrome 浏览器)进行了封装。V8 引 擎执行 Javascript 的速度非常快，性能非常好。

#### 3、DOM 操作

- 浏览器中的 js 大多数情况下是在直接或间接（一些虚拟 DOM 的库和框架）的操作 DOM。因为浏览器中的代码主要是在表现层工作。
- node 是一门服务端技术。没有一个前台页面，所以我们不会再 node 中操作 DOM。

#### 4、I/O 读写

与浏览器不同，我们需要像其他服务端技术一样读写文件，nodejs 提供了比较方便的组件。而浏览器（确保兼容性的）想在页面中直接打开一个本地的图片就麻烦了好多（别和我说这还不简单，相对路径。。。。。。试试就知道了要么找个库要么二进制流，要么上传上去有了网络地址在显示。不然人家为什么要搞一个 js 库呢），而这一切 node 都用一个组件搞定了。

#### 5、模块加载

- javascript 有个特点，就是原生没提供包引用的 API 一次性把要加载的东西全执行一遍，这里就要看各位闭包的功力了。所用东西都在一起，没有分而治之，搞的特别没有逻辑性和复用性。如果页面简单或网站当然我们可以通过一些 AMD、CMD 的 js 库（比如 requireJS 和 seaJS）搞定事实上很多大型网站都是这么干的。

- nodeJS 中提供了 CMD 的模块加载的 API，如果你用过 seaJS，那么应该上手很快。node 还提供了 npm 这种包管理工具，能更有效方便的管理我们饮用的库


### 参考文献 

[浏览器与Node的事件循环(Event Loop)有何区别?](https://juejin.im/post/5c337ae06fb9a049bc4cd218)  