## 前言

当下最流行的两个前端框架都存在 Virtual DOM， 类似“使用 Virtual DOM 有什么优势？” 的面试题也十分频繁，但一直没有太在意。前几天做H5埋点工具时，需要将VUE页面转换为asdTree，在解决问题的调研过程中，看到了一些 Virtual DOM 相关的知识点，但是都是知其然而不知其所以然， 对许多问题都缺乏思考，基此对一些问题进行总结

<br >

<br >

## 常规知识点

### 虚拟DOM同样也是操作DOM，为啥说它快

1. 虚拟DOM不会进行排版与重绘操作
2. 虚拟DOM进行频繁修改，然后一次性比较并修改真实DOM中需要改的部分（注意！），最后并在真实DOM中进行排版与重绘，减少过多DOM节点排版与重绘损耗
3. 真实DOM频繁排版与重绘的效率是相当低的
4. 虚拟DOM有效降低大面积（真实DOM节点）的重绘与排版，因为最终与真实DOM比较差异，可以只渲染局部

<br >

### Virtual Dom 的优势在哪里？

1. 具备跨平台的优势，由于 Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Weex、Node 等
2. 操作 DOM 慢，js运行效率高。我们可以将DOM对比操作放在JS层，提高效率。因为DOM操作的执行速度远不如Javascript的运算速度快，因此，把大量的DOM操作搬运到Javascript中，运用patching算法来计算出真正需要更新的节点，最大限度地减少DOM操作，从而显著提高性能
3. 提升渲染性能 Virtual DOM的优势不在于单次的操作，而是在大量、频繁的数据更新下，能够对视图进行合理、高效的更新

<br >

### Virtual Dom的优势

1. 不会立即进行排版与重绘
2. VDOM频繁修改，一次性比较并修改真实DOM中需要修改的部分，最后在真实DOM中进行重排 重绘，减少过多DOM节点重排重绘的性能消耗
3. VDOM有效降低大面积真实DOM的重绘与重排，与真实DOM比较差异，进行局部渲染

<br >

### 小结

上面是从 Google 搜索到的三个平台中的分析摘选，总结下来大概四点

1. 操作 DOM 太慢，操作 Virtual DOM 对象快
2. 使用 Virtual DOM 可以避免频繁操作 DOM ，能有效减少回流和重绘次数（如果有的话）
3. 有 diff 算法，可以减少没必要的 DOM 操作
4. 跨平台优势，只要有 JS 引擎就能运行在任何地方（Weex/SSR）

<br >

<br >

## Virtual DOM 快？

有人认为操作 Virtual DOM 速度很快？Virtual DOM 是一个用来描述 DOM（注意，并不一定一一对应）的 Javascript 对象，Javascript 操作 Javascript 对象自然是快的

但 Virtual DOM 仍然需要调用 DOM API 去生成真实的 DOM ，而你其实是可以直接调用它们的，所有就有一个很有意思结论，正数再小也不可能比零还小——Virtual DOM 很快，但这并不是它的优势，因你本可以选择不使用 Virtual DOM。除了速度不是优势，Virtual DOM 还有个最大的问题——额外的内存占用，以 Vue 的 Virtual DOM 对象为例，100W 个空的 Virtual DOM(Vue) 会占用 110M 内存

<br >

### 测试代码

```js
let creatVNode = function(type) {
  return {
    __v_isVNode: true,
    SKIP: true,
    type,
    props: null,
    key: null,
    ref: null,
    scopeId: 0,
    children: null,
    component: null,
    suspense: null,
    ssContent: null,
    ssFallback: null,
    dirs: null,
    transition: null,
    el: null,
    anchor: null,
    target: null,
    targetAnchor: null,
    staticCount: 0,
    shapeFlag: 0,
    patchFlag: 0,
    dynamicProps: null,
    dynamicChildren: null,
    appContext: null
  }
}

let counts = 1000000
let list = []
let start = performance.now()
// 创建 VNode(Vue)
// 10000: 1120k
for (let i = 0; i < counts; i++) {
  list.push(creatVNode('div'))
}

// 创建 DOM
// 10000: 320k
// for (let i = 0; i < counts; i++) {
//   list.push(document.createElement('div'))
// }

console.log(performance.now() - start)
```

