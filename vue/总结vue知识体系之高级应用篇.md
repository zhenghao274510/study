### Vue.use

我们使用的第三方 Vue.js 插件。如果插件是一个对象，必须提供`install`方法。如果插件是一个函数，它会被作为`install`方法。`install`方法调用时，会将`Vue`作为参数传入。该方法需要在调用`new Vue()`之前被调用。

我们在使用插件或者第三方组件库的时候用到`Vue.use`这个方法，比如

```js
import iView from 'iview'
Vue.use(iView)
```

那么`Vue.use`到底做了些什么事情呢？我们先来看一下源码

```js
import { toArray } from '../util/index'

export function initUse(Vue: GlobalAPI) {
  Vue.use = function(plugin: Function | Object) {
    const installedPlugins = this._installedPlugins || (this._installedPlugins = [])
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
}
```

我们由以上可以看出，`plugin`参数为函数或者对象类型，首先`Vue`会去寻找这个插件在已安装的插件列表里有没有，如果没有，则进行安装插件，如果有则跳出函数，这保证插件只被安装一次。

接着通过`toArray`方法将参数变成数组，再判断`plugin`的`install`属性是否为函数，或者`plugin`本身就是函数，最后执行`plugin.install`或者`plugin`的方法。

#### 举个例子

下面我们来举个实际例子  
1、编写两个插件

```js
const Plugin1 = {
  install(a) {
    console.log(a)
  }
}

function Plugin2(b) {
  console.log(b)
}

export { Plugin1, Plugin2 }
```

2、引入并 use 这两个插件

```js
import Vue from 'vue'
import { Plugin1, Plugin2 } from './plugins'

Vue.use(Plugin1, '参数1')
Vue.use(Plugin2, '参数2')
```

此时我们运行项目代码就可以用到上面两个插件了。

### Vue.mixin

混入 (mixin) 提供了一种非常灵活的方式，来分发 Vue 组件中的可复用功能。一个混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项。

1、定义一个 mixin.js

```js
export default mixin {
 data() {
  return {
   name: 'mixin'
  }
 },
 created() {
  console.log('mixin...', this.name);
 },
 mounted() {},
 methods: {  //日期转换
   formatDate (dateTime, fmt = 'YYYY年MM月DD日 HH:mm:ss') {
     if (!dateTime) {
      return ''
     }
     moment.locale('zh-CN')
     dateTime = moment(dateTime).format(fmt)
     return dateTime
  }
 }
}
```

2、在vue文件中使用mixin
```js
import '@/mixin'; // 引入mixin文件
export default {
 mixins: [mixin],  //用法
 data() {
  return {
   userName: "adimin",
   time: this.formatDate(new Date()) //这个vue文件的数据源data里面的time就是引用混入进来的方法
  }
 }
} 
```

或者在全局中使用在main.js中，所有页面都能使用了
```js
import mixin from './mixin'
Vue.mixin(mixin)  
```

#### 合并选项
当组件和混入对象含有同名选项时，这些选项将以恰当的方式进行“合并”。
* `data`对象在内部会进行递归合并，并在发生冲突时以组件数据优先。
* 同名钩子函数将合并为一个数组，因此都将被调用。混入对象的钩子将在组件自身钩子之前调用。
* 值为对象的选项，例如 `methods`、`components` 和 `directives`，将被合并为同一个对象。两个对象键名冲突时，取组件对象的键值对。


### Vue.extend
``Vue.extend`` 属于 Vue 的全局 API。它使用基础 Vue 构造器，创建一个“子类”。参数是一个包含组件选项的对象。如下：
```js
<div id="app"></div>

var Profile = Vue.extend({
  template: '<p>{{firstName}} {{lastName}}</p>',
  data: function () {
    return {
      firstName: 'Walter',
      lastName: 'White'
    }
  }
})
// 创建 Profile 实例，并挂载到一个元素上。
new Profile().$mount('#app')
```

#### 应用实例
我们常用 `Vue.extend` 封装一些全局插件，比如 `toast`， `diolog` 等。   

下面以封装一个 `toast` 组件为例。  

