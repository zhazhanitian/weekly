## 前言

Vite 是一款新一代的前端构建工具，是由 Vue.js 的作者尤雨溪开发并推出的。它采用了一种全新的构建方式，以及优化了开发环境的性能和开发体验，成为了近年来备受关注的前端构建工具。本文将从为什么会出现 Vite 的背景、Vite 解决了哪些问题、与市场产品对比的优缺点、Vite 核心原理及相应的源码带注释分析、Vite 遗留的问题、Vite 延伸扩展的生态知识介绍等方面深入分析 Vite。

<br />

## 背景

传统的前端构建工具像 Webpack 和 Rollup 在实现热更新和代码分割等功能时，需要对整个应用进行打包和编译。这样的做法导致了开发环境的启动时间过长，而且修改代码后需要等待重新打包，开发效率较低。随着前端技术的不断发展，前端应用的规模和复杂度也不断增加，需要更快速和高效的开发体验。

因此，尤雨溪基于此需求，开发了 Vite，一款更快速、更轻量的前端构建工具。Vite 的设计初衷就是针对现代浏览器的 ES Modules 进行构建，从而提供更快的开发体验。Vite 采用了一种新的构建方式，利用了浏览器原生的模块系统，可以在开发环境下实现秒级热更新。

<br />

## 解决了哪些问题

#### 更快的启动时间

Vite 利用了浏览器原生的模块系统，不需要在开发环境下进行打包和编译，因此启动速度非常快。开发者只需要运行 `vite` 命令，就可以启动一个开发服务器，几乎可以实现秒级启动。这个特性大大提高了开发效率。

#### 秒级热更新

传统的前端构建工具在实现热更新时，需要重新打包和编译整个应用，耗费时间较长。Vite 利用了浏览器原生的模块系统，通过 HMR（Hot Module Replacement）技术实现秒级热更新。当开发者修改代码时，只需要更新相应的模块，不需要重新加载整个应用，从而实现瞬间更新。

#### 支持多种开发场景

Vite 支持多种开发场景，可以作为一个轻量的静态文件服务器，也可以作为一个现代前端框架（如 Vue.js 和 React）的开发环境。Vite 支持 Vue.js、React、Preact、LitElement、Stencil 等多种框架，也可以支持 TypeScript、Sass、Less 等多种预处理器。

### 优化了开发体验

Vite 支持在浏览器中直接调试源代码，而不需要在 IDE 中进行调试。开发者可以直接在浏览器的控制台中打断点，查看变量的值，从而更加方便地进行调试。此外，Vite 还提供了一种基于浏览器的开发方式，开发者可以利用浏览器中的 DevTools 工具来进行开发和调试。

<br />

## 与市场产品对比的优缺点

#### 优点

- 启动速度快：由于 Vite 不需要在开发环境下进行打包和编译，因此启动速度非常快。
- 热更新快速：Vite 利用了浏览器原生的模块系统，通过 HMR 技术实现秒级热更新。
- 支持多种框架：Vite 支持 Vue.js、React、Preact、LitElement、Stencil 等多种框架，且可以支持 TypeScript、Sass、Less 等多种预处理器。
- 优化了开发体验：Vite 支持在浏览器中直接调试源代码，而不需要在 IDE 中进行调试，从而提高了开发体验。
- 可扩展性强：Vite 可以通过插件扩展其功能，也可以通过自定义 Rollup 配置来进行更高级的定制。

#### 缺点

- Vite 还比较年轻，相较于 Webpack 等成熟的构建工具，生态还不够完善。
- 在处理一些复杂的场景（例如大规模应用、多页面应用等）时，可能需要额外的配置和优化。

<br />

## 核心原理及相应的源码分析

Vite 的核心原理是利用浏览器原生的模块系统，以及基于 Rollup 的打包器。下面我们将对 Vite 的核心源码进行简单的注释分析。

#### vite 代码结构

- `packages/vite`: Vite 的核心代码。
- `packages/playground`: Vite 的官方示例，用于演示 Vite 的使用方法。
- `packages/create-app`: Vite 的官方脚手架，用于快速创建 Vite 应用。

#### vite 核心代码结构

- `src/node`: Node.js 端的代码，主要用于启动开发服务器和进行打包编译。
- `src/client`: 浏览器端的代码，主要用于处理模块热更新、调试和加载器等功能。
- `src/utils`: 工具函数。
- `src/server`:服务器代码，包括中间件和路由等。

#### vite 启动流程

Vite 的启动流程可以分为以下几个步骤：

1. 初始化配置：读取用户传入的配置文件，并进行合并。
2. 创建服务：使用 Koa 创建一个 HTTP 服务器，用于启动开发服务器和打包服务器。
3. 开发服务器：通过中间件将开发服务器挂载到 HTTP 服务器上。
4. 构建打包器：创建 Rollup 实例，用于打包编译代码。
5. 打包服务器：通过中间件将打包服务器挂载到 HTTP 服务器上。
6. 启动服务：启动 HTTP 服务器，监听请求并响应。

