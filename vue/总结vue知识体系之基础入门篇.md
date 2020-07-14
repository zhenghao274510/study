### Vue 的优缺点是什么

优点：

1. 低耦合。视图（View）可以独立于 Model 变化和修改，一个 ViewModel 可以绑定到不同的 View 上，当 View 变化的时候 Model 可以不变，当 Model 变化的时候 View 也可以不变。
2. 可重用性。你可以把一些视图逻辑放在一个 ViewModel 里面，让很多 view 重用这段视图逻辑。
3. 独立开发。开发人员可以专注于业务逻辑和数据的开发（ViewModel），设计人员可以专注于页面设计，使用 Expression Blend 可以很容易设计界面并生成 xml 代码。
4. 可测试。界面素来是比较难于测试的，而现在测试可以针对 ViewModel 来写。
5. vue 是单页面应用，使页面局部刷新，不用每次跳转页面都要请求所有数据和 dom，这样大大加快了访问速度和提升用户体验。而且他的第三方 ui 库很多节省开发时间

缺点：不利于 SEO，社区维护力度不强，相比还不够成熟

### vue 常用指令

- ``v-html / v-text``：把值中的标签渲染出来
- ``v-model``： 放在表单元素上的，实现双向数据绑定
- ``v-bind``（缩写 :）：用于绑定行内属性
- ``v-if / v-show`` 是否能显示，true 能显示，false 不能显示
- ``v-cloak``：需要配合 css 使用：解决小胡子显示问题
- ``v-once`` 对应的标签只渲染一次
- ``v-for`` ：循环显示元素
- ``v-on`` 事件绑定

### 事件修饰符

``Vue.js`` 为 ``v-on`` 提供了事件修饰符，修饰符是由点开头的指令后缀来表示的。

- ``stop``：阻止事件继续传播
- ``prevent``：阻止事件默认行为
- ``capture``：添加事件监听器时使用事件捕获模式
- ``self``：当前元素触发时才触发事件处理函数
- ``once``：事件只触发一次
- ``passive``：告诉浏览器你不想阻止事件的默认行为，不能和.prevent 一起使用。

```html
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="toDo"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="toSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="toDo"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即内部元素触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="toDo">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<div v-on:click.self="toDo">...</div>

<!-- 点击事件将只会触发一次 -->
<a v-on:click.once="toDo"></a>

<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
<div v-on:scroll.passive="onScroll">...</div>
```

### 表单修饰符

* .lazy 在输入框输入完内容，光标离开时才更新视图
* .trim 过滤首尾空格
* .number 如果先输入数字，那它就会限制你输入的只能是数字;如果先输入字符串，那就相当于没有加.number

### 过滤器 filter

过滤器是对即将显示的数据做进一步的筛选处理，然后进行显示，值得注意的是过滤器并没有改变原来的数据，只是在原数据的基础上产生新的数据。

1. 定义过滤器  
   全局注册

```js
Vue.filter('myFilter', function (value1[,value2,...] ) {
// 代码逻辑
})
```

局部注册

```js
new Vue({
 filters: {
    'myFilter': function (value1[,value2,...] ) {
       // 代码逻辑
      }
    }
　})
```

2. 使用过滤器

```html
<!-- 在双花括号中 -->
<div>{{ message | myFilter }}</div>

<!-- 在 `v-bind` 中 -->
<div v-bind:id="message | myFilter"></div>
```

### 计算属性 computed

依赖其它属性值，并且 ``computed`` 的值有缓存，只有它依赖的属性值发生改变，下一次获取 ``computed`` 的值时才会重新计算 ``computed`` 的值；

```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>

<script>
  var vm = new Vue({
    el: '#example',
    data: {
      message: 'Hello'
    },
    computed: {
      // 计算属性的 getter
      reversedMessage: function() {
        // `this` 指向 vm 实例
        return this.message.split('').reverse().join('')
      }
    }
  })
</script>
```

### 监听属性 watch

观察和响应 Vue 实例上的数据变动。类似于某些数据的监听回调 ，每当监听的数据变化时都会执行回调进行后续操作。它可以有三个参数

