#### 前言

前面在做H5埋点打点工具的时候，过程中发现每个步骤都离不开 babel 及相关生态的工具，但是一直以来都没有系统的了解过babel，借此机会系统性学习一波

当聊到 Babel 的作用，基本第一反应都是：用来实现 API polyfill，事实上，Babel 作为前端工程化的基石，作用远不止这些

作为一个庞大的家族，`Babel`生态中有很多概念，比如：`preset`、`plugin`、`runtime`等，然而往往对其理解也止步于`webpack`的`babel-loader`配置

接下来计划从 `Babel` 及其相关生态的作用及具体效果入手，逐步深入学习 `Babel` 核心原理





#### 简介

Babel 是一个用于 JavaScript 的通用多用途编译器，使用 Babel 可以使用（或创建）下一代 的JavaScript，以及下一代 JavaScript 工具

作为一门语言，JavaScript 不断发展，带来了很多新的规范和建议，使用 Babel 可以让你在这些新的规范和建议全面普及之前就提前使用它们

Babel 通过将最新标准的 JavaScript 代码编译为已经在目前可以工作的代码来实现上一段提到的内容。这个过程被称为 “**源代码到源代码**” 的编译，这也被成为 **“转换”**

例如，Babel 可以将最新的 ES2015 的箭头函数语法从

```js
const square = n => n * n
```

转换成下面的内容

```js
const square = function square(n) {
  return n * n;
};
```

然而，Babel 可以胜任更多的工作，因为Babel 支持语法扩展，例如 React 的 JSX 语法或者静态类型检查的 Flow 语法

更近一步，在 Babel 中一切皆插件，而每个人都可以充分利用 Babel 的强大能力来创建属于自己的插件。且 Babel 被组织成几个核心的模块，允许用户利用这些模块来构建下一代 JavaScript 工具链

许多人也是这样去做的，Babel 的生态系统正在茁长的成长。在这本 Babel 手册中，我将讲解 Babel 内建的一些工具以及社区里的一些拥有的工具





####  Babel 模块介绍

因为 JavaScript 社区没有标准的构建工具，框架或平台等，Babel 官方性与其他所有的主要工具进行了集成。无论是来自 Gulp、Browserify，或者是 Ember、Meteor，亦或是 Webpack 等，无论你的启动工具是什么，Babel 都存在一些官方性的集成



##### babel-cli

Babel的CLI是从命令行使用Babel编译文件的简单方法，这里先进行全局安装，以学习基础知识

```js
$ npm install --global babel-cli
```

编译一个文件

```js
$ babel my-file.js
```

这会将编译后的输出直接转储到终端中，要将其写入文件，可以指定 `--out-file` 或 `-o`

```js
$ babel example.js --out-file compiled.js
# or
$ babel example.js -o compiled.js
```

如果想将整个目录编译成一个新目录，可以使用 `--out-dir` 或 `-d` 来完成

```js
$ babel src --out-dir lib
# or
$ babel src -d lib
```





#### 在项目中运行Babel CLI

虽然可以在计算机上全局安装Babel CLI，但最好逐个项目在本地安装它

这有两个主要原因

- 同一台计算机上的不同项目可能取决于Babel的不同版本，从而允许一次更新一个版本
- 这意味着对工作的环境没有隐式依赖。使项目更加可移植且易于设置

可以通过运行以下命令在本地安装Babel CLI

```js
$ npm install --save-dev babel-cli
```

的 `package.json` 中添加，使用命令进行构建

```json
  {
    "name": "my-project",
    "scripts": {
      "build": "babel src -d lib"
    }
  }
```

终端运行

```js
npm run build
```



##### babel-register

运行Babel的下一个最常见的方法是通过 `babel-register` 。通过此选项，仅需要文件即可运行 Babel，这可能会更好地与设置集成

注意这并非供生产使用。部署以这种方式编译的代码被认为是不好的做法。最好在部署之前提前进行编译。但是，这对于构建脚本或在本地运行的其他事情非常有效

首先在项目中创建一个 `index.js` 文件

```js
console.log("Hello world!");
```

