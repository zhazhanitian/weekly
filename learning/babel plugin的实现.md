#### babel plugin 概念

先看一个简单的babel plugin示例

```js
module.exports = function ({ types: t }) {
    const TRUE = t.unaryExpression("!", t.numericLiteral(0), true)
    const FALSE = t.unaryExpression("!", t.numericLiteral(1), true)
    return {
        visitor: {
            BooleanLiteral(path) {
                path.replaceWith(path.node.value ? TRUE : FALSE)
            }
        }
    }
}
```

这个plugin功效很简单，就做如下转换

```js
// 源代码
const x = true

// 转义后的的代码
const x = !0
```

就是把所有的bool类型的值转化成 !0 或者 !1，这是代码压缩的时候常见的一个使用技巧

接着来简单分析一下这个plugin，一个plugin就是一个function，入参就是babel对象，这里利用到了babel中types对象，来自于@babel/types这个库，然后操作path对象进行节点替换操作



##### path

path是一个对象，可以用path访问到当前节点、父节点，也可以去调用添加、更新、移动和删除节点有关的其他很多方法。举几个示例

```js
path是肯定会用到的一个对象。我们可以用过path访问到当前节点，父节点，也可以去调用添加、更新、移动和删除节点有关的其他很多方法。举几个示例
// 访问当前节点的属性，用path.node.property访问node的属性
path.node.node
path.node.left
// 直接改变当前节点的属性
path.node.name = "x"
// 当前节点父节点
path.parent
// 当前节点的父节点的path
path.parentPath
// 访问节点内部属性
path.get('left')
// 删除一个节点
 path.remove()
// 替换一个节点
path.replaceWith()
// 替换成多个节点
path.replaceWithMultiple()
// 插入兄弟节点
path.insertBefore()
path.insertAfter()
// 跳过子节点的遍历
path.skip()
// 完全跳过遍历
path.stop()
```



##### @babel/types

可以理解它为一个工具库，类似于 Lodash，里面封装了非常多的帮做方法，一般用处如下

- 检查节点 一般在类型前面加 is 就是判断是否该类型

  ```js
  // 判断当前节点的left节点是否是identifier类型
  if (t.isIdentifier(path.node.left)) {
    // ...
   }
  
  // 判断当前节点的left节点是否是identifer类型，并且name='n'
  if (t.isIdentifier(path.node.left, { name: "n" })) {
      // ...
  }
  
  // 上述判断等价于
  if (
      path.node.left != null &&
      path.node.left.type === "Identifier" &&
      path.node.left.name === "n"
    ) {
      // ...
    }
  ```



* 构建节点
  直接手写复杂的 AST 结构是不现实的，所以有了一些帮助方法去构建这些节点，示例：

  ```js
  
  // 调用binaryExpression和identifier的构建方法，生成ast
  t.binaryExpression("*", t.identifier("a"), t.identifier("b"))
  
  // 生成如下
  {
    type: "BinaryExpression",
    operator: "*",
    left: {
      type: "Identifier",
      name: "a"
    },
    right: {
      type: "Identifier",
      name: "b"
    }
  }
  
  // 最后经过AST反转回来如下
  a * b
  ```

  其中每一种节点都有自己的构造方法，都有自己特定的入参，详细请参考官方文档



##### scope

作用域的概念，每一个函数，每一个变量都有自己的作用域，在编写babel plugin的时候要特别小心，再改变或者添加代码的时候要注意不要破坏了原有的代码结构

用 path.scope 中的一些方法可以操作作用域，示例：

```js
// 检查变量n是否被绑定（是否在上下文已经有引用）
path.scope.hasBinding("n")

// 检查自己内部是否有引用n
path.scope.hasOwnBinding("n")

// 创建一个上下文中新的引用 生成类似于{ type: 'Identifier', name: '_n2' }
path.scope.generateUidIdentifier("n")

// 重命名当前的引用
path.scope.rename("n", "x")
```





#### babel解析器

接着看看 babel 解析器，看看 babel 是如何运作的，才能更好的知道 plugin 该怎么去实现