1、编写组件
* 根据传入的 type 确定弹窗的类型（成功提示，失败提示，警告，加载，纯文字）
* 设置弹窗消失的时间
```html
<template>
  <div>
    <transition name="fade">
      <div class="little-tip" v-show="showTip">
        <img src="/success.png" alt="" width="36" v-if="type=='success'" />
        <img src="/fail.png" alt="" width="36" v-if="type=='fail'" />
        <img src="/warning.png" alt="" width="36" v-if="type=='warning'" />
        <img src="/loading.png" alt="" width="36" v-if="type=='loading'" class="loading" />
        <span>{{msg}}</span>
      </div>
    </transition>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        showTip: true,
        msg: '',
        type: ''
      }
    },
    mounted() {
      setTimeout(() => {
        this.showTip = false
      }, 1500)
    }
  }
</script>
<style lang="less" scoped>
  /* 样式略 */
</style>
```

2、利用 `Vue.extend` 构造器把 `toast` 组件挂载到 `vue` 实例下
```js
import Vue from 'vue'
import Main from './toast.vue'

let Toast = Vue.extend(Main)

let instance
const toast = function(options) {
  options = options || {}
  instance = new Toast({
    data: options
  })
  instance.vm = instance.$mount()
  document.body.appendChild(instance.vm.$el)
  return instance.vm
}
export default toast
```

3、在 `main.js` 引入 `toast` 组价并挂载在 `vue` 原型上
```js
import Vue from 'vue'
import toast from './components/toast'
Vue.prototype.$toast = toast
```

4、在项目中调用
```js
this.$toast({ msg: '手机号码不能为空' })

this.$toast({
  msg: '成功提示',
  type: 'success'
})
```

#### Vue.extend 和 Vue.component 的区别
* `component`是需要先进行组件注册后，然后在 `template` 中使用注册的标签名来实现组件的使用。`Vue.extend` 则是编程式的写法。
* 控制`component`的显示与否，需要在父组件中传入一个状态来控制或者在组件外部用 `v-if/v-show` 来实现控制，而 `Vue.extend` 的显示与否是手动的去做组件的挂载和销毁。

### Vue.directive
注册或获取全局指令。指令定义函数提供了几个钩子函数（可选）：

* bind: 只调用一次，指令第一次绑定到元素时调用，可以定义一个在绑定时执行一次的初始化动作。
* inserted: 被绑定元素插入父节点时调用（父节点存在即可调用，不必存在于 document 中）。
* update: 被绑定元素所在的模板更新时调用，而不论绑定值是否变化。通过比较更新前后的绑定值。
* componentUpdated: 被绑定元素所在模板完成一次更新周期时调用。
* unbind: 只调用一次， 指令与元素解绑时调用。

#### 应用实例
下面封装一个复制粘贴文本的例子。  

1、编写指令 `copy.js`
```js
const vCopy = { 
  bind (el, { value }) {
    el.$value = value // 用一个全局属性来存传进来的值
    el.handler = () => {
      if (!el.$value) {
        alert('无复制内容')
        return
      }
      // 动态创建 textarea 标签
      const textarea = document.createElement('textarea')
      // 将该 textarea 设为 readonly 防止 iOS 下自动唤起键盘，同时将 textarea 移出可视区域
      textarea.readOnly = 'readonly'
      textarea.style.position = 'absolute'
      textarea.style.left = '-9999px'
      // 将要 copy 的值赋给 textarea 标签的 value 属性
      textarea.value = el.$value
      // 将 textarea 插入到 body 中
      document.body.appendChild(textarea)
      // 选中值并复制
      textarea.select()
      // textarea.setSelectionRange(0, textarea.value.length);
      const result = document.execCommand('Copy')
      if (result) {
        alert('复制成功')
      }
      document.body.removeChild(textarea)
    }
    // 绑定点击事件，就是所谓的一键 copy 啦
    el.addEventListener('click', el.handler)
  },
  // 当传进来的值更新的时候触发
  componentUpdated (el, { value }) {
    el.$value = value
  },
  // 指令与元素解绑的时候，移除事件绑定
  unbind (el) {
    el.removeEventListener('click', el.handler)
  }
}

export default vCopy
```

2、注册指令
```js
import copy from './copy'
// 自定义指令
const directives = {
  copy
}
// 这种写法可以批量注册指令
export default {
  install (Vue) {
    Object.keys(directives).forEach((key) => {
      Vue.directive(key, directives[key])
    })
  }
}
```

3、在 `main.js` 引入并 `use`
```js
import Vue from 'vue'
import Directives from './JS/directives'
Vue.use(Directives)
```

这样就可以在项目直接用 `vCopy` 指令了。