如果使用 `node index.js` 来运行它，那么 Babel 不会编译它，因此需要先设置 `babel-register` 

首先安装 `babel-register`

```js
$ npm install --save-dev babel-register
```

接下来，在项目中创建一个 `register.js` 文件，并编写以下代码：

```js
require("babel-register")
require("./index.js")
```

这是在 Node 的模块系统中注册 Babel 并开始编译每个 `require` 的文件

现在可以使用 `node egister.js` 代替运行 `node index.js` 

```js
$ node register.js
```

注意：不能在要编译的文件中注册 Babel。在 Babel 有机会编译文件之前，Node 正在执行文件

```js
require("babel-register")

// not compiled
console.log("Hello world!")
```



##### babel-node

如果只是通过 `node` CLI 运行某些代码，则集成 Babel 的最简单方法可能是使用 `babel-node` CLI，这在很大程度上只是对 `node` CLI 的替代

注意这并非供生产使用，部署以这种方式编译的代码被认为是不好的做法。最好在部署之前提前进行编译。但是，这对于构建脚本或在本地运行的其他事情非常有效

首先，请确保已安装 `babel-cli`

```js
$ npm install --save-dev babel-cli
```

然后将运行 `node` 的任何位置替换为 `babel-node`

```json
  {
    "scripts": {
      "script-name": "babel-node script.js"
    }
  }
```



##### babel-core

如果出于某种原因需要在代码中使用 Babel，则可以使用 `babel-core` 软件包本身

首先安装 `babel-core`

```js
$ npm install babel-core
```

```js
var babel = require("babel-core")
```

如果具有 JavaScript 字符串，则可以直接使用 `babel.transform` 对其进行编译

```js
babel.transform("code();", options)
// => { code, map, ast }
```

如果使用文件，则可以使用异步api：

```js
babel.transformFile("filename.js", options, function(err, result) {
  result // => { code, map, ast }
})
```

如果出于什么原因已经拥有Babel AST，则可以直接从AST转换

```js
babel.transformFromAst(ast, code, options)
// => { code, map, ast }
```





#### 配置 Babel

上述功能仅运行 Babel 似乎除了将 JavaScript 文件从一个位置复制到另一个位置之外没有执行任何其他操作，这是因为尚未告诉 Babel 该做什么事情，由于Babel是通用编译器，它以多种不同的方式使用，因此默认情况下它不会执行任何操作

可以通过安装 **plugins** 或 **presets** （plugins 组）为Babel提供操作说明



##### .babelrc

在开始告诉 Babel 怎么做之前，需要创建一个配置文件，需要做的就是在项目的根目录下创建一个 `.babelrc` 文件，从这样开始：

```js
{
  "presets": [],
  "plugins": []
}
```

该文件是配置 Babel 以执行所需操作的方式，虽然还可以通过其他方式将选项传递给 Babel，但 `.babelrc` 文件是约定俗成的，也是最好的方法



##### babel-preset-es2015

首先告诉 Babel 将 ES2015（JavaScript标准的最新版本，也称为ES6）编译为ES5（当今大多数JavaScript环境中可用的版本）

通过安装“ es2015” Babel预设来做到这一点（当然目前浏览器支持了绝大部分 ES2015 的特性了，这里是用作演示，使用形式是一致的）

```js
$ npm install --save-dev babel-preset-es2015
```

接下来，将修改 `.babelrc` 以包括该预设

```json
  {
    "presets": [
      "es2015"
    ],
    "plugins": []
  }
```



##### babel-preset-stage-x

JavaScript还提出了一些建议，这些建议正在通过TC39（ECMAScript标准背后的技术委员会）流程纳入标准

此过程分为5个阶段（0-4）。随着提案获得更大的吸引力，并更有可能被采纳为标准，它们经历了各个阶段，最终在阶段4被接纳为标准

这些以babel的形式捆绑为4种不同的预设：

- `babel-preset-stage-0`
- `babel-preset-stage-1`
- `babel-preset-stage-2`
- `babel-preset-stage-3`

