# vue.\$nextTick

Vue 的特点之一就是响应式，但是有些时候数据更新了，我们看到页面上的 DOM 并没有立刻更新。如果我们需要在 DOM 更新之后再执行一段代码时，可以借助 nextTick 实现。

我们先来看一个例子

```js
export default {
  data() {
    return {
      msg: 0
    }
  },
  mounted() {
    this.msg = 1
    this.msg = 2
    this.msg = 3
  },
  watch: {
    msg() {
      console.log(this.msg)
    }
  }
}
```

这里的结果是只输出一个 3，而非依次输出 1，2，3。这是为什么呢？  
vue 的官方文档是这样解释的

> Vue 异步执行 DOM 更新。只要观察到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据改变。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作上非常重要。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部尝试对异步队列使用原生的 Promise.then 和 MessageChannel，如果执行环境不支持，会采用 setTimeout(fn, 0)代替。

假如有这样一种情况，`mounted`钩子函数下一个变量 a 的值会被++循环执行 1000 次。 每次++时，都会根据响应式触发`setter->Dep->Watcher->update->run`。 如果这时候没有异步更新视图，那么每次++都会直接操作 DOM 一次，这是非常消耗性能的。 所以 Vue 实现了一个`queue`队列，在下一个 Tick（或者是当前 Tick 的微任务阶段）的时候会统一执行`queue`中`Watcher`的`run`。同时，拥有相同 id 的`Watcher`不会被重复加入到该`queue`中去，所以不会执行 1000 次`Watcher`的`run`。最终的结果是直接把 a 的值从 1 变成 1000，大大提升了性能。

在 vue 中，数据监测都是通过 Object.defineProperty 来重写里面的 set 和 get 方法实现的，vue 更新 DOM 是异步的，每当观察到数据变化时，vue 就开始一个队列，将同一事件循环内所有的数据变化缓存起来，等到下一次 event loop，将会把队列清空，进行 dom 更新。

想要了解 vue.nextTick 的执行机制，我们先来了解一下 javascript 的事件循环。

### js 事件循环

js 的任务队列分为同步任务和异步任务，所有的同步任务都是在主线程里执行的。异步任务可能会在 macrotask 或者 microtask 里面，异步任务进入 `Event Table` 并注册函数。当指定的事情完成时，`Event Table` 会将这个函数移入 `Event Queue`。主线程内的任务执行完毕为空，会去 `Event Queue` 读取对应的函数，进入主线程执行。上述过程会不断重复，也就是常说的 `Event Loop`(事件循环)。

<img src="../img/js3.png">

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

#### 小结

1. 先执行主线程
2. 遇到宏队列（macrotask）放到宏队列（macrotask）
3. 遇到微队列（microtask）放到微队列（microtask）
4. 主线程执行完毕
5. 执行微队列（microtask），微队列（microtask）执行完毕
6. 执行一次宏队列（macrotask）中的一个任务，执行完毕
7. 执行微队列（microtask），执行完毕
8. 依次循环。。。

### Vue.nextTick 源码

vue 是采用双向数据绑定的方法驱动数据更新的，虽然这样能避免直接操作 dom，提高了性能，但有时我们也不可避免需要操作 DOM，这时就该 Vue.nextTick(callback)出场了，它接受一个回调函数，在 DOM 更新完成后，这个回调函数就会被调用。不管是 vue.nextTick 还是 vue.prototype.\$nextTick 都是直接用的 nextTick 这个闭包函数。

```js
export const nextTick = (function () {
  const callbacks = []
  let pending = false
  let timerFunc

  function nextTickHandler () {
    pending = false
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }
 ...
})()
```

使用数组 callbacks 保存回调函数，pending 表示当前状态，使用函数 nextTickHandler 来执行回调队列。在该方法内，先通过 slice(0)保存了回调队列的一个副本，通过设置 callbacks.length = 0 清空回调队列，最后使用循环执行在副本里的所有函数。

```js
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  var p = Promise.resolve()
  var logError = err => {
    console.error(err)
  }
  timerFunc = () => {
    p.then(nextTickHandler).catch(logError)
    if (isIOS) setTimeout(noop)
  }
} else if (typeof MutationObserver !== 'undefined' && (isNative(MutationObserver) || MutationObserver.toString() === '[object MutationObserverConstructor]')) {
  var counter = 1
  var observer = new MutationObserver(nextTickHandler)
  var textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
} else {
  timeFunc = () => {
    setTimeout(nextTickHandle, 0)
  }
}
```

队列控制的最佳选择是 microtask，而 microtask 的最佳选择是 Promise。但如果当前环境不支持 Promise，就检测到浏览器是否支持 MO，是则创建一个文本节点，监听这个文本节点的改动事件，以此来触发 nextTickHandler（也就是 DOM 更新完毕回调）的执行。此外因为兼容性问题，vue 不得不做了 microtask 向 macrotask 的降级方案。

为让这个回调函数延迟执行，vue 优先用 promise 来实现，其次是 html5 的 MutationObserver，然后是 setTimeout。前两者属于 microtask，后一个属于 macrotask。下面来看最后一部分。

```js
return function queueNextTick(cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) cb.call(ctx)
    if (_resolve) _resolve(ctx)
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```

这就是我们真正调用的 nextTick 函数，在一个 event loop 内它会将调用 nextTick 的 cb 回调函数都放入 callbacks 中，pending 用于判断是否有队列正在执行回调，例如有可能在 nextTick 中还有一个 nextTick，此时就应该属于下一个循环了。最后几行代码是 promise 化，可以将 nextTick 按照 promise 方式去书写（暂且用的较少）。

### 应用场景

场景一、点击按钮显示原本以 v-show = false 隐藏起来的输入框，并获取焦点。

```js
<input id="keywords" v-if="showit">

showInput(){
  this.showit = true
  document.getElementById("keywords").focus()
```

以上的写法在第一个 tick 里，因为获取不到输入框，自然也获取不到焦点。如果我们改成以下的写法，在 DOM 更新后就可以获取到输入框焦点了。

```js
showsou(){
  this.showit = true
  this.$nextTick(function () {
    // DOM 更新了
    document.getElementById("keywords").focus()
  })
}
```

场景二、获取元素属，点击获取元素宽度。

```js
<div id="app">
  <p ref="myWidth" v-if="showMe">{{ message }}</p>
  <button @click="getMyWidth">获取p元素宽度</button>
</div>

getMyWidth() {
  this.showMe = true;
  this.message = this.$refs.myWidth.offsetWidth;
  //报错 TypeError: this.$refs.myWidth is undefined
  this.$nextTick(()=>{
      //dom元素更新后执行，此时能拿到p元素的属性
    this.message = this.$refs.myWidth.offsetWidth;
})
}
```