- ``handler``：其值是一个回调函数。即监听到变化时应该执行的函数
- ``deep``：其值是 true 或 false；确认是否深入监听。
- ``immediate``：其值是 true 或 false，确认是否以当前的初始值执行 handler 的函数

```js
watch:{
  message:{
    handler:function(val, oldVal){
      console.log(val, oldVal)
    },
    deep: true,
    immediate: true
  }
}
```

### computed 和 watch 的区别

- ``computed``： 是计算属性，依赖其它属性值，并且 computed 的值有缓存，只有它依赖的属性值发生改变，下一次获取 computed 的值时才会重新计算 computed 的值；
- ``watch``： 更多的是「观察」的作用，类似于某些数据的监听回调 ，每当监听的数据变化时都会执行回调进行后续操作。

**运用场景**

- 当我们需要进行数值计算，并且依赖于其它数据时，应该使用 computed，因为可以利用 computed 的缓存特性，避免每次获取值时，都要重新计算；
- 当我们需要在数据变化时执行异步或开销较大的操作时，应该使用 watch，使用  watch  选项允许我们执行异步操作 ( 访问一个 API )，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

### 生命周期函数

- ``beforeCreate``(创建前) vue 实例的挂载元素\$el 和数据对象 data 都是 undefined, 还未初始化
- ``created``(创建后) 完成了 data 数据初始化, el 还未初始化
- ``beforeMount``(载入前) vue 实例的\$el 和 data 都初始化了, 相关的 render 函数首次被调用
- ``mounted``(载入后) 此过程中进行 ajax 交互
- ``beforeUpdate``(更新前)
- ``updated``(更新后)
- ``beforeDestroy``(销毁前)
- ``destroyed``(销毁后)

**Vue 的父组件和子组件生命周期钩子执行顺序是什么?**

1. 渲染过程：父组件挂载完成一定是等子组件都挂载完成后，才算是父组件挂载完，所以父组件的 mounted 在子组件 mouted 之后。

- 父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子 created -> 子 beforeMount -> 子 mounted -> 父 mounted

2. 子组件更新过程：

- 影响到父组件：父 beforeUpdate -> 子 beforeUpdate->子 updated -> 父 updted
- 不影响父组件：子 beforeUpdate -> 子 updated

3. 父组件更新过程：

- 影响到子组件：父 beforeUpdate -> 子 beforeUpdate->子 updated -> 父 updted
- 不影响子组件：父 beforeUpdate -> 父 updated

4. 销毁过程：

- 父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed

### 组件注册

组件``（component）``是 Vue.js 最强大的功能之一。组件可以扩展 HTML 元素，封装可重用的代码。组件的使用过程包括定义和注册的过程。

1. 定义组件

```js
// 方法一 Vue.extend
var MyComponent = Vue.extend({
  template: '<div>A custom component!</div>'
})
// 方法二：新建一个.vue 文件
```

2. 注册组件

```js
// 全局注册
Vue.component('my-component', MyComponent)

// 局部注册
new Vue({
  el: '#app',
  components: {
    'my-component': MyComponent
  }
})
```

3. 使用组件

```html
<div id="example">
  <my-component></my-component>
</div>
```

### 组件传值

#### 1. props 父组件给子组件传值

props 值可以是一个数组或对象;

```js
// 数组:不建议使用
props:[]

// 对象
props:{
 inpVal:{
  type:Number, //传入值限定类型
  // type 值可为String,Number,Boolean,Array,Object,Date,Function,Symbol
  // type 还可以是一个自定义的构造函数，并且通过 instanceof 来进行检查确认
  required: true, //是否必传
  default:200,  //默认值,对象或数组默认值必须从一个工厂函数获取如 default:()=>[]
  validator:(value) {
    // 这个值必须匹配下列字符串中的一个
    return ['success', 'warning', 'danger'].indexOf(value) !== -1
  }
 }
}
```

#### 2. \$emit 子组件给父组件传值

