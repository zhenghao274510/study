### 前言
在很久很久以前，在我们前端还只是页面切图仔的年代，我们开发一个html页面，通常会遇到这些情况：
* 需要引入十几个css和js文件，而且因为他们彼此间有着依赖关系，所以引入的顺序还不能乱。
* 传统的``html+css+js``开发方式不能不能很好地运用``less/scss``等css预处理器以及``ES6+``的高级语法。
* 代码复用性差，可维护性差。  

此时就需要一个处理这些问题的工具，webpack应运而生。

webpack可以看做是模块打包工具：它将各种静态资源（比如：``javaScript`` 文件，图片文件，``css``文件等）视为模块，它能够对这些模块进行解析优化和转换等操作，最后将它们打包在一起，打包后的文件可用于在浏览器中使用。

### webpack的优点：
* 代码转换: ``typeScript`` 编译成 ``javaScript``、``scss，less`` 编译成 ``css``.
* 文件优化：压缩 ``javaScript``、``css``、``html`` 代码，压缩合并图片。
* 代码分割：提取多个页面的公共代码、提取首屏不需要执行部分的代码让其异步加载。
* 模块合并：在采用模块化的项目里会有很多个模块和文件，需要构建功能把模块分类合并成一个文件。
* 自动刷新：监听本地源代码的变化，自动重新构建、刷新浏览器。
* 扩展性强，插件机制完善。


### webpack打包过程：

1. 利用babel完成代码转换,并生成单个文件的依赖
2. 从入口开始递归分析，并生成依赖图谱
3. 将各个引用模块打包为一个立即执行函数
4. 将最终的bundle文件写入bundle.js中


### Webpack 的四大核心：

- entry：js 入口源文件
- output：生成文件
- loader：进行文件处理
- plugins：插件，比 loader 更强大，能使用更多 webpack 的 api

## Entry
webpack 应该使用哪个模块做为入口文件，来作为构建其内部依赖图的开始。进去入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的，每个依赖项随即被处理，最后输出到称之为 bundles 的文件中。  


单⼊⼝：entry 是⼀个字符串

```js
module.exports = {
  entry: './src/index.js'
}
```

多⼊⼝：entry 是⼀个对象

```js
module.exports = {
  entry: {
    index: './src/index.js',
    manager: './src/manager.js'
  }
}
```

## Output
告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，这些都可以在webpack的配置文件中指定。


单⼊⼝配置

```js
module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js’,
        path: __dirname + '/dist'
    }
};
```

多⼊⼝配置

```js
module.exports = {
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
}
```

通过[name]占位符确保⽂件名称的唯⼀

## Loader

``loader`` 让 ``webpack`` 能够去处理那些非 ``javaScript`` 文件（``webpack`` 自身只理解 ``javaScript``）。``loader`` 可以将所有类型的文件转换为 ``webpack`` 能够处理的有效模块，然后你就可以利用 ``webpack`` 的打包能力，对它们进行处理。

### loader的特点
* 一个Loader 的职责是单一的，只需要完成一种转换
* 一个Loader 其实就是一个Node.js 模块，这个模块需要导出一个函数
* loader 总是从右到左地被调用。

### 常用的loader
#### 处理样式
* ``css-loader``: 加载.css 文件，
* ``style-loader``:使用 style 标签将 ``css-loader`` 内部样式注入到我们的 html 页面
* ``less-loader, sass-loader``: 解析css预处理器

#### 处理 js
* 让你能使用最新的js代码（ES6，ES7...）
* 让你能使用基于js进行了拓展的语言，比如React的JSX；

#### 处理文件

处理图片资源时，我们常用的两种loader是``file-loader``或者``url-loader``，两者的主要差异在于。``url-loader``可以设置图片大小限制，当图片超过限制时，其表现行为等同于``file-loader``，而当图片不超过限制时，则会将图片以``base64``的形式打包进css文件，以减少请求次数

#### 处理.vue文件  

``vue-loader`` 是 ``webpack`` 的加载器模块，它使我们可以用 ``.vue`` 文件格式编写单文件组件。单文件组件文件有三个部分，即模板、脚本和样式。 ``vue-loader`` 模块允许 ``webpack`` 使用单独的加载器模块（例如 ``sass 或 scss 加载器``）提取和处理每个部分。该设置使我们可以使用 ``.vue`` 文件无缝编写程序。

### 开发一个loader
<!-- 1. 基本形式
```js
module.exports = function (source ) { 
  return source; 
}
```

2. 调用第三方模块
```js
const sass= require('node-sass'); 
  module.exports = function (source) { 
    return sass(source);
}
``` -->

需求：手写一个 ``loader``，将 ``'kobe'`` 转换成 ``'Black Mamba'``。当然大家可以根据自己的需求进行设计。这里只是讲解方法。

#### 1、编写 loader

在根目录下，新建目录 ``kobe-loader`` 作为我们编写 ``loader`` 的名称，执行 ``npm init -y`` 命令，新建一个模块化项目，然后新建 ``index.js`` 文件，相关源码如下：

```js
module.exports = function(content) {
  return content && content.replace(/kobe/gi, 'Black Mamba')
}
```

#### 2、注册模块

正常我们安装的 ``loader`` 是从 ``npm`` 下载安装，但是我们可以在 ``kobe-loader`` 目录底下使用 ``npm link`` 做到在不发布模块的情况下，将本地的一个正在开发的模块的源码链接到项目的 ``node_modules`` 目录下，让项目可以直接使用本地的 ``npm`` 模块。
```
npm link
```

