#### setState的同步和异步

为什么使用setState

- 开发中我们并不能直接通过修改 state 的值来让界面发生更新

- - 因为我们修改了 state 之后, 希望 React 根据最新的 Stete 来重新渲染界面，但是这种方式的修改 React 并不知道数据发生了变化
  - React 并没有实现类似于 Vue2 中的 Object.defineProperty 或者 Vue3 中的Proxy的方式来监听数据的变化
  - 我们必须通过 setState 来告知 React 数据已经发生了变化

- 疑惑: 在组件中并没有实现 steState 方法,，为什么可以调用呢?

- - 原因很简单: setState方法是从 Component 中继承过来的

    <img src="https://qiniu-image.qtshe.com/1E4DB9D4C043ACF6.png" style="zoom:100%;float:left;border-radius: 8px;" />



##### setState异步更新

setState是异步更新的

<img src="https://qiniu-image.qtshe.com/61C41A195.png" style="zoom:100%;float:left;border-radius: 8px;" />

* 为什么setState设计为异步呢?
  * setState 设计为异步其实之前在 GitHub 上也有很多的讨论
  * React核心成员（Redux的作者）Dan Abramov也有对应的回复, 有兴趣的可以看一下
* 简单的总结: setState设计为异步, 可以显著的提高性能
  * 如果每次调用 setState 都进行一次更新, 那么意味着 render 函数会被频繁的调用界面重新渲染, 这样的效率是很低的
  * 最好的方法是获取到多个更新, 之后进行批量更新
* 如果同步更新了 state, 但还没有执行 render 函数, 那么state和props不能保持同步
  * state和props不能保持一致性, 会在开发中产生很多的问题



##### 如何获取异步的结果

如何获取 setState 异步更新state后的值?

* 方式一: setState的回调

  setState接收两个参数: 第二个参数是回调函数(callback), 这个回调函数会在state更新后执行

  <img src="https://qiniu-image.qtshe.com/735FC085B6FCCC.png" style="zoom:100%;float:left;border-radius: 8px;" />

* 方式二: componentDidUpdate生命周期函数

  <img src="https://qiniu-image.qtshe.com/BED2FDA3E83.png" style="zoom:100%;float:left;border-radius: 8px;" />



##### setState一定是异步的吗?

其实可以分成两种情况

* 在组件生命周期或React合成事件中, setState是异步的
* 在setTimeou或原生DOM事件中, setState是同步的

验证如下

* 在setTimeout中的更新 —> 同步更新

  <img src="https://qiniu-image.qtshe.com/7A23DF89CA4.png" style="zoom:100%;float:left;border-radius: 8px;" />

* 在原生DOM事件 —> 同步更新

  <img src="https://qiniu-image.qtshe.com/27BF99353663.png" style="zoom:81%;float:left;border-radius: 8px;" />



##### 源码

<img src="https://qiniu-image.qtshe.com/4958365A7C1.png" style="zoom:47%;float:left;border-radius: 8px;" />





#### setState的合并

##### 数据的合并

通过setState去修改message，是不会对其他 state 中的数据产生影响的

源码中其实是有对 原对象 和 新对象 进行合并的

<img src="https://qiniu-image.qtshe.com/25F570F32C90.png" style="zoom:43%;float:left;border-radius: 8px;" />



##### 多个state的合并

* 当我们的多次调用了 setState, 只会生效最后一次state

  <img src="https://qiniu-image.qtshe.com/9476711757DE08E.png" style="zoom:70%;float:left;border-radius: 8px;" />



* setState合并时进行累加: 给setState传递函数, 使用前一次state中的值

  <img src="https://qiniu-image.qtshe.com/DCBCC3876017E.png" style="zoom:100%;float:left;border-radius: 8px;" />





#### React 更新机制

我们在前面已经学习React的渲染流程

<img src="https://qiniu-image.qtshe.com/5CEE8CD317DB.png" style="zoom:100%;float:left;border-radius: 8px;" />

那么 React 的更新流程呢?

<img src="https://qiniu-image.qtshe.com/51FB654594F.png" style="zoom:100%;float:left;border-radius: 8px;" />



##### React 更新流程