注意没有阶段 4 的 `preset` ，因为它只是上面的 es2015 预设

这些预设中的每个预设都需要用于后续阶段的预设。即 `babel-preset-stage-1` 需要 `babel-preset-stage-2` ，而 `babel-preset-stage-3` 也需要

安装 stage

```js
$ npm install --save-dev babel-preset-stage-2
```

然后可以将其添加到 `.babelrc` 配置中

```json
  {
    "presets": [
      "es2015",
      "react",
      "stage-2"
    ],
    "plugins": []
  }
```





#### 执行 Babel 生成的代码

##### babel-polyfill

几乎所有未来 JavaScript 语法都可以使用 Babel 进行编译，但 API 并非如此

例如，以下代码具有需要编译的箭头函数功能：

```js
function addAll() {
  return Array.from(arguments).reduce((a, b) => a + b);
}
```

在编译之后会变成如下这样：

```js

function addAll() {
  return Array.from(arguments).reduce(function(a, b) {
    return a + b;
  });
}
```

但是，由于 `Array.from` 并非在每个JavaScript环境中都存在，因此在编译之后它仍然无法使用：

```js
Uncaught TypeError: Array.from is not a function
```

为了解决这个问题，使用一种叫做 **Polyfill**的东西，简而言之，Polyfill 是一段代码，该代码复制当前运行时中不存在的 API，允许在当前环境可用之前能提前使用 `Array.from` 等 API

Babel使用出色的 **core-js**作为其polyfill，以及定制的 **regenerator**运行时，以使生成器和异步函数正常工作

要包含 Babel polyfill，请首先使用npm安装它：

```js
$ npm install --save babel-polyfill
```

然后只需将 polyfill 包含在任何需要它的文件的顶部：

```js
import "babel-polyfill"
```



##### babel-runtime

为了实现 ECMAScript 规范的详细信息，Babel将使用 “**helper**” 方法来保持生成的代码干净

由于这些 “helper” 方法会变得很长，而且它们被添加到每个文件的顶部，因此可以将它们移动到 `require` 的单个“运行时”中

首先安装 `babel-plugin-transform-runtime` 和 `babel-runtime` ：

```js
$ npm install --save-dev babel-plugin-transform-runtime
$ npm install --save babel-runtime
```

然后更新 `.babelrc` 文件：

```json
  {
    "plugins": [
      "transform-es2015-classes"
    ]
  }
```

现在，Babel 将如下代码

```js
class Foo {
  method() {}
}
```

编译结果

```js
import _classCallCheck from "babel-runtime/helpers/classCallCheck";
import _createClass from "babel-runtime/helpers/createClass";

let Foo = function () {
  function Foo() {
    _classCallCheck(this, Foo);
  }

  _createClass(Foo, [{
    key: "method",
    value: function method() {}
  }]);

  return Foo;
}();
```

而不是将 `_classCallCheck` 和 `_createClass` helper 函数放在需要的每个文件中





#### Babel 和其他工具结合

一旦掌握了 Babel，Babel 便会很直接地进行设置，但是使用其他工具进行设置可能非常困难。接着尝试与其他项目紧密合作，以使体验尽可能轻松



##### 静态分析工具

较新的标准为语言带来了许多新语法，而静态分析工具才刚刚开始利用它

Linting，ESLint 是最受欢迎的 Lint 工具之一，因此 Babel 维护了官方的 **babel-eslint** 集成，首先安装 `eslint` 和 `babel-eslint`

```js
$ npm install --save-dev eslint babel-eslint
```

接下来，在项目中创建或使用现有的 `.eslintrc` 文件，并将解析器设置为 `babel-eslint`

```json
  {
    "parser": "babel-eslint",
    "rules": {
      ...
    }
  }
```

现在将一个 `lint` 任务添加到 npm `package.json` 脚本中：

```json
  {
    "name": "my-module",
    "scripts": {
      "lint": "eslint my-files.js"
    },
    "devDependencies": {
      "babel-eslint": "...",
      "eslint": "..."
    }
  }
```

然后只需运行任务即可完成所有设置

```js
$ npm run lint
```