##### tiny-compiler 编译器

假如现在有一些新特性的语法，其中 add、subtract 是普通的函数名，需要转义到正常的 javascript 语法，以便让浏览器能够兼容的运行

```js
(add 2 2)
(subtract 4 2)
(add 2 (subtract 4 2))
```

要转义成如下

```js
add(2, 2)
subtract(4, 2)
add(2, subtract(4, 2))
```

编译器都分为三个步骤

- Parsing 解析
- Transformation 转义
- Code Generation 代码生成





#### Parsing 解析

Parsing 阶段分成两个子阶段

* Lexical Analysis 词法分析
* Syntactic Analysis语法分析，先写好我们要转化的代码

```js
// 这是我们要转化的code
(add 2 (subtract 4 2))
```



##### Lexical Analysis 词法分析

Lexical Analysis 词法分析可以理解为把代码拆分成最小的独立的语法单元，去描述每一个语法，可以是操作符，数字，标点符号等，最后生成token数组

```js
// 第一步，Lexical Analysis，转化成tokens类似如下
[
    { type: 'paren', value: '(' },
    { type: 'name', value: 'add' },
    { type: 'number', value: '2' },
    { type: 'paren', value: '(' },
    { type: 'name', value: 'subtract' },
    { type: 'number', value: '4' },
    { type: 'number', value: '2' },
    { type: 'paren', value: ')' },
    { type: 'paren', value: ')' },
]
```

实现如下

```js
function tokenizer(input) {
    let current = 0
    let tokens = []
    while (current < input.length) {
        let char = input[current]
        // 处理(
        if (char === '(') {
            tokens.push({
                type: 'paren',
                value: '('
            })
            current++
            continue
        }
        // 处理)
        if (char === ')') {
            tokens.push({
                type: 'paren',
                value: ')'
            })
            current++
            continue
        }
        // 处理空白字符
        let WHITESPACE = /\s/
        if (WHITESPACE.test(char)) {
            current++
            continue
        }
        // 处理数字
        let NUMBERS = /[0-9]/
        if (NUMBERS.test(char)) {
            let value = ''
            while (NUMBERS.test(char)) {
                value += char
                char = input[++current]
            }
            tokens.push({ type: 'number', value })
            continue
        }
        // 处理字符串
        if (char === '"') {
            let value = ''
            char = input[++current]
            while (char !== '"') {
                value += char
                char = input[++current]
            }
            char = input[++current]
            tokens.push({ type: 'string', value })
            continue
        }
        // 处理函数名
        let LETTERS = /[a-z]/i
        if (LETTERS.test(char)) {
            let value = ''
            while (LETTERS.test(char)) {
                value += char
                char = input[++current]
            }
            tokens.push({ type: 'name', value })
            continue
        }
        // 报错提示
        throw new TypeError('I dont know what this character is: ' + char)
    }
    return tokens
}
```



##### Syntactic Analysis 语法分析

Syntactic Analysis 语法分析就是根据上一步的tokens数组转化成语法之前的关系，这就是Abstract Syntax Tree,也就是我们常说的AST

```js
// 第二步，Syntactic Analysis，转化成AST类似如下
{
    type: 'Program',
    body: [{
        type: 'CallExpression',
        name: 'add',
        params: [{
            type: 'NumberLiteral',
            value: '2'
        }, {
            type: 'CallExpression',
            name: 'subtract',
            params: [{
                type: 'NumberLiteral',
                value: '4'
            }, {
                type: 'NumberLiteral',
                value: '2'
            }]
        }]
    }]
}
```

parser 转化成 AST 的实现如下