* React在 props 或 state 发生改变时，会调用 React 的 render 方法，会创建一颗不同的树

* React需要基于这两颗不同的树之间的差别来判断如何有效的更新UI

* 如果一棵树参考另外一棵树进行完全比较更新, 那么即使是最先进的算法, 该算法的复杂程度为 O(n 3 ^3 3)，其中 n 是树中元素的数量

  如果在 React 中使用了该算法, 那么展示 1000 个元素所需要执行的计算量将在十亿的量级范围

* 这个开销太过昂贵了, React的更新性能会变得非常低效

* 于是，React对这个算法进行了优化，将其优化成了O(n)，如何优化的呢？

  * 同层节点之间相互比较，不会跨节点比较
  * 不同类型的节点，产生不同的树结构
  * 开发中，可以通过key来指定哪些节点在不同的渲染下保持稳定

  <img src="https://qiniu-image.qtshe.com/F60B121BA12A31C.png" style="zoom:80%;float:left;border-radius: 8px;" />



##### 优化场景

1. 当节点为不同的元素，React会拆卸原有的树，并且建立起新的树

   * 当一个元素从 <a> 变成 <img>，从 <Article> 变成 <Comment>，或从 <button> 变成 <div> 都会触发一个完整的重建流程

   * 当卸载一棵树时，对应的DOM节点也会被销毁，组件实例将执行 componentWillUnmount() 方法

   * 当建立一棵新的树时，对应的 DOM 节点会被创建以及插入到 DOM 中，组件实例将执行 componentWillMount() 方法，紧接着 componentDidMount() 方法

   比如下面的代码更改

   React 会销毁 Counter 组件并且重新装载一个新的组件，而不会对Counter进行复用

   <img src="https://qiniu-image.qtshe.com/53BBB08320.png" style="zoom:100%;float:left;border-radius: 8px;" />



2. 对比同一类型的元素

   当比对两个相同类型的 React 元素时，React 会保留 DOM 节点，仅对比更新有改变的属性

   * 通过比对这两个元素，React知道只需要修改 DOM 元素上的 className 属性

   比如下面的代码更改

   <img src="https://qiniu-image.qtshe.com/5C16BAF1641.png" style="zoom:100%;float:left;border-radius: 8px;" />

   比如下面的代码更改

   * 当更新 style 属性时，React 仅更新有所改变的属性
   * 通过比对这两个元素，React 知道只需要修改 DOM 元素上的 color 样式，无需修改 fontWeight

   <img src="https://qiniu-image.qtshe.com/5A97306EBC1A5.png" style="zoom:100%;float:left;border-radius: 8px;" />

   如果是同类型的组件元素

   * 组件会保持不变，React会更新该组件的props，并且调用componentWillReceiveProps() 和 componentWillUpdate() 方法
   * 下一步，调用 render() 方法，diff 算法将在之前的结果以及新的结果中进行递归



3. 对子节点进行递归

   在默认条件下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation

   我们来看一下在最后插入一条数据的情况

   <img src="https://qiniu-image.qtshe.com/737DBFDE82B7D.png" style="zoom:100%;float:left;border-radius: 8px;" />

   面两个比较是完全相同的，所以不会产生mutation

   最后一个比较，产生一个mutation，将其插入到新的DOM树中即可

   但是如果我们是在前面插入一条数据

   <img src="https://qiniu-image.qtshe.com/91E342CD0173.png" style="zoom:100%;float:left;border-radius: 8px;" />

   React会对每一个子元素产生一个mutation，而不是保持 <li>星际穿越</li> 和 <li>盗梦空间</li>的不变

   这种低效的比较方式会带来一定的性能问题





#### React 性能优化

##### key的优化

我们在前面遍历列表时，总是会提示一个警告，让我们加入一个key属性

<img src="https://qiniu-image.qtshe.com/FE085AB474D4.png" style="zoom:100%;float:left;border-radius: 8px;" />

场景一：在最后位置插入数据

* 这种情况，有无key意义并不大

场景二：在前面插入数据

* 这种做法，在没有 key 的情况下，所有的<li>都需要进行修改

场景二：在下面案例: 当子元素 (这里的`li`元素) 拥有 `key` 时

