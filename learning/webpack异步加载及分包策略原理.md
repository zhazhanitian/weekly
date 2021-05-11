#### webpack 异步加载

webpack ensure 有人称它为异步加载，也有人称为代码切割，他其实就是将 js 模块给独立导出一个.js 文件，然后使用这个模块的时候，再创建一个 script 对象，加入到 document.head 对象中，浏览器会自动帮我们发起请求，去请求这个 js 文件，然后写个回调函数，让请求到的 js 文件做一些业务操作





#### 看个例子

##### 场景

main.js 依赖两个 js 文件，testA 是点击 aBtn 按钮后，才执行的逻辑，testB 是点击 bBtn 按钮后，才执行的逻辑



##### Vue项目结构

<img src="https://qiniu-image.qtshe.com/EC79565BA5.png" style="zoom:40%;float:left;" />



##### webpack 打包的配置的代码

```js
// webpack.config.js

const path = require('path') // 路径处理模块
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin') // 引入CleanWebpackPlugin插件

module.exports = {
  entry: {
    index: path.join(__dirname, '/src/main.js'),
  },
  output: {
    path: path.join(__dirname, '/dist'),
    filename: 'index.js',
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, '/index.html'),
    }),
    new CleanWebpackPlugin(), // 所要清理的文件夹名称
  ],
}
```



##### App.vue页面代码

```vue
<template>
  <div id="app">
    <button id="aBtn">按钮A</button>
    <button id="bBtn">按钮B</button>
  </div>
</template>

<script>
import testA from './testA.js'
import testB from './testB.js'

export default {
  name: 'app',

  mounted() {
    document.getElementById('aBtn').onclick = function () {
      alert(testA)
    }

    document.getElementById('bBtn').onclick = function () {
      alert(testB)
    }
  }
}
</script>
```



##### test.js 文件内容

```js
// testA.js
const testA = 'hello A'
module.exports = testA

// testB.js
const testB = 'hello B'
module.exports = testB
```



##### 构建结论

此时，我们对项目进行 npm run build， 打包出来的只有两个 JS 文件（map文件不算）

<img src="https://qiniu-image.qtshe.com/8290ED18378.png" style="zoom:50%;float:left;" />

由此可见，此时 webpack 把 main.js 依赖的两个文件都同时打包到同一个 js 文件，并在 index.html 中引入。但是 testA.js 和 testB.js 都是点击相应按钮才会执行的逻辑，如果用户并没有点击相应按钮，而且这两个文件又是比较大的话，这样是不是就导致首页默认加载的 js 文件太大，从而导致首页渲染较慢呢？那么有能否实现当用户点击按钮的时候再加载相应的依赖文件呢？

webpack.ensure 就解决了这个问题





#### require.ensure 异步加载

##### JS 依赖异步加载

下面我们将 JS 依赖改成异步加载的方式

```vue
<template>
  <div id="app">
    <button id="aBtn">按钮A</button>
    <button id="bBtn">按钮B</button>
  </div>
</template>

<script>
export default {
  name: 'app',

  mounted() {
    document.getElementById('aBtn').onclick = function () {
      //异步加载A
      require.ensure([], function () {
        let A = require('./testA.js')
        alert(A)
      })
    }

    document.getElementById('bBtn').onclick = function () {
      //异步加载A
      require.ensure([], function () {
        let B = require('./testB.js')
        alert(B)
      })
    }
  }
}
</script>
```



##### 构建

此时，我们再进行一下打包

<img src="https://qiniu-image.qtshe.com/572F7D19B46.png" style="zoom:50%;float:left;" />

发现多了两个 JS 文件，而我们打开页面时只引入了 index.js 一个文件，当点击按钮 A 的时候才引入 testA.js 文件，点击按钮 B 的时候才引入 testB.js 文件，这样就满足了我们按需加载的需求

require.ensure 这个函数是一个代码分离的分割线，表示回调里面的 require 是我们想要进行分割出去的，即 require('./testA.js')，把 testA.js 分割出去，形成一个 webpack 打包的单独 js 文件，它的语法如下

```js
require.ensure(dependencies: String[], callback: function(require), chunkName: String)
```

我们打开 testA.js 的打包文件，发现它的代码如下

```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([
  ["chunk-2d0d0056"],
  {
    "65cd": function(n, o) {
      var p = "hello A";
      n.exports = p;
    }
  }
]);
//# sourceMappingURL=chunk-2d0d0056.a12d217c.js.map
```

由上面的代码可以看出

1. 异步加载的代码，会保存在一个全局的 webpackJsonp 中
2. webpackJsonp.push 的的值，两个参数分别为异步加载的文件中存放的需要安装的模块对应的 id 和异步加载的文件中存放的需要安装的模块列表
3. 在满足某种情况下，会执行具体模块中的代码