```js
function parser(tokens) {
    let current = 0
    function walk() {
        let token = tokens[current]
        // 处理数字
        if (token.type === 'number') {
            current++
            return {
                type: 'NumberLiteral',
                value: token.value
            }
        }
        // 处理字符串
        if (token.type === 'string') {
            current++
            return {
                type: 'StringLiteral',
                value: token.value
            }
        }
        // 处理括号表达式
        if (
            token.type === 'paren' &&
            token.value === '('
        ) {
            token = tokens[++current]
            let node = {
                type: 'CallExpression',
                name: token.value,
                params: []
            }
            token = tokens[++current]
            while (
                (token.type !== 'paren') ||
                (token.type === 'paren' && token.value !== ')')
            ) {
                node.params.push(walk())
                token = tokens[current]
            }
            current++
            return node
        }
        throw new TypeError(token.type)
    }
    let ast = {
        type: 'Program',
        body: []
    }
    while (current < tokens.length) {
        ast.body.push(walk())
    }
    return ast
}
```

从上述代码来看，跟阶段AST是根节点是 type= Program，body 是一个嵌套的 AST 数组结构。再单独处理了 number 和 string 类型之后，再递归的调用walk函数，以解决嵌套的括号表达式





#### Transformation 转义

##### traverser 遍历器

我们最终的目的肯定是想转化成我们想要的代码，那就得更改我们刚刚得到的AST结构

那怎么去改AST呢？直接去操作这个树结构肯定是不现实的，所以我们需要遍历这个AST，利用深度优先遍历的方法遍历这些节点，当遍历到某个节点时，再去调用这个节点对应的方法，再方法里面改变这些节点的值就轻而易举了

假如我们有这样的一个 visitor，就是上文说道的遍历时调用的方法

```js
const visitor = {
    NumberLiteral: {
        enter(node, parent) { },
        exit(node, parent) { }
    }
}
```

由于深度优先遍历的特性，我们遍历到一个节点时有 enter 和 exit 的概念，代表着遍历一些类似于 CallExpression 这样的节点时，这个语句，enter 表示开始解析，exit 表示解析完毕



然后有一个函数，接受 ast 和 vistor 作为参数，实现遍历，类似于

```js
traverse(ast, {
    CallExpression: {
        enter(node, parent) {
            // ...
        },
        exit(node, parent) {
            // ...
        }
    }
})
```

traverser实现如下

```js
function traverser(ast, visitor) {
    // 遍历一个数组节点
    function traverseArray(array, parent) {
        array.forEach(child => {
            traverseNode(child, parent)
        })
    }
    // 遍历节点
    function traverseNode(node, parent) {
        let methods = visitor[node.type]
        // 先执行enter方法
        if (methods && methods.enter) {
            methods.enter(node, parent)
        }
        switch (node.type) {
            // 一开始节点的类型是Program，去接着解析body字段
            case 'Program':
                traverseArray(node.body, node)
                break
            // 当节点类型是CallExpression，去解析params字段
            case 'CallExpression':
                traverseArray(node.params, node)
                break
            // 数字和字符串没有子节点，直接执行enter和exit就好
            case 'NumberLiteral':
            case 'StringLiteral':
                break
            // 容错处理
            default:
                throw new TypeError(node.type)
        }
        // 后执行exit方法
        if (methods && methods.exit) {
            methods.exit(node, parent)
        }
    }
    // 开始从根部遍历
    traverseNode(ast, null)
}
```



##### transformer 转换器

有了traverser遍历器后，就开始遍历吧，先看看前后两个 AST 的对比

<img src="https://qiniu-image.qtshe.com/9048F81CF8707.png" style="zoom:43%;float: left;" />

注意这里多了一种 ExpressionStatement 的type，以表示 subtract(4, 2) 这样的结构

遍历的过程就是把左侧 AST 转化成右侧 AST

```js
function transformer(ast) {
    let newAst = {
        type: 'Program',
        body: []
    }
    // 给节点一个_context,让遍历到子节点时可以push内容到parent._context中
    ast._context = newAst.body
    traverser(ast, {
        CallExpression: {
            enter(node, parent) {
                let expression = {
                    type: 'CallExpression',
                    callee: {
                        type: 'Identifier',
                        name: node.name
                    },
                    arguments: []
                }
                // 让子节点可以push自己到expression.arguments中
                node._context = expression.arguments
                // 如果父节点不是CallExpression，则外层包裹一层ExpressionStatement
                if (parent.type !== 'CallExpression') {
                    expression = {
                        type: 'ExpressionStatement',
                        expression: expression
                    }
                }
                parent._context.push(expression)
            }
        },
        NumberLiteral: {
            enter(node, parent) {
                parent._context.push({
                    type: 'NumberLiteral',
                    value: node.value
                })
            }
        },
        StringLiteral: {
            enter(node, parent) {
                parent._context.push({
                    type: 'StringLiteral',
                    value: node.value
                })
            }
        }
    })
    return newAst
}
```





