

#### 前景

前两天再看2020年前端发展趋势的时候，看到一个信息W3C官网宣布：第4种Web语言来了：WebAssembly



#### WebAssembly

据 W3C 官网上月宣布，WebAssembly 正式成为 World Wide Web Consortium (W3C) 的标准，加入到了 HTML、CSS 和 JavaScript 的行列，WebAssembly 也正式抵达了 1.0 版本，它已获得了主流浏览器 Firefox、Chrome、Safari 和 Edge 的支持

WebAssembly源于 Mozilla 发起的Asm.js项目，也被称为wasm，设计补充 JavaScript，其本地解码速度比 JS 解析快得多，让高性能的 Web 应用在浏览器上运行成为可能



#### 背景

看到新的一波洪流扑面而来，不禁让我想起了开始的前端三大主流框架Angular为啥逐渐不再耳闻，从而联想到了Angular被众人所认知的最大弊端脏检查机制，然而一下子又想不起来了Angular的脏检查机制原理是什么了，怎么实现的，为什么那么多人认为他的脏检查机制太消耗资源它却依旧能存活，而且不断的处于迭代中

基于上面的问题进行了一波学习了解



#### Angular

了解过Angular的都知道我们常说的Angularjs是指Angular 1，而Angular指的是从Angular 2及以后的迭代版本，现在 Angular 已经快发展到了 Angular9，谷歌仍然在维护 AngularJS，由此可以看出Angular并没有短期退出舞台的想法

如下对一些专业名词作简单介绍，便于理清发展流程

##### MVVM 数据双向绑定

2005年，微软的架构师 John Gossman 推出了 MVVM 模式，其核心部分是 DataBinding 机制。顾名思义，其功能就是将 Model 的数据绑定到 View 层，甚至是将 View 层控件的变换绑定到Model的双向绑定

但是 Model 是对现实场景的建模，但并非对UI界面的建模，所以 Model 往往并不能直接与 View 进行绑定，所以需要添加一个中介，对View层进行建模，即 ViewModel

MVVM真正的创举在于对视图建模的思想，特别是当下的移动时代，对于开发有着丰富界面交互体验的开发者来说，ViewModel似乎更契合业务场景。同时，强大的DataBinding机制也大大减少了工作量

##### Angularjs

Angularjs 诞生于2009年，由Misko Hevery 等人创建，后为Google所收购。是一款优秀的前端JS框架，已经被用于Google的多款产品当中。AngularJS是一个基于MVC处理模式，实现了MVVM数据双向绑定的用于开发动态Web项目的框架，以其数据和展现分离、MVVM、MVC、DI等强大的特性活跃于前端开发市场，AngularJS有着诸多特性，最为核心的是：MVC、模块化、自动化双向数据绑定、依赖注入等等，其中在前端开发常用的有：数据双向绑定、基本模板指令、自定义指令、表单验证、路由操作、依赖注入、过滤器、内置服务、自定义服务、组件、模块。是前端敏捷开发使用的主流的必须掌握的框架之一

作为首款成熟的 MVVM 前端框架，Angularjs 可以说是试吃第一口双向数据绑定苹果的先行者，设计思想被后来者多方借鉴，自有其独到之处，不过接下来主要分析一下其脏检查机制

##### Angular 的数据响应机制（脏检查原理）

熟悉 vue 的人都知道，vue 的数据响应机制是通过发布订阅模式去监听实现的，并且在 vue 2 和 vue 3 中实现的方式还不相同，vue 2 基于 Object.defineProperty 实现，而 vue 3 却是通过 proxy 实现

那么，在代码层面，Angularjs 及 Angular 是怎么做到监听数据变动然后更新界面的呢？答案是，他们根本不监听数据的变动，而是在恰当的时机从 rootScope 开始遍历所有 scope，检查它们上面的属性值是否有变化，如果有变化，就用一个变量 dirty 记录为 true，再次进行遍历，如此往复，直到某一个遍历完成时，这些 scope 的属性值都没有变化时，结束遍历。由于使用了一个 dirty 变量作为记录，因此被称为脏检查机制

简而言之脏值检测的基本原理是存储旧数值，并在进行检测时，把当前时刻的新值和旧值比对，若相等则没有变化，反之则检测到变化，需要更新视图

Angular 把页面切分成若干个Component（组件），组成一棵组件树，进入脏值检测后，从根组件自顶向下进行检测，Angular 有两种策略：Default和OnPush，它们配置在组件上，决定脏值检测过程中不同的行为

这里面有三个问题

1. “恰当的时机”是什么时候
2. 如何做到知道属性值是否有变化
3. 这个遍历循环是怎么实现的

要解决这三个问题，我们需要深入了解angular的 ，watch，apply，digest

##### $watch绑定要检查的值







比如说目前最火的 `React` ，它采用的是`虚拟DOM`，简单来说就是将页面上的`DOM`和`JS`里面的`虚拟DOM`进行对比，然后将不一样的地方渲染到页面上去，这个思想就是`AngularJS`的脏检查机制，只不过`AngularJS`是检查的数据，`React`是检查的`DOM`而已。

MVVM最早由微软提出来，它借鉴了桌面应用程序的MVC思想，在前端页面中，把Model用纯JavaScript对象表示，View负责显示，两者做到了最大限度的分离。也就是我们常说的，前后分离，真正在这里得以实现