* React 使用 key 来匹配原有树上的子元素以及最新树上的子元素：
* 下面这种场景下, key为 111 和 222 的元素仅仅进行位移，不需要进行任何的修改
* 将key为 333 的元素插入到最前面的位置即可

> key的注意事项
>
> * key应该是唯一的
> * key不要使用随机数（随机数在下一次render时，会重新生成一个数字）
> * 使用index作为key，对性能是没有优化的



##### render函数被调用

我们使用之前的一个嵌套案例，在App中，我们增加了一个计数器的代码，当点击 +1 时，会重新调用 App 的 render 函数

而当 App 的 render函数被调用时，所有的子组件的 render 函数都会被重新调用

<img src="https://qiniu-image.qtshe.com/7A49D5F66F.png" style="zoom:100%;float:left;border-radius: 8px;" />

那么，可以思考一下，在以后的开发中，我们只要是修改 了App中的数据，所有的子组件都需要重新render，进行 diff 算法，性能必然是很低的，事实上，很多的组件没有必须要重新render，它们调用 render 应该有一个前提，就是依赖的数据(state、 props) 发生改变时，再调用自己的render方法

如何来控制 render 方法是否被调用呢？



##### shouldComponentUpdate

React给我们提供了一个生命周期方法 shouldComponentUpdate（很多时候，我们简称为SCU），这个方法接受参数，并且需要有返回值；主要作用是：控制当前类组件对象是否调用render方法

该方法有两个参数

* nextProps修改之后, 最新的 porps属性
* nextState 修改之后, 最新的 state 属性

该方法返回值是一个 booolan 类型

* 值为true, 那么就需要调用 render 方法
* 值为false, 那么不需要调用 render 方法

比如我们在App中增加一个message属性

* JSX中并没有依赖这个message, 那么它的改变不应该引起重新渲染
* 但是通过setState修改 state 中的值, 所以最后 render 方法还是被重新调用了

```js
// 决定当前类组件对象是否调用render方法
// 参数一: 最新的props
// 参数二: 最新的state
shouldComponentUpdate(nextProps, nextState) {
  // 默认是: return true
  // 不需要在页面上渲染则不调用render函数
  return false
}
```



##### PureComponent

如果所有的类，我们都需要手动来实现 shouldComponentUpdate， 那么会给我们开发者增加非常多的工作量，我们设想一下在shouldComponentUpdate中的各种判断目的是什么？props 或者 state 中数据是否发生了改变，来决定shouldComponentUpdate返回 true 或 false

事实上 React 已经考虑到了这一点, 所以 React 已经默认帮我们实现好了, 如何实现呢？

* 将 class 继承自 PureComponent
* 内部会进行浅层对比最新的 state 和 porps，如果组件内没有依赖 porps或state 将不会调用render
* 解决的问题：比如某些子组件没有依赖父组件的state或props，但却调用了render函数

<img src="https://qiniu-image.qtshe.com/61A8E25E66983.png" style="zoom:60%;float:left;border-radius: 8px;" />

<img src="https://qiniu-image.qtshe.com/FA6F9FFD9DA.png" style="zoom:60%;float:left;border-radius: 8px;" />



##### shallowEqual方法

这个方法中，调用 !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)，这个 shallowEqual 就是进行浅层比较

<img src="https://qiniu-image.qtshe.com/A1375CB6F1FFA.png" style="zoom:60%;float:left;border-radius: 8px;" />





#### 高阶组件memo

函数式组件如何解决render: 在没有依赖 state 或 props 但却重新渲染 render 问题

我们需要使用一个高阶组件memo

* 将之前的Header、Banner、ProductList都通过 memo 函数进行一层包裹

* Footer没有使用 memo 函数进行包裹；

* 最终的效果是，当counter发生改变时，Header、Banner、ProductList的函数不会重新执行，而 Footer 的函数会被重新执行

<img src="https://qiniu-image.qtshe.com/00F4DCF900047.png" style="zoom:40%;float:left;border-radius: 8px;" />





#### React知识点总结脑图

<img src="https://qiniu-image.qtshe.com/0C06D6B46C4E.png" style="zoom:100%;float:left;" />



 style="zoom:100%;float:left;border-radius: 8px;"



































