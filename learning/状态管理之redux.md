## 背景

> 本节内容相对粗浅，更多具体详细内容及具体实现需要到官网了解：http://cn.redux.js.org/

上周分享了状态管理中Flux出现的背景及功能原理，结论可如下：

1. Flux的做法
   在Flux中，Store只有get方法，没有set方法，因此View只能通过get方法获取Store的状态，无法直接去修改状态，如果View要修改Store状态的话，只有派发一个action对象给Dispatcher
2. 优点
   Flux最大的优点就是纠正MVC框架中无法禁止Model与View通信的缺点，实现更严格的数据流控制，即“单向数据流”的管理方式
3. 缺点：
   * store之间的依赖：如果两个Store之间有逻辑依赖关系，就必须用上Dispatcher的waitFor函数，需要Token标识
   * 难以进行服务器端渲染
   * Store混杂了逻辑与状态，当我们需要动态替换一个Store的逻辑时，只能把这个Store整体替换掉，那也就无法保持Store中存储的状态

可以看出Flux还是有些不足，而起Flux更注重理论，即其官网描述其只是一种开发模式，并非一个框架，续之，在Flux后，为了更好的实现MVC，Redux模式出现

<br >

<br >

## Redux

#### 简介

不同于 Flux ，Redux 不再有 dispatcher 的概念（Store已经集成了dispatch方法）。其次它依赖纯函数来替代事件处理器（即原来Flux中Dispatcher.register((action) 注册逻辑处理这块），这个纯函数叫做Reducer。另外使用到了一个新概念 context ，在React 组件间，数据是通过 props 属性由上向下（由父及子）进行传递的，当遇到多个层级多个组件间共享一个props，这种树形的由上而下的传参方式就显得过于繁琐，context 便很巧妙的解决了这个问题，参数只需从树顶点设置一次，便可在其所有枝节点都能共享到

<br >

#### 基本概念

我们把Flux看做一个框架的理念的话，Redux是Flux的一种实现
Flux的基本原则是“单向数据流”，Redux在此基础上强调三个基本原则：

![image-20200228203140066](https://qiniu-app.qtshe.com/image-20200228203140066.png)

* 唯一数据源
* 保持状态只读
* 数据改变只能通过纯函数完成

1. 唯一数据源
   唯一数据源指的是应用的状态数据应该只存储在唯一的一个Store上

2. 保持状态只读
   保持状态只读，就是说不能去直接修改状态，要修改Store的状态，必须要通过派发一个action对象完成。这点可能比较费解，因为驱动用户界面更改的是状态，如果状态都是只读的不能修改，怎么可能引起用户界面的变化呢？当然，要驱动用户界面渲染，就要改变应用的状态，但是改变状态的方法不是去修改状态上的值，而是创建一个新的状态对象返回给Redux，由Redux完成新的状态的组装

3. 数据改变只能通过纯函数完成
   这里所说的纯函数就是把Reducer，Redux这个名字的前三个字母Red代表的就是Reducer，其实Redux名字的含义就是Reducer+Flux
   Reducer不是一个Redux特定的术语，而是一个计算机科学中的通用概念。以javascript为例，数组类型就有reduce函数，接受的参数就是一个reducer，reduce做的事情就是把数组所有元素依次做“规约”，对每个元素都调用一次参数reducer，通过reducer函数完成规约所有元素的功能
   在Redux中，每个reducer函数如下：
   * reducer（state，action）
     第一个参数state是当前的状态，第二个参数action是接收到的action对象，而reducer函数要做的事情，就是根据state和action的值产生一个新的对象返回，注意reducer必须是一个纯函数，也就是说函数的返回结果必须完全由参数state和action决定，而且不产生任何副作用，也不能修改参数state和action对象

<br >

#### 流程

![image-20200228210701015](https://qiniu-app.qtshe.com/image-20200228210701015.png)

1. store通过reducer创建了初始状态
2. view通过store.getState()获取到了store中保存的state挂载在了自己的状态上
3. 用户产生了操作，调用了actions 的方法
4. actions的方法被调用，创建了带有标示性信息的action
5. actions将action通过调用store.dispatch方法发送到了reducer中
6. reducer接收到action并根据标识信息判断之后返回了新的state
7. store的state被reducer更改为新state的时候，store.subscribe方法里的回调函数会执行，此时就可以通知view去重新获取state

**store action view的基本关系就是**

* store -> view  : view基于store的数据展示相应的界面，view监听store的数据变化，数据变化触发界面变化

* view -> action : 用户在view上的交互产生影响数据的action（当然其他因素，比如网络变化也会产生action）

* action -> reducer -> store : action传递给store之后，首先是交给reducer处理数据变化，然后将变化后的数据保存到store（之后就是store->view store又引起了view变化）

所以它是一个单向的数据流动循环 store -> view -> action -> store

<br >

#### 关键词解析

* Store 就是保存数据的地方，你可以把它看成一个容器，整个应用只能有一个 Store
  Redux 提供createStore这个函数，用来生成 Store

* Store对象包含所有数据，如果想得到某个时点的数据，就要对 Store 生成快照，这种时点的数据集合，就叫做 State
* 当前时刻的 State，可以通过store.getState()拿到
* store.subscribe()
  Store 允许使用store.subscribe方法设置监听函数，一旦 State 发生变化，就自动执行这个函数
* stroe的实现提供了三个方法
  * store.getState()
  * store.dispatch()
  * store.subscribe()
* Action
  State 的变化，会导致 View 的变化，但是用户接触不到 State，只能接触到 View，所以State 的变化必须是 View 导致的，Action 就是 View 发出的通知，表示 State 应该要发生变化了

* Action Creator
  View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator

* store.dispatch()
  store.dispatch()是 View 发出 Action 的唯一方法

* Reducer
  Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer

* 纯函数
  Reducer 函数最重要的特征是，它是一个纯函数，也就是说，只要是同样的输入，必定得到同样的输出