#### vite 热更新流程

Vite 的热更新流程可以分为以下几个步骤：

1. 监听模块：在浏览器端监听模块的变化，使用 WebSocket 技术实现。
2. 生成补丁：当模块发生变化时，生成对应的补丁。
3. 通知更新：将补丁发送给客户端，触发客户端的热更新流程。
4. 客户端更新：客户端接收到补丁后，更新相应的模块。

#### vite 加载器流程

Vite 的加载器流程可以分为以下几个步骤：

1. 检查模块类型：判断模块类型，包括 JS、CSS、HTML 等。
2. 转换代码：根据模块类型进行对应的转换，例如将 SASS 转换为 CSS。
3. 处理依赖：解析模块的依赖关系，生成依赖图。
4. 输出模块：将模块输出到浏览器端。

#### vite 插件扩展

Vite 支持插件扩展，开发者可以通过编写插件来扩展 Vite 的功能。插件可以包括：

- 增强加载器：例如增强 CSS 加载器，支持 PostCSS、CSS Modules 等功能。
- 增强 Rollup 配置：例如增加自定义的 Rollup 插件、修改 Rollup 配置等功能。
- 增强服务器：例如增强 HTTP 服务器、增加中间件等功能。

#### vite 源码分析

Vite 的核心原理是采用了 ES6 模块原生的浏览器解析能力，通过服务端动态构建的方式来实现模块化开发。在开发模式下，Vite 会启动一个开发服务器，利用浏览器的原生 ES6 模块加载能力，在浏览器中直接运行代码，不需要打包和构建，从而实现了非常快速的开发和热更新体验。在生产模式下，Vite 会使用 Rollup 进行打包构建，生成高效的静态资源。

下面对 Vite 核心代码做简单分析和注释：

```js
// src/node/server.js

const Koa = require('koa');
const { createServer: createViteServer } = require('vite');

async function createServer() {
  const app = new Koa();

  // 通过 createViteServer 创建 Vite 服务器实例
  const vite = await createViteServer({
    server: { middlewareMode: 'html' }
  });

  // 注册 Vite 中间件到 Koa
  app.use(vite.middlewares);

  // 启动服务器
  app.listen(3000);
}

createServer();

```

上述代码是启动 Vite 服务器的主要代码，其中 `createViteServer` 函数会返回一个 Vite 服务器实例。这个实例中包含了 Vite 的核心功能，如编译器、打包器、中间件等。在代码中，通过 `vite.middlewares` 将 Vite 的中间件注册到 Koa 服务器中，从而实现 Vite 的开发服务器。

接下来是 `createViteServer` 函数中的一段核心代码，用于处理模块依赖和热更新：

```js
// src/node/server/serverPluginModuleRewrite.js

import { parse } from 'url';
import { readBody } from '../utils';
import { resolver } from 'vite';

export function moduleRewritePlugin() {
  return {
    name: 'vite:module-rewrite',
    async transformIndexHtml(html, ctx) {
      const { pathname } = parse(ctx.url);
      if (pathname !== '/') {
        return;
      }
      const { template, root } = ctx;
      const resolved = await resolver.request(
        '/src/main.js',
        `/${root}/index.html`
      );
      const code = await readBody(resolved);
      return {
        html: template.replace(`<!--APP-->`, `<script>${code}</script>`),
      };
    },
  };
}

```

上述代码是 Vite 中的一个插件，用于处理模块依赖和热更新。在开发模式下，Vite 会拦截浏览器请求，解析出请求的模块依赖，并通过动态引入和热更新的方式将依赖注入到浏览器中。这个插件就是实现这一功能的核心代码。

在这段代码中，首先通过 `resolver.request` 函数解析出 `/src/main.js` 模块的绝对路径，然后读取模块代码，最后将代码注入到模板中返回。这样就实现了在浏览器中动态加载模块的功能。

除了上述的插件，Vite 还包含了很多其他的插件，用于处理各种不同的需求，如 CSS 处理、图片处理、压缩代码等。这些插件在 Vite 的开发和构建过程中发挥着重要的作用。

下面是 Vite 中一个用于处理 CSS 的插件代码：