#### CodeGeneration 代码生成

最后一个阶段就是用心生成的 AST 生成我们最后的代码了，也是生成 AST 的一个反过程

```js

function codeGenerator(node) {
    switch (node.type) {
        // 针对于Program，处理其中的body属性，依次再递归调用codeGenerator
        case 'Program':
            return node.body.map(codeGenerator).join('\n')
        // 针对于ExpressionStatement，处理其中的expression属性，再后面添加一个分号
        case 'ExpressionStatement':
            return ( codeGenerator(node.expression) + ';')
        // 针对于CallExpression，左侧处理callee，括号中处理arguments数组
        case 'CallExpression':
            return ( codeGenerator(node.callee) + '(' + node.arguments.map(codeGenerator) .join(', ') + ')' )
        // 直接返回name
        case 'Identifier':
            return node.name
        // 返回数字的value
        case 'NumberLiteral':
            return node.value
        // 字符串类型添加双引号
        case 'StringLiteral':
            return '"' + node.value + '"'
        // 容错处理
        default:
            throw new TypeError(node.type)
    }
}
```





#### plugin 实现

写一个自定义 plugin 步骤如下

- 这个plugin用来干嘛
- 源代码的AST
- 转换后代码的AST



##### plugin的目的

现在就做一个自定义的plugin，大家在应用写代码的时候可以通过webpack配置alias，比如说配置`@` -> `./src`，这样import的时候就直接从src目录下找所需要的代码了，这就是我们这个plugin的目的

我们有如下配置

```js
"alias": {
   "@": "./src"
}
```

源代码以及要转化的代码如下

```js
// ./src/index.js
import add from '@/common'  // -> import add from "./common"

// ./src/foo/test/index.js
import add from '@/common'  // ->  import add from "../../common"
```



##### 源码的AST展示如下

<img src="https://qiniu-image.qtshe.com/1CEB060582.png" style="zoom:43%;float:left;" />

那我们只需要找到 ImportDeclaration 节点中将 source 改成转换之后的代码是不是就可以了



##### 实现

```js
const localPath = require('path')
module.exports = function ({ types: t }) {
    return {
        visitor: {
            ImportDeclaration(path, state) {
                // 从state中拿到外界传进的参数，这里我们外界设置了alias
                const { alias } = state.opts
                if (!alias) return
                // 拿到当前文件的信息
                const { filename, root } = state.file.opts
                // 找到相对地址
                const relativePath = localPath.relative(root, localPath.dirname(filename))
                // 利用path获取当前节点中source的value，这里对应的就是 '@/common'了
                let importSource = path.node.source.value
                // 遍历我们的配置，进行关键字替换
                Object.keys(alias).forEach(key => {
                    const reg = new RegExp(`^${key}`)
                    if (reg.test(importSource)) {
                        importSource = importSource.replace(reg, alias[key])
                        importSource = localPath.relative(relativePath, importSource)
                    }
                })
                // 利用t.StringLiteral构建器构建一个StringLiteral类型的节点，赋值给source
                path.node.source = t.StringLiteral(importSource)
            }
        }
    }
}
```



##### 使用

回到babel的配置文件中来，这里我们用的是 babel.config.json

```js
{
    "plugins": [
        [
        // 这里使用本地的文件当做plugin，实际上可以把自己制作的plugin发布成npm包供大家使用
            "./plugin.js", 
        // 传配置到plugin的第二个参数state.opts中
            {
                "alias": {
                    "@": "./src"
                }
            }
        ]
    ]
}
```

这样一个plugin的流程就 over 了

