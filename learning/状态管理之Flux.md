## Flux是什么

> Flux is the application architecture that Facebook uses for building client-side web applications. It complements React's composable view components by utilizing a unidirectional data flow. It's more of a pattern rather than a formal framework, and you can start using Flux immediately without a lot of new code

这是Flux官方介绍，Flux是由Facebook官方提出的一套前端应用架构模式。它的核心概念是单向数据流。它不是具体的框架，而更像是一种软件开发模式

<br >

#### Flux模式的结构图

<img src="https://qiniu-app.qtshe.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_afd5f722-7e2d-445a-9c8a-f6eb6ee9510c.png" style="zoom:50%;" />

<br >

<br >

## Flux出现的背景

2014年，facebook提出了一个新的概念：Flux，旨在解决这些问题，其核心思想是“**组件化 + 单向数据流**”，在开始介绍之前，先说一说MV\*，应该都听说过开始的MVC，在这之后又衍生出了MVP和MVVM，这些都可以统称为MV\*。但是，随着前端代码复杂度的增加，人们发现越来越难以管理程序的状态，模块之间耦合严重，代码难以调试，因此很多人认为“前端MVC已死”

<br >

#### 网络神图

<img src="https://qiniu-app.qtshe.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_52e07466-9900-4260-9357-3fb8f266231b.png" alt="企业微信截图_52e07466-9900-4260-9357-3fb8f266231b" style="zoom:80%;" />

<br >

相对于MVC架构：Flux中的Store包含了应用的所有数据，Dispatcher替换了原来的Controller，当Action触发时，决定了Store如何更新。当Store变化后，View同时被更新，还可以生成一个由Dispatcher处理的Action。这确保了数据在系统组件间单向流动。当系统有多个Store和View时，仍可视为只有一个Store和一个View，因为数据只朝一个方向流动，并且不同的Store和View之间不会直接影响彼此

其实Flux并不是什么新鲜事物，其背后还是经典的MVC思想，但是实现方式上有所不同。Flux的核心是“组件化+单向数据流“，下面逐一进行介绍

<br >

<br >

## Flux vs MV\*

Flux被认为是MVC模式的一种替代方案。在Flux的文档中将其解释为“一种使用单向数据流而不是MVC”的应用架构模式，将Flux和MVC相比，需要理解下面三点：

将Flux和MVC相比，需要理解下面三点：

1. 在javascript的世界中，“MVC”意味着”MV*”
2. Flux并不会比MV*要简单，MVC不具有可扩展性
3. 相较于MV*，Flux模式中的代码更具有可预测性

<br >

#### MVC

