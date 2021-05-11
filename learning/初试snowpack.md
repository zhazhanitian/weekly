#### 前言

前几天看一篇关于 snowpack 简介的文章，便去去看了下这个包是干什么的，看了下官网，发现这个东西还是蛮有意思的，号称无需打包工具（Webpack，Parcel）便能将代码结果实时展现在浏览器中，可以先看以下的图，是不是很惊人

<img src="https://qiniu-image.qtshe.com/6649A0C97D91.png" style="zoom:30%;float:left;" />

随着 vue3、vite 的热度上升，与 vite 类似的 snowpack 的关注度也逐渐增加了，目前 snowpack 在 Github 上已经有了将近 1w stars，snowpack 的代码很轻量，接下来会从实现原理的角度开始了解 snowpack 的特点。同时看看，作为一个以原生 JavaScript 模块化为核心的年轻的构建工具，它是如何实现“老牌”构建工具所提供的那些特性的





#### snowpack发展

2019 一篇名为 A Future Without Webpack 的文章开始将 snowpack 带入众人的视野， 通过它可以简单的了解到了 pika/snowpack 这个项目，不过当时还叫 pika/web

文章的核心观点如下

> 在如今（2019年），我们完全可以抛弃打包工具，而直接在浏览器中使用浏览器原生的 JavaScript 模块功能。这主要基于三点考虑：
>
> 1. 兼容性可接受：基本主流的浏览器版本都支持直接使用 JavaScript Module 了（当然，IE 一如既往除外）
> 2. 性能问题的改善：之前打包的一个重要原因是 HTTP/1.1 的特性导致，我们合并请求来优化性能；而如今 HTTP/2 普及之后，这个性能问题不像以前那么突出了
> 3. 打包的必要性：打包工具的存在主要就是为了处理模块化与合并请求，而以上两点基本解决这两个问题；再加之打包工具越来越复杂，此消彼长，其存在的必要性自然被作者所质疑

很多人认为 webpack 之类的打包工具，“发家”后转型做构建工具并非最优解，实是一种阴差阳错的阶段性成果，所以当时对这个项目提到的观点也很赞同，其中最令人深省的当属它提到的

> In 2019, you should use a bundler because you want to, not because you need to.



snowpack 的最初版核心目标就是不再打包业务代码，而是直接使用浏览器原生的 JavaScript Module 能力

<img src="https://qiniu-image.qtshe.com/9259C50DA1A.png" style="zoom:70%;float:left;margin: 20px 0px;" />

所以从它的处理流程上来看，对业务代码的模块，基本只需要把 ESM 发布（拷贝）到发布目录，再将模块导入路径从源码路径换为发布路径即可

而对 node_modules 则通过遍历 package.json 中的依赖，按该依赖列表为粒度将 node_modules 中的依赖打包。以 node_modules 中每个包的入口作为打包 entry，使用 rollup 生成对应的 ESM 模块文件，放到 web_modules 目录中，最后替换源码的 import 路径，是得可以通过原生 JavaScript Module 来加载 node_modules 中的包

```js
- import { createElement, Component } from "preact";
- import htm from "htm";

+ import { createElement, Component } from "/web_modules/preact.js";
+ import htm from "/web_modules/htm.js";
```

对比了一下19年版本，从 v0.4.0 版本的源码可以看出，其初期功能确实非常简单，甚至有些简陋，以至于缺乏很多现代前端开发所需的特性，明显是不能用于生产环境的

大众的直观感受来说，它当时就欠缺以下能力，所以没能引入洪流：

1. import CSS / image / …：由于 webpack 一切皆模块的理念 + 组件化开发的深入人心，import anything 的书写模式已经深入开发者的观念中，对 CSS 等内容依赖与加载能力的缺失，将成为它的阿克琉斯之踵
2. 语法转换能力：作为目标成为构建工具的 snowpack，并没有能够编译 Typescript、JSX 等语法文件的能力，你当然可以再弄一个和它毫无关系的工具来处理语法，但这不就是构建工具应该集成的吗
3. HMR：这可能不那么要命，但俗话说「由俭入奢易，由奢入俭难」，被“惯坏”开发者们自然会有人抵触这一特性的缺失
4. 性能：虽说它指出，上了 HTTP2 后，使用 JavaScript modules 性能并不会差，但毕竟没有实践过，对此还是抱有怀疑
5. 环境变量：这虽然是一个小特性，但在我接触过的大多数项目中都会用到它，它可以帮助开发者自动测卸载线上代码中的调试工具，可以根据环境判断，自动将埋点上报到不同的服务上。确实需要一个这样好用的特性