触发子组件触发父组件给自己绑定的事件,其实就是子传父的方法

```js
// 父组件
<v-Header @title="title">
// 子组件
this.$emit('title',{title:'这是title'})
```

#### 3. vuex 数据状态管理

- ``state``:定义存贮数据的仓库 ,可通过 this.\$store.state 或 mapState 访问
- ``getter``:获取 store 值,可认为是 store 的计算属性,可通过 this.\$store.getter 或 mapGetters 访问
- ``mutation``:同步改变 store 值,可通过 mapMutations 调用
- ``action``:异步调用函数执行 mutation,进而改变 store 值,可通过 this.\$dispatch 或 mapActions 访问
- ``modules``:模块,如果状态过多,可以拆分成模块,最后在入口通过...解构引入

#### 4. attrs 和 listeners

``attrs`` 获取子传父中未在 props 定义的值

```js
// 父组件
<home title="这是标题" width="80" height="80" imgUrl="imgUrl"/>
// 子组件
mounted() {
  console.log(this.$attrs) //{title: "这是标题", width: "80", height: "80", imgUrl: "imgUrl"}
}

// 相对应的如果子组件定义了 props,打印的值就是剔除定义的属性
props: {
  width: {
    type: String,
    default: ''
  }
},
mounted() {
  console.log(this.$attrs) //{title: "这是标题", height: "80", imgUrl: "imgUrl"}
}
```

``listeners``:场景:子组件需要调用父组件的方法。
解决:父组件的方法可以通过 ``v-on="listeners"`` 传入内部组件——在创建更高层次的组件时非常有用

```js
// 父组件
<home @change="change"/>

// 子组件
mounted() {
  console.log(this.$listeners) //即可拿到 change 事件
}
```

#### 5. provide 和 inject

``provide`` 和 ``inject`` 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中; 并且这对选项需要一起使用; 以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。

```js
//父组件:
provide: { //provide 是一个对象,提供一个属性或方法
  foo: '这是 foo',
  fooMethod:()=>{
    console.log('父组件 fooMethod 被调用')
  }
},

// 子或者孙子组件
inject: ['foo','fooMethod'], //数组或者对象,注入到子组件
mounted() {
  this.fooMethod()
  console.log(this.foo)
}
//在父组件下面所有的子组件都可以利用inject
```

#### 6. \$refs

通常用于父组件调用子组件的方法

```js
// 父组件
<home ref="child"/>

mounted(){
  console.log(this.$refs.child) //即可拿到子组件的实例,就可以直接操作 data 和 methods
}
```

#### 7. EventBus

1. 就是声明一个全局 Vue 实例变量 EventBus , 把所有的通信数据，事件监听都存储到这个变量上;
2. 类似于 Vuex。但这种方式只适用于极小的项目 3.原理就是利用 emit 并实例化一个全局 vue 实现数据共享

```js
// 在 main.js
Vue.prototype.$eventBus = new Vue()

// 传值组件
this.$eventBus.$emit('eventTarget', '这是eventTarget传过来的值')

// 接收组件
this.$eventBus.$on('eventTarget', v => {
  console.log('eventTarget', v) //这是eventTarget传过来的值
})
```

### 路由配置和使用

1. 配置路由信息
```js
let routes = [
  {
    path: '/home',
    component: home
  },
  {
    path: '/list',
    component: list
  }
]

let router = new VueRouter({
  routes: routes
})
let vm = new Vue({
  el: '#app',
  router
})
```

在html使用
```html
<div id="app">
   <router-link to='/home' active-class='current'>首页</router-link>
   <router-link to='/list' tag='div'>列表</router-link>
   <router-view></router-view>
</div>
```
此外，``vue-router``还可以通过一下方式配置动态路由
* ``query``传参（问号传参）
* ``params``传参（路径传参）

