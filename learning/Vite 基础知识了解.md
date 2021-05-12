## 前言

#### Vite（法语单词，“快” 的意思）是一种新型的前端构建工具

最初是配合 Vue3.0 一起使用的，后来适配了各种前端项目，目前提供了 Vue、React、Preact 框架模板

目前来说，Vue 使用的是 vue-cli 脚手架，React 一般使用 create-react-app 脚手架。虽然脚手架工具不一样，但是内部的打包工具都是 Webpack

为什么要开发一个全新的构建工具，是 Webpack 不香了吗？

Vite 方式构建的项目，和使用 Webpack 构建的项目，有什么不同？

一个新工具的出现，一定是为了解决现有工具存在的问题的，否则新工具就没有存在的价值和意义

<br >

<br >

## vite

vite —— 一个由 vue 作者尤雨溪开发的 web 开发工具，它具有以下特点：

1. 快速的冷启动
2. 即时的模块热更新
3. 真正的按需编译

从作者在微博上的发言：

> Vite，一个基于浏览器原生 ES imports 的开发服务器。利用浏览器去解析 imports，在服务器端按需编译返回，完全跳过了打包这个概念，服务器随起随用。同时不仅有 Vue 文件支持，还搞定了热更新，而且热更新的速度不会随着模块增多而变慢。针对生产环境则可以把同一份代码用 rollup 打。虽然现在还比较粗糙，但这个方向我觉得是有潜力的，做得好可以彻底解决改一行代码等半天热更新的问题

中可以看出 vite 主要特点是基于浏览器 native 的 [ES module](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) 来开发，省略打包这个步骤，因为需要什么资源直接在浏览器里引入即可

基于浏览器 ES module 来开发 web 应用也不是什么新鲜事，snowpack 也基于此，不过目前此项目社区中并没有流行起来，vite 的出现也许会让这种开发方式再火一阵子

有趣的是 vite 算是革了 webpack 的命了（生产环境用 rollup）

<br >

### vite 的使用方式