```js
// src/node/server/serverPluginCss.ts

import path from 'path';
import { readBody, cachedRead } from '../utils';
import { Plugin } from './Plugin';
import { ResolvedConfig } from '../config';
import { TransformResult } from 'rollup';
import { parse, compileTemplate } from '@vue/compiler-sfc';

export const cssPlugin: Plugin = ({
  app,
  watcher,
  resolver,
  config,
  pluginContext,
}) => {
  app.use(async (ctx, next) => {
    // 如果是 CSS 请求，解析 CSS 的依赖关系
    if (ctx.path.endsWith('.css')) {
      const id = resolver.requestToFile(ctx.path);
      const css = await cachedRead(id);
      const { descriptor } = parse(css);
      const styles = descriptor.styles;
      const imports = descriptor.cssVars.length
        ? `import { applyCSSVariables } from "${pluginContext.server.publicPath}/@vite/css-var-declaration"\n`
        : ``;

      // 处理 CSS 中的 URL，将其改为绝对路径
      const cssWithAbsoluteUrl = styles.reduce((acc, s) => {
        s.content = s.content.replace(/url\((.+?)\)/g, (full, rawUrl) => {
          if (/^(data:|http)/.test(rawUrl)) {
            // do not transform data-uri or http(s) urls
            return full;
          }
          const resolved = resolver.request(
            path.posix.resolve(id, '..', rawUrl)
          );
          return `url(${resolved})`;
        });
        return acc + s.content + `\n`;
      }, '');

      // 编译 CSS，将其中的 CSS 变量和类名改为哈希值
      const { code } = compileTemplate({
        source: cssWithAbsoluteUrl,
        id: id,
        scoped: !!descriptor.styles.some((s) => s.scoped),
        filename: id,
        preprocessLang: 'css',
        compilerOptions: {
          scopeId: descriptor.scopeId,
          module: 'esm',
        },
      });

      // 返回编译后的 CSS 代码
      ctx.type = 'text/css';
      ctx.body = `${imports}${code}`;
    } else {
      await next();
    }
  });
};

```

这段代码是 Vite 中用于处理 CSS 的插件代码。在开发模式下，如果浏览器请求了 CSS 文件，这个插件会通过解析 CSS 的依赖关系，处理其中的 URL，将其改为绝对路径，最终将编译后的 CSS 代码返回给浏览器。在处理 CSS 时，Vite 还会将其中的 CSS 变量和类名改为哈希值，以防止命名冲突。

通过对 Vite 的核心代码和插件代码的分析，可以看出 Vite的核心思想是将代码拆分成小块，通过动态引入的方式实现快速加载，从而提高开发效率和用户体验。在实现这一思想的过程中，Vite 还使用了很多创新的技术，如 ES 模块和浏览器原生的模块加载，使得代码的加载速度更快，开发体验更加流畅。

<br />

## Vite 遗留的问题

虽然 Vite 在开发体验和性能方面都有很大的优势，但它仍然存在一些问题。

首先，Vite 对于大型应用的支持还不够完善。由于 Vite 在开发过程中使用了动态引入的方式，因此对于大型应用而言，它可能需要大量的网络请求，导致加载时间变慢。此外，在使用 Vite 进行构建时，如果应用中包含了大量的第三方库，Vite 的构建时间也会变得很长，影响开发效率。

其次，Vite 的插件生态还比较薄弱。虽然 Vite 内置了很多有用的插件，但是如果我们需要使用一些特定的插件，可能需要自己编写或寻找其他开发者编写的插件。在这一点上，Vite 还需要更多的社区支持和贡献。

<br />

<br />

## Vite 延伸扩展的生态知识介绍

Vite 作为一个新兴的工具，在社区中已经逐渐得到了认可和推广。除了 Vite 自身的功能外，它还可以与其他工具和框架结合使用，构建出更加完善和高效的开发体验。

下面介绍一些与 Vite 相关的工具和框架：

#### Vue.js

Vue.js 是一个流行的前端框架，与 Vite 配合使用可以实现更加高效的开发体验。Vue.js 3.0 与 Vite 相互兼容，Vue.js 的开发团队也在积极地推广 Vite。

#### React

React 是另一个流行的前端框架，与 Vite 结合使用可以实现更加高效的开发体验。React 开发团队也在推广 Vite，推荐开发者使用 Vite 进行 React 项目的开发。

#### Rollup

Rollup 是一个 JavaScript 模块打包器，它可以将多个模块打包成一个文件，提高应用的性能和加载速度。Vite 也是基于 Rollup 开发的，因此 Vite 的插件开发也可以参考 Rollup 的插件开发方式。

#### Snowpack

Snowpack 是一个类似于 Vite 的工具，它也可以实现快速的开发和构建体验。与 Vite 不同的是，Snowpack 使用的是 ESM（ES Modules）本地开发体验，可以更加轻量级，更适合中小型项目的开发。

#### Preact

Preact 是一个轻量级的 React 替代品，它可以提供类似于 React 的 API，并且拥有更小的代码体积和更快的渲染速度。Preact 与 Vite 结合使用可以实现更快的开发和构建速度。

#### Windi CSS

Windi CSS 是一个类似于 Tailwind CSS 的 CSS 框架，它可以帮助开发者更快地编写样式，并且可以与 Vite 结合使用，实现更快的开发和构建速度。

<br />

## 总结

Vite 是一个快速、简单、现代化的构建工具，它采用了新的构建方式，可以在开发过程中提供更快的构建速度、更好的开发体验，并且支持插件扩展和生态扩展，为前端开发带来了更多的可能性。虽然 Vite 还存在一些问题，但它的出现已经让前端开发变得更加轻松和快捷，相信在未来的发展中，Vite 会越来越完善和成熟，成为前端开发不可或缺的工具之一。