### 路由懒加载
Vue 项目中实现路由按需加载（路由懒加载）的 3 中方式：
```js
// 1、Vue异步组件技术：
{
	path: '/home',
	name: 'Home',
	component: resolve => reqire(['path路径'], resolve)
}

// 2、es6提案的import()
const Home = () => import('path路径')

// 3、webpack提供的require.ensure()
{
	path: '/home',
	name: 'Home',
	component: r => require.ensure([],() =>  r(require('path路径')), 'demo')
}
```

### 路由守卫
vue-router 提供的导航守卫主要用来通过跳转或取消的方式守卫导航。有多种机会植入路由导航过程中：全局的, 单个路由独享的, 或者组件级的。  

**全局前置守卫** 
常用于判断登录状态和菜单权限校验
```js
router.beforeEach((to, from, next) => {
  let isLogin = sessionStorage.getItem('isLogin') || ''
  if (!isLogin && to.meta.auth) {
    next('/login')
  } else {
    next()
  }
})
```
* ``to``: Route: 即将要进入的目标 路由对象
* ``from``: Route: 当前导航正要离开的路由
* ``next``: Function: 一定要调用该方法来 resolve 这个钩子。执行效果依赖 next 方法的调用参数。

**组件内的守卫**
* ``beforeRouteEnter``
* ``beforeRouteUpdate`` 
* ``beforeRouteLeave``


### 路由缓存 keepalive
``keep-alive`` 是 Vue 提供的一个抽象组件，用来对组件进行缓存，从而节省性能，由于是一个抽象组件，所以在 v 页面渲染完毕后不会被渲染成一个 DOM 元素。

```html
<keep-alive>
  <router-view></router-view>
</keep-alive>
```

当组件在 ``keep-alive`` 内被切换时组件的 ``activated``、``deactivated`` 这两个生命周期钩子函数会被执行

#### 1. 使用参数include/exclude

- include: 字符串或正则表达式。只有匹配的组件会被缓存。
- exclude: 字符串或正则表达式。任何匹配的组件都不会被缓存。

```html
<keep-alive include="a,b">
  <router-view></router-view>
</keep-alive>
<keep-alive exclude="c">
  <router-view></router-view>
</keep-alive>
```

``include`` 属性表示只有 name 属性为 a，b 的组件会被缓存，（注意是组件的名字，不是路由的名字）其它组件不会被缓存。
``exclude`` 属性表示除了 name 属性为 c 的组件不会被缓存，其它组件都会被缓存。

#### 2. 使用\$route.meta 的 keepAlive 属性

需要在 router 中设置 router 的元信息 meta

```js
export default new Router({
  routes: [
    {
      path: '/',
      name: 'Hello',
      component: Hello,
      meta: {
        keepAlive: false // 不需要缓存
      }
    },
    {
      path: '/page1',
      name: 'Page1',
      component: Page1,
      meta: {
        keepAlive: true // 需要被缓存
      }
    }
  ]
})
```

在 app.vue 进行区别缓存和不用缓存的页面

```html
<div id="app">
  <router-view v-if="!$route.meta.keepAlive"></router-view>
  <keep-alive>
    <router-view v-if="$route.meta.keepAlive"></router-view>
  </keep-alive>
</div>
```

### hash 和 history模式

- hash 模式：在浏览器中符号“#”，#以及#后面的字符称之为 hash，用 `window.location.hash` 读取。特点：hash 虽然在 URL 中，但不被包括在 HTTP 请求中；用来指导浏览器动作，对服务端安全无用，hash 不会重加载页面。

- history 模式：history 采用 HTML5 的新特性；且提供了两个新方法： `pushState()， replaceState()`可以对浏览器历史记录栈进行修改，以及`popState`事件的监听到状态变更。

- hash 模式中`（ http://localhost:8080#home）`，即使不需要配置，静态服务器始终会去寻找`index.html`并返回给我们，然后`vue-router`会获取 #后面的字符作为参数，对前端页面进行变换。

- history 模式中，我们所想要的情况就是：输入`http://localhost:8080/home`，但最终返回的也是`index.html`，然后`vue-router`会获取 home 作为参数，对前端页面进行变换。那么在`nginx`中，谁能做到这件事呢？答案就是`try_files`。