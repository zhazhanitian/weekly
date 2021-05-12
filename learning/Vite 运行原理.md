## 前言

前面粗略了解了一下 Vite 的基础功能及大致背景，接下来将深入探索其运行原理

<br >

<br >

## ES module

要了解 vite 的运行原理，首先要知道什么是 ES module，目前流览器对其的支持如下

<img src="https://qiniu-image.qtshe.com/19A5AF56-F4D911AB.png" style="zoom:35%;float:left;" />

主流的浏览器（IE11除外）均已经支持，其最大的特点是在浏览器端使用 `export` `import` 的方式导入和导出模块，在 script 标签里设置 `type="module"` ，然后使用模块内容

```js
<script type="module">
  import { bar } from './bar.js‘
</script>
```

当 html 里嵌入上面的 script 标签时候，浏览器会发起 http 请求，请求 htttp server 托管的 bar.js ，在 bar.js 里，我们用 named export 导出 bar 变量，在上面的 script 中能获取到 bar 的定义

```js
// bar.js 
export const bar = 'bar';
```

<br >

<br >

## 在 vite 中的作用

打开运行中的 vite 项目，访问 view-source 可以发现 html 里有段这样的代码

```js
<script type="module">
    import { createApp } from '/@modules/vue'
    import App from '/App.vue'
    createApp(App).mount('#app')
</script>
```

从这段代码中，我们能 get 到以下几点信息

- 从 `http://localhost:3000/@modules/vue` 中获取 `createApp` 这个方法
- 从 `http://localhost:3000/App.vue` 中获取应用入口
- 使用 `createApp` 创建应用并挂载节点

<br >

`createApp` 是 vue3.X 的 api，只需知道这是创建了 vue 应用即可，vite 利用 ES module，把 “构建 vue 应用” 这个本来需要通过 webpack 打包后才能执行的代码直接放在浏览器里执行，这么做是为了：

1. 去掉打包步骤
2. 实现按需加载

<br >

### 去掉打包步骤

打包的概念是开发者利用打包工具将应用各个模块集合在一起形成 bundle，以一定规则读取模块的代码——以便在不支持模块化的浏览器里使用

为了在浏览器里加载各模块，打包工具会借助胶水代码用来组装各模块，比如 webpack 使用 `map` 存放模块 id 和路径，使用 `__webpack_require__` 方法获取模块导出

vite 利用浏览器原生支持模块化导入这一特性，省略了对模块的组装，也就不需要生成 bundle，所以打包这一步就可以省略了

<br >

### 实现按需打包

前面说到，webpack 之类的打包工具会将各模块提前打包进 bundle 里，但打包的过程是静态的——不管某个模块的代码是否执行到，这个模块都要打包到 bundle 里，这样的坏处就是随着项目越来越大打包后的 bundle 也越来越大

开发者为了减少 bundle 大小，会使用动态引入 `import()` 的方式异步的加载模块（ 被引入模块依然需要提前打包)，又或者使用 tree shaking 等方式尽力的去掉未引用的模块，然而这些方式都不如 vite 的优雅，vite 可以只在需要某个模块的时候动态（借助 `import()` ）的引入它，而不需要提前打包，虽然只能用在开发环境，不过这就够了

<br >

<br >

## vite 如何处理 ESM

既然 vite 使用 ESM 在浏览器里使用模块，那么这一步究竟是怎么做的？

上文提到过，在浏览器里使用 ES module 是使用 http 请求拿到模块，所以 vite 必须提供一个 web server 去代理这些模块，上文中提到的 `koa` 就是负责这个事情，vite 通过对请求路径的劫持获取资源的内容返回给浏览器，不过 vite 对于模块导入做了特殊处理

<br >

### @modules 是什么？

通过工程下的 index.html 和开发环境下的 html 源文件对比，发现 script 标签里的内容发生了改变，由

```js
<script type="module">
    import { createApp } from 'vue'
    import App from '/App.vue'
    createApp(App).mount('#app')
</script>
```

变成了

```js
<script type="module">
    import { createApp } from '/@modules/vue'
    import App from '/App.vue'
    createApp(App).mount('#app')
</script>
```

vite 对 import 都做了一层处理，其过程如下