![企业微信截图_b3dbe029-433f-4dd6-8bd3-a3969ce73e8e](https://qiniu-app.qtshe.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_b3dbe029-433f-4dd6-8bd3-a3969ce73e8e.png)

用户首先通过View发起交互，View调用Controller执行业务逻辑，Controller修改Model，然后View通过观察者模式检测到Model的变化，刷新界面显示

从这里可以看出，主要业务逻辑都在Controller中，Controller会变得很重

MVC比较明显的缺点：

- View依赖特定的Model，无法组件化
- View和Controller紧耦合，如果脱离Controller，View难以独立应用（功能太少）

<br >

#### MVP

![企业微信截图_fad8f1e7-b053-41ac-be17-e2296a38c3c4](https://qiniu-app.qtshe.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_fad8f1e7-b053-41ac-be17-e2296a38c3c4.png)

<br >

为了克服MVC的上述缺点，MVP应运而生。在MVP中，View和Model是没有直接联系的，所有操作都必须通过Presenter进行中转。View向Presenter发起调用请求，Presenter修改Model，Model修改完成后通知Presenter，Presenter再调用View的相关接口刷新界面，这样，View就不需要监听具体Model的变化了，只需要提供接口给Presenter调用就可以了

MVP具有以下优点：

- View可以组件化，不需要了解业务逻辑，只需提供接口给Presenter
- 便于测试：只需要给Presenter mock一个View，实现View的接口即可

<br >

#### MVVM

![企业微信截图_d9ad0d4b-2f80-44af-9170-b718b231e6f3](https://qiniu-app.qtshe.com/2f11a8c90s.png)

为了进一步解放生产力，把Presenter中调用View的接口同步数据变化的重复工作抽象出来，做成一个binder模块，这就变成了MVVM。开发者只需要指明绑定关系，binder模块会自动完成数据同步，这就是所谓的“**双向数据流**”，不管哪一端的数据发生变化，都会立即同步到另一端。实际上，Vue.js、Angular这些流行的前端框架都使用了双向数据流设计

双向数据流极大地简化了开发者的工作，但是诟病也随之而来。由于绑定的随意性，某个View对Model进行的修改有可能会对其他的View造成“连锁反应”，再加上各种异步回调，给代码调试造成了很大的困难，往往难以定位数据到底是被谁修改掉的。用专业一点的术语来讲，代码的“**可预测性**”非常差。因此，为了提高可预测性，很多人主张回归到“单向数据流”模式，其中的典型代表就是facebook的Flux框架

<br >

#### Flux工作流程说明

<img src="https://qiniu-app.qtshe.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_b8c6a5da-fd88-4725-9795-8e7d09aa4728.png" alt="企业微信截图_b8c6a5da-fd88-4725-9795-8e7d09aa4728" style="zoom:67%;" />

* Action，它是用来描述一个行为的对象，每个action里都包含了某个行为的相关信息。比如， {actionName: 'CREATE_POST', data: {'content': 'new stuff'}}
* Dispatcher，它是一个信息分发中心，它是action和store的连接中心。它可以使用dispatch方法执行一个action，并且可以用register方法注册回调，在回调中处理store中的数据
* Store，它是数据操作的唯一地方，当数据发生变化时，它可以使用emit方法向其它地方发送名为'change'的广播，告知它们store已经发生了变化
* View，视图层监听了'change'事件，一旦change事件被触发，视图层就会调用setState方法来更新相应的UI State

虽然Flux并不比MVC简单，但是Flux图表中的可预测性要大大高于MVC图表

Flux中的派发器确保了系统中一次只会有一个action流。如果一个action还没有处理完，那么这时再派发一个action将会触发一个错误：

这是使得代码可预测性提高的另一种方式。它促使开发者能够开发出让数据源之间的交互变得简单的代码

派发器也能让开发者指明回调函数执行的顺序，其中会使用`waitFor`方法来告诉回调函数依次执行

<br >

<br >

## Flux解析

#### 基本流程

1. 用户触发视图事件
2. 事件中通过调用（直接或者间接） dispatcher 实例的 dispatch 方法触发一个 action 事件
3. action 的触发会导致 store 中进行数据变更
4. 监听 store 中数据的 view 获得更新

<br >

#### 原则

1. 数据在 Flux 应用中的流向是单一方向的 Action -> Dispatcher -> Store -> View

2. Dispatcher 是一个中央集线器，管理所有Flux 应用中的数据流，它只是回调到 store 中的注册表，用来分配 action 到 store，当一个 action 被 dispatch 的时候，所有注册过这个 Dispatcher 的 store 都会收到通知，判断 action.type 之后触发相应的更新

   **Actions**

   * 有 action creater 创建，描述了触发 action 的内容

3. Store 中包含应用的状态和逻辑，类似于传统 MVC 架构中的 Model ，Store 将自己注册到 Dispatcher 并提供回调，在 Store 的注册回调中，一个基于action.type 的 switch 语句用于解释分析，并为 Store 内部的方法提供适当的钩子，钩子中对当前状态更改，更改后会触发广播事件，这个时候视图查询到状态更改，更新自己

   **stores**

   * 可以借助 flux/utils 的 ReduceStore 实现用于缓存数据

   - 对外提供可访问数据的方法
   - 响应 dispatcher 分发的 action
   - 只有在 dispatch 可以触发 data 的更新
   - 按照页面分离，也就是每个页面创建一个 Store

<br >

<br >

## Flux的难点

没有哪一种模式是完美无缺的，Flux也一样。一般来说，它有以下几个缺点：

- 代码编写更加模板化
- 移植现有代码比较困难
- 单元测试难以进行

为了处理数据流，在Flux应用种我们需要添加更多的文件和代码。和为Flux应用中已经存在的一个数据源添加代码相比，我们为应用添加一个新的数据源将是一件无比痛苦的事情。在将来我们或许可以使用生成器是的Flux代码的设施变得简单

尝试Flux最简单的方式是开启一个新项目。在新项目种，要将其他的部分移植到Flux架构将是一件非常有挑战的事情。运用从本文和Flux文档中学到的知识，你完全可以像Facebook和其他使用Flux的公司一样，将Flux架构运用到你的项目中

当你将现有项目移植到Flux架构时，你可以每次为Flux架构添加一个数据源。然而，在尝试使用Flux管理一块数据时，你需要考虑有多少组件会使用这块数据。如果这块数据在大部分的组件中都有被使用，那么将这块数据移植到Flux中将是一件复杂的工作

在Flux中，组件开始依赖ActionCreators以及Store，以及其他的依赖项目。这将使得编写单元测试异常复杂。如果你在应用种将Store的交互严格限制在顶级的”controller”上，那么你可以对子组件进行单元测试而无需担心Stores。如果你要测试那些需要发送Actions以及监[听Stores的组件，我们需要模拟一些Store种方法，或者模拟Actions和Stores用于接收数据的API

<br >

<br >

## 补充

* 简述Flux流程：当用户在View上发生交互行为时，Dispatcher使用dispatch方法触发一个action（它包含了actionType和要传递的数据），在整个程序的总调度中心Dispatcher里面注册了各种类型的action类型，在对应的类型中，store实现了“订阅-发布”功能对这个action进行响应，同时对数据做相应的处理。数据处理完成后，触发一个自定义的change事件。View层在初始化完成时注册这个change事件，当接收到store的广播后，view层响应这个事件并重新渲染UI界面

* Flux单向数据流，提供了可预测的状态，避免了多向数据流带来的混乱。但与些同时，Flux架构会增加我们的代码量。所以，如果我们的程序足够简单，并且组件之间没有共享数据，那么使用Flux只会给你徒增烦恼

* Flux架构的核心思想是单向数据流，它的核心流程是Dispatcher。Facebook有其官方的Dispatcher实现，即Flux-Dispatcher