[快速通道](https://vitejs.dev/guide/ )

同常见的开发工具一样，vite 提供了用 npm 或者 yarn 一建生成项目结构的方式，使用 yarn 在终端执行

```js
$ yarn create vite-app <project-name>
$ cd <project-name>
$ yarn
$ yarn dev
```

即可初始化一个 vite 项目（默认应用模板为 vue3.x），生成的项目结构十分简洁

```js
|____node_modules
|____App.vue // 应用入口
|____index.html // 页面入口
|____vite.config.js // 配置文件
|____package.json
```

执行 `yarn dev` 即可启动应用

<br >

<br >

## Vite 解决了 Webpack 哪些问题

随着项目的复杂度升级，代码规范和管理就必须要同步提升。于是，编程社区中提出了多种模块化规范，服务端选择了 CommonJS 规范，客户端选择 AMD 规范较多，但是，两种模块化规范也都存在一定的问题，都是 JS 编程，有两个不同的模块化规范，在 JS 语言层面还是不够的，终于在 ES6 中，ECMA 委员会推出了语言层面模块系统：**ES Modules 规范**

模块化可以帮助我们更好地解决复杂应用开发过程中的代码组织问题，但是随着模块化思想的引入，我们的前端应用又会产生了一些新的问题，比如：

* 首先，我们所使用的 ES Modules 模块系统本身就存在环境兼容问题。尽管现如今主流浏览器的最新版本都支持这一特性，但是目前还无法保证用户的浏览器使用情况。所以我们还需要解决兼容问题

* 其次，模块化的方式划分出来的模块文件过多，而前端应用又运行在浏览器中，每一个文件都需要单独从服务器请求回来。零散的模块文件必然会导致浏览器的频繁发送网络请求，影响应用的工作效率

* 最后，谈一下在实现 JS 模块化的基础上的发散。随着应用日益复杂，在前端应用开发过程中不仅仅只有 JavaScript 代码需要模块化，HTML 和 CSS 这些资源文件也会面临需要被模块化的问题。而且从宏观角度来看，这些文件也都应该看作前端应用中的一个模块，只不过这些模块的种类和用途跟 JavaScript 不同

对于开发过程而言，模块化肯定是必要的，所以我们需要在前面所说的模块化实现的基础之上引入更好的方案或者工具，去解决上面提出的 3 个问题，让我们的应用在开发阶段继续享受模块化带来的优势，又不必担心模块化对生产环境所产生的影响

<br >

*本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器（module bundler）*

<img src="https://qiniu-image.qtshe.com/5Fsdsd179FE8104D.png" style="zoom:60%;float:left;" />

<br >

Vue 脚手架工具 vue-cli 使用 webpack 进行打包，开发时可以启动本地开发服务器，实时预览。因为需要对整个项目文件进行打包，开发服务器启动缓慢

<img src="https://qiniu-image.qtshe.com/5F12312e179FE8104D.png" style="zoom:60%;float:left;" />

<br >

而对于开发时文件修改后的热更新 HMR 也存在同样的问题

Webpack 的热更新会以当前修改的文件为入口重新 build 打包，所有涉及到的依赖也都会被重新加载一次

Vite 则很好地解决了上面的两个问题

<img src="https://qiniu-image.qtshe.com/123123RR179FE8104D.png" style="zoom:60%;float:left;" />

<br >

### 打包问题

vite 只启动一台静态页面的服务器，对文件代码不打包，服务器会根据客户端的请求加载不同的模块处理，实现真正的按需加载

<img src="https://qiniu-image.qtshe.com/QWEQWE123FE8104D.png" style="zoom:60%;float:left;" />

<br >

### 热更新问题

vite 采用立即编译当前修改文件的办法。同时 vite 还会使用缓存机制( http 缓存 => vite 内置缓存 )，加载更新后的文件内容

所以，vite 具有了快速冷启动、按需编译、模块热更新等优良特质

综上所述，vite 构建项目与 vue-cli 构建的项目在开发模式下还是有比较大的区别：

* Vite 在开发模式下不需要打包可以直接运行，使用的是 ES6 的模块化加载规则；Vue-CLI 开发模式下必须对项目打包才可以运行
* Vite 基于缓存的热更新，Vue-CLI 基于 Webpack 的热更新

<br >

<br >

## vite 启动链路

### 命令解析

这部分代码在 src/node/cli.ts 里，主要内容是借助 minimist —— 一个轻量级的命令解析工具解析 npm scripts，解析的函数是 `resolveOptions` ，精简后的代码片段如下

```js
function resolveOptions() {
    // command 可以是 dev/build/optimize
    if (argv._[0]) {
        argv.command = argv._[0]
    }
    return argv
}
```

拿到 options 后，会根据 `options.command` 的值判断是执行在开发环境需要的 runServe 命令或生产环境需要的 runBuild 命令

```js
if (!options.command || options.command === 'serve') {
    runServe(options)
 } else if (options.command === 'build') {
    runBuild(options)
 } else if (options.command === 'optimize') {
    runOptimize(options)
 }
```

在 `runServe` 方法中，执行 server 模块的创建开发服务器方法，同样在 `runBuild` 中执行 build 模块的构建方法

<br >

### server

这部分代码在 src/node/server/index.ts 里，主要暴露一个 `createServer` 方法

vite 使用 koa 作 web server，使用 clmloader 创建了一个监听文件改动的 watcher，同时实现了一个插件机制，将 koa-app 和 watcher 以及其他必要工具组合成一个 context 对象注入到每个 plugin 中

context 组成如下

<img src="https://qiniu-image.qtshe.com/TRDGR5F123E8104D.png" style="zoom:67%;float:left;" />

plugin 依次从 context 里获取上面这些组成部分，有的 plugin 在 koa 实例添加了几个 middleware，有的借助 watcher 实现对文件的改动监听，这种插件机制带来的好处是整个应用结构清晰，同时每个插件处理不同的事情，职责更分明

<br >

### plugin

plugin大致分类如下

- 用户注入的 plugins —— 自定义 plugin
- hmrPlugin —— 处理 hmr
- htmlRewritePlugin —— 重写 html 内的 script 内容
- moduleRewritePlugin —— 重写模块中的 import 导入
- moduleResolvePlugin ——获取模块内容
- vuePlugin —— 处理 vue 单文件组件
- esbuildPlugin —— 使用 esbuild 处理资源
- assetPathPlugin —— 处理静态资源
- serveStaticPlugin —— 托管静态资源
- cssPlugin —— 处理 css/less/sass 等引用

接下来看看 plugin 的实现方式，开发一个用来拦截 json 文件 plugin 可以这么实现

```js
interface ServerPluginContext {
  root: string
  app: Koa
  server: Server
  watcher: HMRWatcher
  resolver: InternalResolver
  config: ServerConfig
}

type ServerPlugin = （ctx:ServerPluginContext）=> void;

const JsonInterceptPlugin:ServerPlugin = ({app})=>{
    app.use(async (ctx, next) => {
      await next()
      if (ctx.path.endsWith('.json') && ctx.body) {
        ctx.type = 'js'
        ctx.body = `export default json`
      }
  })
}
```

vite 背后的原理都在 plugin 里，有兴趣可以每一个都了解一下

<br >

### build

这部分代码在 node/build/index.ts 中，build 目录的结构虽然与 server 相似，同样导出一个 build 方法，同样也有许多 plugin，不过这些 plugin 与 server 中的用途不一样，因为 build 使用了 rollup ，所以这些 plugin 也是为 rollup 打包的 plugin

<br >

结语：Vite原理下期再见