1. 在 koa 中间件里获取请求 body
2. 通过 es-module-lexer 解析资源 ast 拿到 import 的内容
3. 判断 import 的资源是否是绝对路径，绝对视为 npm 模块
4. 返回处理后的资源路径：`"vue" => "/@modules/vue"`

这部分代码在 serverPluginModuleRewrite 这个 plugin 中

<br >

### 为什么需要 @modules？

如果我们在模块里写下以下代码的时候，浏览器中的 esm 是不可能获取到导入的模块内容的

```js
import vue from 'vue'
```

因为 vue 这个模块安装在 node_modules 里，以往使用 webpack，webpack遇到上面的代码，会帮我们做以下几件事

- 获取这段代码的内容
- 解析成 AST
- 遍历 AST 拿到 import 语句中的包的名称
- 使用 enhanced-resolve 拿到包的实际地址进行打包

<br >

但是浏览器中 ESM 无法直接访问项目下的 node_modules，所以 vite 对所有 import 都做了处理，用带有 **@modules** 的前缀重写它们

从另外一个角度来看这是非常比较巧妙的做法，把文件路径的 rewrite 都写在同一个 plugin 里，这样后续如果加入更多逻辑，改动起来不会影响其他 plugin，其他 plugin 拿到资源路径都是 **@modules** ，比如说后续可能加入 alias 的配置：就像 webpack alias 一样：可以将项目里的本地文件配置成绝对路径的引用

<br >

### 怎么返回模块内容

在下一个 koa middleware 中，用正则匹配到路径上带有 **@modules** 的资源，再通过 `require('xxx')` 拿到 包的导出返回给浏览器

以往使用 webpack 之类的打包工具，它们除了将模块组装到一起形成 bundle，还可以让使用了不同模块规范的包互相引用，比如

- ES module (esm) 导入 cjs
- CommonJS (cjs) 导入 esm
- dynamic import 导入 esm
- dynamic import 导入 cjs

<br >