#### import() 按需加载

webpack4+ 官方文档提供了模块按需切割加载，配合 es6 的按需加载 import()  方法，可以做到减少首页包体积，加快首页的请求速度，只有其他模块，只有当需要的时候才会加载对应 js

import() 的语法十分简单，该函数只接受一个参数，就是引用包的地址，并且使用了 promise 式的回调，获取加载的包。在代码中所有被 import() 的模块，都将打成一个单独的包，放在 chunk 存储的目录下。在浏览器运行到这一行代码时，就会自动请求这个资源，实现异步加载

下面我们将上述代码改成 import() 方式

```vue
<template>
  <div id="app">
    <button id="aBtn">按钮A</button>
    <button id="bBtn">按钮B</button>
  </div>
</template>

<script>
export default {
  name: 'app',

  mounted() {
    document.getElementById('aBtn').onclick = function () {
      //异步加载A
      import('./testA').then(data => {
        alert(data.A)
      })
    }

    document.getElementById('bBtn').onclick = function () {
      //异步加载b
      import('./testB').then(data => {
        alert(data.B)
      })
    }
  }
}
</script>
```

此时打包出来的文件和 webpack.ensure 方法是一样的





#### 路由懒加载

为什么需要懒加载？像 vue 这种单页面应用，如果没有路由懒加载，运用 webpack 打包后的文件将会很大，造成进入首页时，需要加载的内容过多，出现较长时间的白屏，运用路由懒加载则可以将页面进行划分，需要的时候才加载页面，可以有效的分担首页所承担的加载压力，减少首页加载用时

vue 路由懒加载有以下三种方式

- vue 异步组件
- ES6 的  import()
- webpack 的 require.ensure()



##### vue 异步组件

这种方法主要是使用了 resolve 的异步机制，用 require 代替了 import 实现按需加载

```js
export default new Router({
  routes: [
    {
      path: '/home',',
      component: (resolve) => require(['@/components/home'], resolve),
    },
    {
      path: '/detail',',
      component: (resolve) => require(['@/components/detail'], resolve),
    }
  ]
})
```



##### ES6 的 import()

vue-router 在官网提供了一种方法，可以理解也是为通过 Promise 的 resolve 机制，因为 Promise 函数返回的 Promise 为 resolve 组件本身，而我们又可以使用 import 来导入组件

```js
export default new Router({
  routes: [
    {
      path: '/home',
      component: () => import('@/components/home'),
    },
    {
      path: '/detail',
      component: () => import('@/components/detail')
    }
  ]
})
```



##### require.ensure

这种模式可以通过参数中的 webpackChunkName 将 js 分开打包

```js
export default new Router({
  routes: [
    {
      path: '/home',
      component: (resolve) => require.ensure([], () => resolve(require('@/components/home')), 'home')
    },
    {
      path: '/detail',
      component: (resolve) => require.ensure([], () => resolve(require('@/components/detail')), 'detail')
    }
  ]
})
```





#### **webpack 分包策略**

在 webpack 打包过程中，经常出现 vendor.js， app.js 单个文件较大的情况，这偏偏又是网页最先加载的文件，这就会使得加载时间过长，从而使得白屏时间过长，影响用户体验。所以我们需要有合理的分包策略



##### CommonsChunkPlugin

在 Webapck4.x 版本之前，我们都是使用 CommonsChunkPlugin 去做分离

```js
plugins: [
  new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks: function (module, count) {
      return (
        module.resource &&
        /.js$/.test(module.resource) &&
        module.resource.indexOf(path.join(__dirname, './node_modules')) === 0
      )
    }
  }),
  new webpack.optimize.CommonsChunkPlugin({
    name: 'common',
    chunks: 'initial',
    minChunks: 2
  })
]
```

我们把以下文件单独抽离出来打包

​	•	node_modules 文件夹下的，模块

​	•	被 3 个 入口 chunk 共享的模块



##### optimization.splitChunks

webpack 4+ 最大的改动就是废除了 CommonsChunkPlugin 引入了 optimization.splitChunks。如果你的 mode 是 production，那么 webpack4+ 就会自动开启 Code Splitting

它内置的代码分割策略是这样的

- 新的 chunk 是否被共享或者是来自 node_modules 的模块
- 新的 chunk 体积在压缩之前是否大于 30kb
- 按需加载 chunk 的并发请求数量小于等于 5 个
- 页面初始加载时的并发请求数量小于等于 3 个

虽然在 webpack4+ 会自动开启 Code Splitting，但是随着项目工程的最大，这往往不能满足我们的需求，我们需要再进行个性化的优化