<br >

### 内存占用截图

![](https://qiniu-image.qtshe.com/F812125B0184DD.png)

<br >

令人意外的是 100W 个空的 DOM 对象只占用 45M 内存，不清楚在 DOM 属性明显更多的情况下 Chrome 是如何优化的，或则是 Dev Tools 存在问题，这一点待后续调研

这里 Virtual DOM 不但执行快没有用，还增加了大量的内存消耗，所以我们说它快自然是有问题的，因为没有 Virtual DOM 时更快

<br >

<br >

## Virtual DOM 减少回流和重绘？

也有人认为 Virtual DOM 能减少页面的 relayout 和 repaint ，通常有两个原因来支撑这个观点

1. DOM 操作会先改变 Virtual DOM ，所以一些无效该变（比如把文本 A 修改为 B ，然后再修改为 A）就不会调用 DOM API ，也就不会导致浏览做无效的回流和重绘
2. DOM 操作会先改变 Virtual DOM ，最终由 Virtual DOM 调用 `patch` 方法批量操作 DOM ，批量操作就不会导致过程中出现无意义的回流和重绘

<br >

### 无效回流与重绘

第一个观点看着很有道理，但有个问题很难解释：浏览器的 UI 线程在什么时候去执行回流和重绘？要知道现代浏览器在设计上为了避免高复杂度，Javascript 线程和 UI 线程是互斥的，即如果浏览器要在 Javascript 执行期间触发 relayout/repaint 则必须先挂起 Javascript 线程，这是个连我都觉得蠢的设计，显然不会出现在各大浏览器身上

事实上也确实如此，无论在一次事件循环中调用多少次的 DOM API ，浏览器也只会触发一次回流与重绘（如果需要），并且如果多次调用并没有修改 DOM 状态，那么回流与重绘一次都不会发生

Timeline 截图（没有回流和重绘发生）

![](https://qiniu-image.qtshe.com/qweqwe2125B0184DD.png)

<br >

测试代码

```js
<body>
  <div class="app"></div>
  <script> let counts = 1000
    let $app = document.querySelector('.app')
    setTimeout(() => {
      for (let i = 0; i < counts; i++) {
        $app.innerHTML = 'aaaa'
        $app.style = 'margin-top: 100px'
        $app.innerHTML = ''
        $app.style = ''
      }
    }, 1000) </script>
</body>
```

<br >

### 无意义的回流与重绘

第二个观点是比较有意思的，虽然看了上面的分析，应该也知道它是错的，批量操作并不能减少回流与重绘，因为它们本身就只会触发一次。但这里还是要列出来证明一下，因为这是当下众多前端的一个固有认知，这里后续会进行深入和了解并和其他前端交流沟通看看，为啥大家认为批量操作会减少回流与重绘

批量操作并不能减少回流与重绘，原因也和上文一致，Javascript 是单线程且与 UI 线程互斥，所以直接放测试数据

<br >

Javascript 执行耗时（数据取3次平均值）

<img src="https://qiniu-image.qtshe.com/ioi2125B0184DD.png" style="zoom:110%;float:left;" />

<img src="https://qiniu-image.qtshe.com/bn2125B0184DD.png" style="zoom:110%;float:left;" />

<br >

Layout 耗时（数据取3次平均值）

<img src="https://qiniu-image.qtshe.com/mn2125B0184DD.png" style="zoom:111%;float:left;" />

<img src="https://qiniu-image.qtshe.com/jh2125B0184DD.png" style="zoom:110%;float:left;" />

<br >

测试代码

```vue
<body>
  <div class="app"></div>
  <script> let counts = 1000
    let $app = document.querySelector('.app')
    let start = performance.now()
    // 单独操作
    // for (let i = 0; i < counts; i++) {
    //   let node = document.createTextNode(`${i}, `)
    //   $app.append(node)
    // }

    // 批量操作
    let $tempContainer = document.createElement('div')
    for (let i = 0; i < counts; i++) {
      let node = document.createTextNode('node,')
      $tempContainer.append(node)
    }
    $app.append($tempContainer)

    console.log(performance.now() - start) </script>
</body>
```

可以看到的是，批量处理和单次处理再 Layout 期间耗时是几乎一致的，虽然在 script 执行阶段还是存在一定的性能优势（大概 30%），但大抵上只要你用好 DOM 操作，批量或不批量带来的性能影响是很小的（ 10W 次调用多损耗 27ms ）

这里又牵扯出一个问题，为什么在 script 执行阶段还是存在一定的性能差距？

<br >

<br >

## Virtual DOM 有 diff 算法？

严格来说 diff 算法和 Virtual DOM 是两个独立的东西，二者互相之间也没有充分必要的关联，比如 svelte 没有 Virtual DOM 也有其自己的 diff 算法

但由于前端框架存在 Virtual DOM 就总有 diff 算法，并且使用了 Virtual DOM 对 diff 算法也有两个助力

1. 得益于 Virtual DOM 的抽象能力，diff 算法更容易被实现和理解
2. 得益于 Virtual DOM Tree 总是在内存中， diff 算法功能可以更强大（比如组件移动，没有完整的 Tree 结构是不可能实现的）

diff 算法能减少 DOM API 调用，显然是存在设计和性能优势的，而由于 Virtual DOM 的存在，diff 算法可以更方便且更强大，所以可以认同这是 Virtual DOM 的优势，但不能用“Virtual DOM 有 diff 算法”这样的表述

<br >

<br >

## Virtual DOM 有跨平台优势？

上面提到的 svelte 没有 Virtual DOM ，但一样可以实现服务端渲染，这说明跨平台并不依赖于 Virtual DOM 

其实只要 Javascript 框架有实现平台 API 分发机制，就能在不同平台执行不同的渲染方法，即拥有跨平台能力。这个能力的根本，是 Javascript 代码能低代价地在各个平台运行（得利于浏览器在各个平台的普及和 NodeJS），也就是常说的 Javascript 的优势之一是跨平台。所以把跨平台当做 Virtual DOM 的优势，其实是不正确的，但我们或许应该去思考下他们为什么会这么认为

可能是这两个原因

1. Virtual DOM 的优势，可以在不接触真实 DOM 的情况下操作 DOM，并且性能更好

   在 Virutal DOM 上的改动，最终还是会调用平台 API 去操作真实的 DOM ，所以没有 Virtual DOM 只是相当于少了一个中间抽象层，并不影响跨平台能力有无。但还是需要明白，就目前的分析来看，这个抽象层对跨平台能力还是提供了相当大的方便（或者说助力）的

2. Virtual DOM 在 Vue 中很重要，Vue 本身就是一个围绕 Virtual DOM 创建起来的框架，脱离了 Virtual DOM 其设计思想必然会和当下迥乎不同

<br >

<br >

## 总结

本文从互联网上摘选了部分对开发者对 Virtual DOM 优点的认知，也从现实生活中了解到一些误解，总结为 Virtual DOM 的四个“优势”，并分别对这四个“优势”进行了单独分析或举证

最终识别了几个关于 Virtual DOM 优势误区

1. 操作 DOM 太慢，操作 Virtual DOM 对象快 ❌

   Virtual DOM 很快，但这并不是它的优势，因你本可以选择不使用 Virtual DOM 

2. 使用 Virtual DOM 可以避免频繁操作 DOM ，能有效减少回流和重绘次数 ❌

   无论你在一次事件循环中调用多少次的 DOM API ，浏览器也只会触发一次回流与重绘（如果需要），并且如果多次调用并没有修改 DOM 状态，那么回流与重绘一次都不会发生。批量操作也不能减少回流与重绘

3. Virtual DOM 有跨平台优势 ❌

   跨平台是 Javascript 的优势，与 Virtual DOM 无关

这里也提到了 Virtual DOM 真正的优点是其抽象能力和常驻内存的特性，让框架能更容易实现更强大的 diff 算法，缺点是增加了框架复杂度，也占用了更多的内存