#### snowpack 的进化

2020 年上半年，随着 vue3 的不断曝光，与其有一定关联的另一个项目 vite 也逐渐吸引了人们的目光。而其介绍中提到的 snowpack 也突然吸引到了更多的热度与讨论。当时很多人可能只是对 pika 感到熟悉，好奇才会点开 snowpack 项目主页，这时snowpack 以 pika/snowpack v2 版本再次走入大众视野。而且项目源码也不再是之前那唯一而简单的 index.ts，在核心代码外，还包含了诸多官方插件

已经完全变样的 Readme，明显之前的那些问题，应该已经有了解决方案

<img src="https://qiniu-image.qtshe.com/1608445154045.jpg" style="zoom:67%;float:left;" />

##### import CSS

import CSS 的问题还有一个更大的范围，就是非 JavaScript 资源的加载，包括图片、JSON 文件、文本等

```js
import './index.css'
```

上面这种语法目前浏览是不支持的。所以 snowpack 用了一个和之前 webpack 很类似的方式，将 CSS 文件变为用于注入样式的 JS 模块。如果熟悉 webpack，肯定知道如果只是在 loader 中处理 CSS，那么并不会生成单独的 CSS 文件（这就是为什么会有  [mini-css-extract-plugin](https://link.zhihu.com/?target=https%3A//webpack.js.org/plugins/mini-css-extract-plugin/)），而是加载一个 JS 模块，然后在 JS 模块中通过 DOM API 将 CSS 文本作为 style 标签的内容插入到页面中

为此，snowpack 自己写了一个简单的模板方法，生成将 CSS 样式注入页面的 JS 模块。下面这段代码可以实现样式注入的功能：

```js
const code = '.test { height: 100px }';
const styleEl = document.createElement("style");
const codeEl = document.createTextNode(code);
styleEl.type = 'text/css';
styleEl.appendChild(codeEl);
document.head.appendChild(styleEl);
```

可以看到，除了第一行式子的右值，其他都是不变的，因此可以很容易生成一个符合需求的 JS 模块：

```js
const jsContent = `
  const code = ${JSON.stringify(code)};
  const styleEl = document.createElement("style");
  const codeEl = document.createTextNode(code);
  styleEl.type = 'text/css';
  styleEl.appendChild(codeEl);
  document.head.appendChild(styleEl);
`;
fs.writeFileSync(filename, jsContent);
```

snowpack 中的实现代码比我们上面多了一些东西，不过与样式注入无关

通过将 CSS 文件的内容保存到 JS 变量，然后再使用 JS 调用 DOM API 在页面注入 CSS 内容即可使用 JavaScript Modules 的能力加载 CSS。而源码中的 index.css 也会被替换为 index.css.proxy.js：

```js
- import './index.css';
+ import './index.css.proxy.js';
```

proxy 这个名词之后会多次出现，因为为了能够以模块化方式导入非 JS 资源，snowpack 把生成的中间 JavaScript 模块都叫做 proxy。这种实现方式也几乎和 webpack 一脉相承



##### 图片的 import

在目前的前端开发场景中，还有一类非常典型的资源就是图片

```js
import avatar from './avatar.png'
function render() {
  return (
    <div class="user">
      <img src={avatar} />
    </div>
  );
}
```

上面代码的书写方式已经普遍应用在很多项目代码中了，接着对比一下 snowpack 是怎么处理的

太阳底下没有新鲜事，snowpack 和 webpack 一样，对于代码中导入的 avatar 变量，最后其实都是该静态资源的 URI

以 snowpack 提供的官方 React 模版为例来看看图片资源的引入处理

```js
npx create-snowpack-app snowpack-test --template @snowpack/app-template-react
```

初始化模版运行后，可以看到源码与构建后的代码差异如下：

```js
- import React, { useState } from 'react'
- import logo from './logo.svg'
- import './App.css'

+ import React, { useState } from '/web_modules/react.js'
+ import logo from './logo.svg.proxy.js'
+ import './App.css.proxy.js'
```

与 CSS 类似，也为图片（svg）生成了一个 JS 模块 logo.svg.proxy.js，其模块内容为

```js
// logo.svg.proxy.js
export default "/_dist_/logo.svg"
```

套路与 webpack 如出一辙。以 build 命令为例，来看一下 snowpack 的处理方式

首先是将源码中的静态文件（logo.svg）拷贝到发布目录

```js
allFiles = glob.sync(`**/*`, {
  ...
})
const allBuildNeededFiles: string[] = []
await Promise.all(
  allFiles.map(async (f) => {
    f = path.resolve(f); // this is necessary since glob.sync() returns paths with / on windows.  path.resolve() will switch them to the native path separator.
    ...
    return fs.copyFile(f, outPath);
  }),
)
```

然后，可以看到 snowpack 中的一个叫  transformEsmImports 的关键方法调用。这个方法可以将源码 JS 中 import 的模块路径进行转换。例如对 node_modules 中的导入都替换为 web_modules。在这里对 svg 文件的导入名也会被加上 .proxy.js：

```js
code = await transformEsmImports(code, (spec) => {
  ……
  if (spec.startsWith('/') || spec.startsWith('./') || spec.startsWith('../')) {
    const ext = path.extname(spec).substr(1);
    if (!ext) {
      ……
    }
    const extToReplace = srcFileExtensionMapping[ext];
    if (extToReplace) {
      ……
    }
    if (spec.endsWith('.module.css')) {
      ……
    } else if (!isBundled && (extToReplace || ext) !== 'js') {
      const resolvedUrl = path.resolve(path.dirname(outPath), spec);
      allProxiedFiles.add(resolvedUrl);
      spec = spec + '.proxy.js';
    }
    return spec;
  }
  ……
})
```

此时， svg 文件和源码的导入语法（ import logo from './logo.svg.proxy.js'）均已就绪，最后剩下的就是生成 proxy 文件了。也非常简单：

```js
for (const proxiedFileLoc of allProxiedFiles) {
  const proxiedCode = await fs.readFile(proxiedFileLoc, {encoding: 'utf8'});
  const proxiedExt = path.extname(proxiedFileLoc);
  const proxiedUrl = proxiedFileLoc.substr(buildDirectoryLoc.length);
  const proxyCode = wrapEsmProxyResponse({
    url: proxiedUrl,
    code: proxiedCode,
    ext: proxiedExt,
    config,
  });
  const proxyFileLoc = proxiedFileLoc + '.proxy.js';
  await fs.writeFile(proxyFileLoc, proxyCode, {encoding: 'utf8'});
}
```

wrapEsmProxyResponse 是一个生成 proxy 模块的方法，目前只处理包括 JSON、image 和其他类型的文件，对于其他类型（包括了图片），就是非常简单的导出 url：

```js
return `export default ${JSON.stringify(url)};`;
```

所以，对于 CSS 与图片，由于浏览器模块规范均不支持该类型，所以都会转换为 JS 模块，这块 snowpack 和 webpack 实现很类似



#####  HMR（热更新）

<img src="https://qiniu-image.qtshe.com/21608446310286.jpg" style="zoom:60%;float:left;" />

仔细看 wrapEsmProxyResponse 方法，会发现对于 CSS “模块”，它除了有注入 CSS 的功能代码外，还多着这么几行：

```js
import * as __SNOWPACK_HMR_API__ from '/${buildOptions.metaDir}/hmr.js'
import.meta.hot = __SNOWPACK_HMR_API__.createHotContext(import.meta.url)
import.meta.hot.accept()
import.meta.hot.dispose(() => {
  document.head.removeChild(styleEl);
})
```

这些代码就是用来实现热更新的，也就是 HMR（Hot Module Replacement）。它使得当一个模块更新时，应用会在前端自动替换该模块，而不需要 reload 整个页面。这对于依赖状态构建的单页应用开发非常友好

import.meta 是一个包含模块元信息的对象，例如模块自身的 url 就可以在这里面取到。而 HMR 其实和 import.meta 没太大关系，snowpack 只是借用这块地方存储了 HMR 相关功能对象。所以不必过分纠结于它

再来仔细看看上面这段 HMR 的功能代码，下面这段对比一下

```js
import _ from 'lodash'
import printMe from './print.js'
function component() {
  const element = document.createElement('div')
  const btn = document.createElement('button')
  element.innerHTML = _.join(['Hello', 'webpack'], ' ')
  btn.innerHTML = 'Click me and check the console!'
  btn.onclick = printMe
  element.appendChild(btn)
  return element
}

document.body.appendChild(component())

+ + if (module.hot) {
+   module.hot.accept('./print.js', function() {
+     console.log('Accepting the updated printMe module!')
+     printMe()
+   })
+ }
```

上面的代码取自 webpack 官网上 HMR 功能的使用说明，可见，snowpack 站在“巨人”的肩膀上，沿袭了 webpack 的 API，其原理也及其相似。网上关于 webpack HMR 的讲解文档很多，这里就不细说了，基本的实现原理就是：

- snowpack 进行构建，并 watch 源码
- 在 snowpack 服务端与前端应用间建立 websocket 连接
- 当源码变动时，重新构建，完成后通过 websocket 将模块信息（id/url）推送给前端应用
- 前端应用监听到这个消息后，根据模块信息加载模块
- 同时，触发该模块之前注册的回调事件，这个在以上代码中就是传入 accept 和 dispose 中的方法

因此，wrapEsmProxyResponse 里构造出的这段代码

```js
import.meta.hot.dispose(() => {
  document.head.removeChild(styleEl)
})
```

其实就是表示，当该 CSS 更新并要被替换时，需要移除之前注入的样式。而执行顺序是：远程模块 --> 加载完毕 --> 执行旧模块的 accept 回调 --> 执行旧模块的 dispose 回调

snowpack 中 HMR 前端核心代码放在了 [assets/hmr.js](https://link.zhihu.com/?target=https%3A//github.com/pikapkg/snowpack/blob/v2.5.1/assets/hmr.js)。代码也非常简短，其中值得一提的是，不像 webpack 使用向页面添加 script 标签来加载新模块，snowpack 直接使用了原生的 dynamic import 来加载新模块：

```js
const [module, ...depModules] = await Promise.all([
  import(id + `?mtime=${updateID}`),
  ...deps.map((d) => import(d + `?mtime=${updateID}`)),
])
```

也是秉承了使用浏览器原生 JavaScript Modules 能力的理念



##### 环境变量

通过环境来判断是否关闭调试功能是一个非常常见的需求

```js
if (process.env.NODE_ENV === 'production') {
  disableDebug()
}
```

snowpack 中也实现了环境变量的功能。从使用文档上来看，可以在模块中的 import.meta.env 上取到变量。像下面这么使用：

```js
if (import.meta.env.NODE_ENV === 'production') {
  disableDebug()
}
```

以 build 的源码为例，看看环境变量是如何被注入进去的，在代码生成的阶段上，通过 [wrapImportMeta 方法的调用](https://zhuanlan.zhihu.com/p/149351900/https%3C/code%3E://github.com/pikapkg/snowpack/blob/v2.5.1/src/commands/build.ts#L346)生成了新的代码段

```js
code = wrapImportMeta({code, env: true, hmr: false, config})
```

从源码看看经过 wrapImportMeta 处理后的代码和之前有什么区别

```js
export function wrapImportMeta({
  code,
  hmr,
  env,
  config: {buildOptions},
}: {
  code: string;
  hmr: boolean;
  env: boolean;
  config: SnowpackConfig;
}) {
  if (!code.includes('import.meta')) {
    return code;
  }
  return (
    (hmr
      ? `import * as  __SNOWPACK_HMR__ from '/${buildOptions.metaDir}/hmr.js';\nimport.meta.hot = __SNOWPACK_HMR__.createHotContext(import.meta.url);\n`
      : ``) +
    (env
      ? `import __SNOWPACK_ENV__ from '/${buildOptions.metaDir}/env.js';\nimport.meta.env = __SNOWPACK_ENV__;\n`
      : ``) +
    '\n' +
    code
  );
}
```

对于包含 import.meta 调用的代码，snowpack 都会在里面注入对 env.js 模块的导入，并将导入值赋在 import.meta.env 上，因此构建后的代码会变为：

```js
+ import __SNOWPACK_ENV__ from '/__snowpack__/env.js';
+ import.meta.env = __SNOWPACK_ENV__;

if (import.meta.env.NODE_ENV === 'production') {
  disableDebug();
}
```

如果是在开发环境下，还会加上 env.js 的 HMR。而 env.js 的内容也很简单，就是直接将 env 中的键值作为对象的键值，通过 export default 导出

默认情况下 env.js 只包含 MODE 和 NODE_ENV 两个值，你可以通过 @snowpack/plugin-dotenv 插件来直接读取 .env 相关文件



##### CSS Modules 的支持

CSS 的模块化一直是一个难题，其一个重要的目的就是做 CSS 样式的隔离。常用的解决方案包括：

- 使用 BEM 这样的命名方式
- 使用 webpack 提供的 CSS Module 功能
- 使用 styled components 这样的 CSS in JS 方案
- shadow dom 的方案

snowpack 也提供了类似 webpack 中的 CSS Modules 功能

```js
import styles from './index.module.css';
function render() {
  return <div className={styles.main}>Hello world!</div>;
}
```

而在 snowpack 中启用 CSS Module 必须要以 .module.css 结尾，只有这样才会将文件特殊处理：

```js
if (spec.endsWith('.module.css')) {
  const resolvedUrl = path.resolve(path.dirname(outPath), spec)
  allCssModules.add(resolvedUrl)
  spec = spec.replace('.module.css', '.css.module.js')
}
```

而所有 CSS Module 都会经过 wrapCssModuleResponse 方法的包装，其主要作用就是将生成的唯一 class 名的 token 注入到文件内，并作为 default 导出：

```js
_cssModuleLoader = _cssModuleLoader || new (require('css-modules-loader-core'))();
const {injectableSource, exportTokens} = await _cssModuleLoader.load(code, url, undefined, () => {
  throw new Error('Imports in CSS Modules are not yet supported.');
});
return `
  ……
  export let code = ${JSON.stringify(injectableSource)};
  let json = ${JSON.stringify(exportTokens)};
  export default json;
  ……
`;
```

这里我将 HMR 和样式注入的代码省去了，只保留了 CSS Module 功能的部分。可以看到，它其实是借力了 css-modules-loader-core 来实现的 CSS Module 中 token 生成这一核心能力
对于导出的默认对象，键为 CSS 源码中的 classname，而值则是构建后实际的 classname



##### 性能问题

雅虎性能优化 35 条军规，其中就提到了通过合并文件来减少请求数。这既是因为 TCP 的慢启动特点，也是因为浏览器的并发限制。而伴随这前端富应用需求的增多，前端页面再也不是手工引入几个 script 脚本就可以了。同时，浏览器中 JS 原生的模块化能力缺失也让算是火上浇油，到后来再加上 npm 的加持，打包工具呼之欲出。webpack 也是那个时代走过来的产物

随着近年来 HTTP/2 的普及，5G 的发展落地，浏览器中 JS 模块化的不断发展，这个合并请求的“真理”也许值得我们再重新审视一下。去年 PHILIP WALTON 在博客上发的「Using Native JavaScript Modules in Production Today」就推荐可以在生产环境中尝试使用浏览器原生的 JS 模块功能

「Using Native JavaScript Modules in Production Today」 这片文章提到，根据之前的测试，非打包代码的性能较打包代码要差很多。但该实验有偏差，同时随着近期的优化，非打包的性能也有了很大提升。其中推荐的实践方式和 snowpack 对 node_modules 的处理基本如出一辙。保证了加载不会超过 100 个模块和 5 层的深度

同时，由于业务技术形态的原因，我所在的业务线经历了一次构建工具迁移，对于模块的处理上也用了类似的策略：业务代码模块不合并，只打包 node_modules 中的模块，都走 HTTP/2。但是没有使用原生模块功能，只是模块的分布状态与 snowpack 和该文中提到的类似。从上线后的性能数据来看，性能并未下降。当然，由于并非使用原生模块功能来加载依赖，所以并不全完相同。但也算有些参考价值





#### 总结

snowpack 的一大特点是快 —— 全量构建快，增量构建也快。因为不需要打包，所以它不需要像 webpack 那样构筑一个巨大的依赖图谱，并根据依赖关系进行各种合并、拆分计算。snowpack 的增量构建基本就是改动一个文件就处理这个文件即可，模块之间算是“松散”的耦合

而 webpack 还有一大痛点就是“外部“依赖的处理，“外部”依赖是指：

- 模块 A 运行时对 B 是有依赖关系
- 但是不希望在 A 构建阶段把 B 也拿来一起构建

这时候 B 就像是“外部”依赖。在之前典型的一个解决方式就是 external，当然还可以通过使用前端加载器加载 UMD、AMD 包。或者更进一步，在 webpack 5 中使用 Module Federation 来实现。这一需求的可能场景就是微前端。各个前端微服务如果要统一一起构建，必然会随着项目的膨胀构建越来越慢，所以独立构建，动态加载运行的需求也就出现了