然后在项目根目录执行以下命令，将注册到全局的本地 ``npm`` 模块链接到项目的 ``node_modules`` 下

```
$ npm link kobe-loader
```

注册成功后，我们可以在 ``node_modules`` 目录下能查找到对应的 ``loader``。

#### 3、在 webpack 中配置 loader

在 ``webpack.base.conf.js`` 加上如下配置

```js
{
  test:/\.js/,
  loader: 'kobe-loader'
}
```

此时，我们在所有 js 文件下书写的 ``'kobe'`` 就全部替换成 ``'Black Mamba'``了。

#### 4、配置参数

上面我们是写死的替换文案，假如我想通过配置项来改变，可以在 loader 中做以下调整

```js
// custom-loader/index.js
var utils = require('loader-utils')
module.exports = function (content) {
  const options = utils.getOptions(this)
  return content && content.replace(/kobe/gi, options.name)
}

// webpack.base.conf.js
{
  test:/\.js/,
  use: {
    loader: 'kobe-loader',
    options: {
      name: 'kobe',
    }
  }
}
```


## Plugin
专注处理 webpack 在编译过程中的某个特定的任务的功能模块，可以称为插件。

### Plugin 的特点
* 是一个独立的模块
* 模块对外暴露一个 js 函数
* 函数的原型 ``(prototype)`` 上定义了一个注入 ``compiler`` 对象的 ``apply ``方法 ``apply`` 函数中需要有通过 ``compiler`` 对象挂载的 ``webpack`` 事件钩子，钩子的回调中能拿到当前编译的 ``compilation`` 对象，如果是异步编译插件的话可以拿到回调 ``callback``
* 完成自定义子编译流程并处理 ``complition`` 对象的内部数据
* 如果异步编译插件的话，数据处理完成后执行 ``callback`` 回调。

### 常用Plugin
- ``HotModuleReplacementPlugin`` 代码热替换。因为 ``Hot-Module-Replacement`` 的热更新是依赖于 ``webpack-dev-server``，后者是在打包文件改变时更新打包文件或者 reload 刷新整个页面，``HRM`` 是只更新修改的部分。
- ``HtmlWebpackPlugin``, 生成 html 文件。将 webpack 中`entry`配置的相关入口 chunk 和 `extract-text-webpack-plugin`抽取的 css 样式 插入到该插件提供的`template`或者`templateContent`配置项指定的内容基础上生成一个 html 文件，具体插入方式是将样式`link`插入到`head`元素中，`script`插入到`head`或者`body`中。

- ``ExtractTextPlugin``, 将 css 成生文件，而非内联 。该插件的主要是为了抽离 css 样式,防止将样式打包在 js 中引起页面样式加载错乱的现象。
- ``NoErrorsPlugin``报错但不退出 webpack 进程
- ``UglifyJsPlugin``，代码丑化，开发过程中不建议打开。 ``uglifyJsPlugin`` 用来对 js 文件进行压缩，从而减小 js 文件的大小，加速 load 速度。``uglifyJsPlugin`` 会拖慢 webpack 的编译速度，所有建议在开发简单将其关闭，部署的时候再将其打开。多个 html 共用一个 js 文件(chunk)，可用 ``CommonsChunkPlugin``

- ``purifycss-webpack``  。打包编译时，可剔除页面和 js 中未被使用的 css，这样使用第三方的类库时，只加载被使用的类，大大减小 css 体积
- ``optimize-css-assets-webpack-plugin``   压缩 css，优化 css 结构，利于网页加载和渲染
- ``webpack-parallel-uglify-plugin``   可以并行运行 UglifyJS 插件，这可以有效减少构建时间



### 开发一个 plugin

- Webpack 在编译过程中，会广播很多事件，例如 run、compile、done、fail 等等，可以查看官网；
- Webpack 的事件流机制应用了观察者模式，我们编写的插件可以监听 Webpack 事件来触发对应的处理逻辑；
- 插件中可以使用很多 Webpack 提供的 API，例如读取输出资源、代码块、模块及依赖等；

#### 1、编写插件

在根目录下，新建目录 my-plugin 作为我们编写插件的名称，执行 npm init -y 命令，新建一个模块化项目，然后新建 index.js 文件，相关源码如下：

```js
class MyPlugin {
  constructor(doneCallback, failCallback) {
    // 保存在创建插件实例时传入的回调函数
    this.doneCallback = doneCallback
    this.failCallback = failCallback
  }
  apply(compiler) {
    // 成功完成一次完整的编译和输出流程时，会触发 done 事件
    compiler.plugin('done', stats => {
      this.doneCallback(stats)
    })
    // 在编译和输出的流程中遇到异常时，会触发 failed 事件
    compiler.plugin('failed', err => {
      this.failCallback(err)
    })
  }
}
module.exports = MyPlugin
```

#### 2、注册模块

按照以上的方法，我们在 my-plugin 目录底下使用 npm link 做到在不发布模块的情况下，将本地的一个正在开发的模块的源码链接到项目的 node_modules 目录下，让项目可以直接使用本地的 npm 模块。

```
npm link
```

然后在项目根目录执行以下命令，将注册到全局的本地 npm 模块链接到项目的 node_modules 下

```
$ npm link my-plugin
```

注册成功后，我们可以在 node_modules 目录下能查找到对应的插件了。

#### 3、配置插件
在 webpack.base.conf.js 加上如下配置

```js
plugins: [
  new MyPlugin(
    stats => {
      console.info('编译成功!')
    },
    err => {
      console.error('编译失败!')
    }
  )
]
```
执行运行 or 编译命令，就能看到我们的 plugin 起作用了。