关于 es module 的坑可以看这篇文章(https://zhuanlan.zhihu.com/p/40733281)

起初在 vite 还只是为 vue3.x 设计的时候，对 vue esm 包是经过特殊处理的，比如：需要 `@vue/runtime-dom` 这个包的内容，不能直接通过 `require('``@vue/runtime-dom'`）得到，而需要通过 `require('@vue/runtime-dom/dist/runtime-dom.esm-bundler.js'` 的方式，这样可以使得 vite 拿到符合 esm 模块标准的 vue 包

目前社区中大部分模块都没有设置默认导出 esm，而是导出了 cjs 的包，既然 vue3.0 需要额外处理才能拿到 esm 的包内容，那么其他日常使用的 npm 包是不是也同样需要支持？答案是肯定的，目前在 vite 项目里直接使用 lodash 还是会报错的

<img src="https://qiniu-image.qtshe.com/219A5AEEW23DD911AB.png" style="zoom:70%;float:left;" />

<br >

不过 vite 在最近的更新中，加入了 `optimize` 命令，这个命令专门为解决模块引用的坑而开发，例如我们要在 vite 中使用 lodash，只需要在 **vite.config.js** （vite 配置文件）中，配置 `optimizeDeps` 对象，在 `include` 数组中添加 lodash

```js
// vite.config.js
module.exports = {
  optimizeDeps: {
    include: ["lodash"]
  }
}
```

这样 vite 在执行 runOptimize 的时候中会使用 roolup 对 lodash 包重新编译，将编译成

不过这里还有个问题，由于在 depOptimizer.ts 中，vite 只会处理在项目下 package.json 里的 `dependencies` 里声明好的包进行处理，所以无法在项目里使用

```js
import pick from 'lodash/pick'
```

的方式单使用 pick 方法，而要使用

```js
import lodash from 'lodash'

lodash.pick()
```

的方式，这可能在生产环境下使用某些包的时候对 bundle 的体积有影响

返回模块的内容的代码在：serverPluginModuleResolve.ts 这个 plugin 中

<br >

<br >

## vite 如何编译模块

最初 vite 为 vue3.x 开发，所以这里的编译指的是编译 vue 单文件组件，其他 es 模块可以直接导入内容

<br >

### SFC

vue 单文件组件（简称 SFC） 是 vue 的一个亮点，前端届对 SFC 褒贬不一，个人看来，SFC 是利大于弊的，虽然 SFC 带来了额外的开发工作量，比如为了解析 template 要写模板解析器，还要在 SFC 中解析出逻辑和样式，在 vscode 里要写 vscode 插件，在 webpack 里要写 vue-loader，但是对于使用方来说可以在一个文件里可以同时写 template、js、style，省了各文件互相跳转

与 vue-loader 相似，vite 在解析 vue 文件的时候也要分别处理多次，我们打开浏览器的 network，可以看到

<img src="https://qiniu-image.qtshe.com/23EEE40E2744DD8D.png" style="zoom:40%;float:left;" />

1 个请求的 query 中什么都没有，另 2 个请求分别通过在 query 里指定了 type 为 style 和 template

先来看看如何将一个 SFC 变成多个请求，我们从第一次请求开始分析，简化后的代码如下

```js
function vuePlugin({app}){
  app.use(async (ctx, next) => {
    if (!ctx.path.endsWith('.vue') && !ctx.vue) {
      return next()
    }

    const query = ctx.query
    // 获取文件名称
    let filename = resolver.requestToFile(publicPath)

    // 解析器解析 SFC
    const descriptor = await parseSFC(root, filename, ctx.body)
    if (!descriptor) {
      ctx.status = 404
      return
    }
    // 第一次请求 .vue
    if (!query.type) {
      if (descriptor.script && descriptor.script.src) {
        filename = await resolveSrcImport(descriptor.script, ctx, resolver)
      }
      ctx.type = 'js'
      // body 返回解析后的代码
      ctx.body = await compileSFCMain(descriptor, filename, publicPath)
    }
    
    // ...
}
```

<br >

在 compileSFCMain 中是一段长长的 generate 代码

```js
function compileSFCMain(descriptor, filePath: string, publicPath: string) {
  let code = ''
  if (descriptor.script) {
    let content = descriptor.script.content
    code += content.replace(`export default`, 'const __script =')
  } else {
    code += `const __script = {}`
  }

  if (descriptor.styles) {
    code += `\nimport { updateStyle } from "${hmrClientId}"\n`
    descriptor.styles.forEach((s, i) => {
      const styleRequest = publicPath + `?type=style&index=${i}`
      code += `\nupdateStyle("${id}-${i}", ${JSON.stringify(styleRequest)})`
    })
    if (hasScoped) {
      code += `\n__script.__scopeId = "data-v-${id}"`
    }
  }

  if (descriptor.template) {
    code += `\nimport { render as __render } from ${JSON.stringify(
      publicPath + `?type=template`
    )}`
    code += `\n__script.render = __render`
  }
  code += `\n__script.__hmrId = ${JSON.stringify(publicPath)}`
  code += `\n__script.__file = ${JSON.stringify(filePath)}`
  code += `\nexport default __script`
  return code
}
```

<br >

直接看 generate 后的代码

```js
import { updateStyle } from "/vite/hmr"
updateStyle("c44b8200-0", "/App.vue?type=style&index=0")
__script.__scopeId = "data-v-c44b8200"
import { render as __render } from "/App.vue?type=template"
__script.render = __render
__script.__hmrId = "/App.vue"
__script.__file = "/Users/muou/work/playground/vite-app/App.vue"
export default __script
```

<br >

出现了 `vite/hmr` 的导入，`vite/hmr` 具体内容我们下文再分析，从这段代码中可以看到，对于 style vite 使用 `updateStyle` 这个方法处理，`updateStyle` 内容非常简单，这里就不贴代码了，就做了 1 件事：通过创建 style 元素，设置了它的 innerHtml 为 css 内容

这两种方式都会使得浏览器发起 http 请求，这样就能被 koa 中间件捕获到了，所以就形成了上文我们看到的：对一个 .vue 文件处理三次的情景

这部分代码在：serverPluginVue 这个 plugin 里

<br >

### css

如果在 vite 项目里引入一个 sass 文件会怎么样？

最初 vite 只是为 vue 项目开发，所以并没有对 css 预编译的支持，不过随着后续的几次大更新，在 vite 项目里使用 sass/less 等也可以跟使用 webpack 的时候一样优雅了，只需要安装对应的 css 预处理器即可

在 cssPlugin 中，通过正则：`/(.+).(less|sass|scss|styl|stylus)$/` 判断路径是否需要 css 预编译，如果命中正则，就借助 cssUtils 里的方法借助 postcss 对要导入的 css 文件编译

<br >

<br >

## vite 热更新的实现

上文中出现了 vite/hmr ，这就是 vite 处理热更新的关键，在 serverPluginHmr plugin 中，对于 `path` 等于  `vite/hmr` 做了一次判断

```js
app.use(async (ctx, next) => {
  if (ctx.path === '/vite/hmr') {
      ctx.type = 'js'
      ctx.status = 200
      ctx.body = hmrClient
  }
}
```

hmrClient 是 vite 热更新的客户端代码，需要在浏览器里执行，这里先来说说通用的热更新实现，热更新一般需要四个部分：

1. 首先需要 web 框架支持模块的 rerender/reload
2. 通过 watcher 监听文件改动
3. 通过 server 端编译资源，并推送新模块内容给 client 

1. client 收到新的模块内容，执行 rerender/reload

<br >

vite 也不例外同样有这四个部分，其中客户端代码在：client.ts 里，服务端代码在 serverPluginHmr 里，对于 vue 组件的更新，通过 vue3.x 中的 `HMRRuntime` 处理的

<br >

### client 端

在 client 端， `WebSocket` 监听了一些更新的类型，然后分别处理，它们是

- vue-reload —— vue 组件更新：通过 import 导入新的 vue 组件，然后执行 `HMRRuntime.reload`
- vue-rerender —— vue template 更新：通过 import 导入新的 template ，然后执行 `HMRRuntime.rerender`
- vue-style-update —— vue style 更新：直接插入新的 stylesheet
- style-update —— css 更新：document 插入新的 stylesheet
- style-remove —— css 移除：document 删除 stylesheet
- js-update —— js 更新：直接执行
- full-reload —— 页面 roload：使用 `window.reload` 刷新页面

<br >

### server 端

在 server 端，通过 watcher 监听页面改动，根据文件类型判断是 js Reload 还是 Vue Reload

```js
watcher.on('change', async (file) => {
    const timestamp = Date.now()
    if (file.endsWith('.vue')) {
      handleVueReload(file, timestamp)
    } else if (
      file.endsWith('.module.css') ||
      !(file.endsWith('.css') || cssTransforms.some((t) => t.test(file, {})))
    ) {
      // everything except plain .css are considered HMR dependencies.
      // plain css has its own HMR logic in ./serverPluginCss.ts.
      handleJSReload(file, timestamp)
    }
})
```

<br >

在 handleVueReload 方法里，会使用解析器拿到当前文件的 template/script/style ，并且与缓存里的上一次解析的结果进行比较，如果 template 发生改变就执行 vue-rerender，如果 style 发生改变就执行 vue-style-update，简化后的逻辑如下

```js
async function handleVueReload(
    file
    timestamp,
    content
  ) {
    // 获取缓存
    const cacheEntry = vueCache.get(file）

    // 解析 vue 文件                                 
    const descriptor = await parseSFC(root, file, content)
    if (!descriptor) {
      // read failed
      return
    }
    // 拿到上一次解析结果
    const prevDescriptor = cacheEntry && cacheEntry.descriptor
    
    // 设置刷新变量
    let needReload = false // script 改变标记
    let needCssModuleReload = false // css 改变标记
    let needRerender = false // template 改变标记

    // 判断 script 是否相同
    if (!isEqual(descriptor.script, prevDescriptor.script)) {
      needReload = true
    }

     // 判断 template 是否相同
    if (!isEqual(descriptor.template, prevDescriptor.template)) {
      needRerender = true
    }
      
    // 通过 send 发送 socket
    if (needRerender){
      send({
        type: 'vue-rerender',
        path: publicPath,
        timestamp
      })  
    }
 }
```

`handleJSReload` 方法则是根据文件路径引用，判断被哪个 vue 组件所依赖，如果未找到 vue 组件依赖，则判断页面需要刷新，否则走组件更新逻辑，这里就不贴代码了

整体代码在 client.ts 和 serverPluginHmr.ts 里

