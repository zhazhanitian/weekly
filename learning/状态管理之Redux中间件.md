## 前言

Flux的出现，帮前端开发人员解决了一些问题的同时，基于 Flux 也有一些问题，Flux 当中通常有多个 Store， 分别监听比较麻烦， 跨 Store 操作还有同步顺序，挺烦的，然后 Redux 出来了，首先是很配合作者之前的 react-hot-loader 做热替换， 因为 Store 变成了一个， 概念上明确了很多， 然后提供了一些方案可以将数据注入到深度嵌套的组件当中，不需要一层一层重复传递回掉函数

但是，有这样一个问题？我们之前用的Redux都是在Action发出之后立即执行Reducer，计算出state，这是同步操作

我们知道redux的核心，就是控制和管理所有的数据输入输出，因此有了dispatch，由于dispatch是一个很纯的纯函数，就是单纯的派发action来更改数据，其功能简单且固定，假如现在产品经理A某有个需求，要求记录每次的dispatch记录，我们怎么办呢？

如果想异步操作怎么办？

单向数据流的情况下，组件之间数据如何通信？组件的公共数据应该放在哪里？

这里就需要用到Redux中间件来协助我们解决问题了

<br >

<br >

## 回顾redux

#### redux基本架构图

![](https://qiniu-app.qtshe.com/1583490655614.jpg)

<br >

#### dispatch函数

<img src="https://qiniu-app.qtshe.com/1583490771200.jpg" style="zoom:85%;" />

<br >

#### With Redux

<img src="https://qiniu-app.qtshe.com/D4E014F5-45D4-412D-8EC7-94919FE1CBEA.png" style="zoom:40%;" />

<br >

<br >

## 中间件的概念

redux是有流程的，那么，我们该把这个异步操作放在哪个环节比较合适呢

Reducer？纯函数只承担计算State功能，不适合其它功能

View？与State一一对应，可以看做是State的视觉层，也不适合承担其它功能

Action？它是一个对象，即存储动作的载体，只能被操作

其实，也只有dispatch能胜任此重任了，那么怎么在dispatch中添加其它操作呢

**中间件函数本质就是对dispatch方法的扩展，那么如何优雅的扩展dispatch方法？**

```javascript
let next = store.dispatch

store.dispatch = function(action) {
  console.log('老状态', store.getState())
  next(action)
  console.log('新状态', store.getState())
}
```

示例中可以看出，我们对store.dispatch重新进行了定义，在发送action的前后，做了打印

这是中间件的大致雏形，当然真正的中间件要比这么复杂多了

<br >

<br >

## 中间件洋葱模型

当用户调用dispatch时，首先执行mid1中间件，在mid1中调用next执行mid2中间件，依次执行，最终调用dispatch，dispatch调用结束后，依次执行剩余中间件代码

<img src="https://qiniu-app.qtshe.com/1583492847357.jpg" style="zoom:50%;" />

<br >

<br >

## Redux中间件机制概念

Redux本身就提供了非常强大的数据流管理功能，但这并不是它唯一的强大之处，它还提供了利用中间件来扩展自身功能，以满足用户的开发需求，首先我们来看中间件的定义

> It provides a third-party extension point between dispatching an action, and the moment it reaches
>  the reducer.

<br >

这是Dan Abramov 对middleware的描述。简单来讲，Redux middleware 提供了一个分类处理 action 的机会。在 middleware 中，我们可以检阅每一个流过的 action,并挑选出特定类型的 action 进行相应操作，以此来改变 action。这样说起来可能会有点抽象，我们直接来看图，这是在没有中间件情况下的 redux 的数据流:

<img src="https://qiniu-app.qtshe.com/1583492074215.jpg" style="zoom:55%;" />

<br >

上面是很典型的一次 redux 的数据流的过程，但在增加了 middleware 后，我们就可以在这途中对 action 进行截获，并进行改变。且由于业务场景的多样性，单纯的修改 dispatch 和 reduce 人显然不能满足大家的需要，因此对 redux middleware 的设计是可以自由组合，自由插拔的插件机制。也正是由于这个机制，我们在使用 middleware 时，我们可以通过串联不同的 middleware 来满足日常的开发，每一个 middleware 都可以处理一个相对独立的业务需求且相互串联：

<img src="https://qiniu-app.qtshe.com/1583492181497.jpg" style="zoom:55%;" />

<br >

如上图所示，派发给 redux Store 的 action 对象，会被 Store 上的多个中间件依次处理，如果把 action 和当前的 state 交给 reducer 处理的过程看做默认存在的中间件，那么其实所有的对 action 的处理都可以有中间件组成的。值得注意的是这些中间件会按照指定的顺序一次处理传入的 action，只有排在前面的中间件完成任务之后，后面的中间件才有机会继续处理 action，同样的，每个中间件都有自己的“熔断”处理,当它认为这个 action 不需要后面的中间件进行处理时，后面的中间件也就不能再对这个 action 进行处理了

而不同的中间件之所以可以组合使用，是因为 Redux 要求所有的中间件必须提供统一的接口，每个中间件的尉氏县逻辑虽然不一样，但只要遵循统一的接口就能和redux以及其他的中间件对话了

<br >

<br >

## 理解Redux中间件机制

由于redux 提供了 applyMiddleware 方法来加载 middleware，因此我们首先可以看一下 redux 中关于 applyMiddleware 的源码：

```javascript
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    // 利用传入的createStore和reducer和创建一个store
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
      )
    }
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    // 让每个 middleware 带着 middlewareAPI 这个参数分别执行一遍
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    // 接着 compose 将 chain 中的所有匿名函数，组装成一个新的函数，即新的 dispatch
    dispatch = compose(...chain)(store.dispatch)
    return {
      ...store,
      dispatch
    }
  }
}
```

从上面的代码我们不难看出，applyMiddleware 这个函数的核心就在于在于组合 compose，通过将不同的 middlewares 一层一层包裹到原生的 dispatch 之上，然后对 middleware 的设计采用柯里化的方式，以便于compose ，从而可以动态产生 next 方法以及保持 store 的一致性

如果觉到绕，不妨看一个啥都不干的中间件是如何实现的

```javascript
const doNothingMidddleware = (dispatch, getState) => next => action => next(action)
```

上面这个函数接受一个对象作为参数，对象的参数上有两个字段 dispatch 和 getState，分别代表着 Redux Store 上的两个同名函数，但需要注意的是并不是所有的中间件都会用到这两个函数。然后 doNothingMidddleware 返回的函数接受一个 next 类型的参数，这个 next 是一个函数，如果调用了它，就代表着这个中间件完成了自己的职能，并将对 action 控制权交予下一个中间件。但需要注意的是，这个函数还不是处理 action 对象的函数，它所返回的那个以 action 为参数的函数才是。最后以 action 为参数的函数对传入的 action 对象进行处理，在这个地方可以进行操作，比如：

- 调动dispatch派发一个新 action 对象
- 调用 getState 获得当前 Redux Store 上的状态
- 调用 next 告诉 Redux 当前中间件工作完毕，让 Redux 调用下一个中间件
- 访问 action 对象 action 上的所有数据

在具有上面这些功能后，一个中间件就足够获取 Store 上的所有信息，也具有足够能力可用之数据的流转

<br >

<br >

## Redux异步处理

#### redux-thunk

直接看看最出名的中间件 redux-thunk 的实现

```javascript
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }
    return next(action);
  };
}
const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;
export default thunk;
```

redux-thunk的代码很简单，它通过函数是变成的思想来设计的，它让每个函数的功能都尽可能的小，然后通过函数的嵌套组合来实现复杂的功能，我上面写的那个最简单的中间件也是如此（当然那是个瓜皮中间件）。redux-thunk 中间件的功能也很简单。首先检查参数 action 的类型，如果是函数的话，就执行这个 action 韩湖水，并把 dispatch, getState, extraArgument 作为参数传递进去，否则就调用 next 让下一个中间件继续处理 action 

需要注意的是，每个中间件最里层处理 action 参数的函数返回值都会影响 Store 上的 dispatch 函数的返回值，但每个中间件中这个函数返回值可能都不一样。就比如上面这个 redux-thunk 中间件，返回的可能是一个 action 函数，也有可能返回的是下一个中间件返回的结果。因此，dispatch 函数调用的返回结果通常是不可控的，我们最好不要依赖于 dispatch 函数的返回值

上面我们已经对redux-thunk进行说明，它通过多参数的 currying 以实现对函数的惰性求值，从而将同步的 action 转为异步的 action，在理解了redux-thunk后，我们在实现数据请求时，action就可以这么写了：

```javascript
function getWeather(url, params) {
  return (dispatch, getState) => {
    fetch(url, params)
      .then(result => {
      dispatch({
        type: 'GET_WEATHER_SUCCESS', payload: result,
      });
    })
      .catch(err => {
      dispatch({
        type: 'GET_WEATHER_ERROR', error: err,
      })
    })
  }
}
```

尽管redux-thunk很简单，而且也很实用，但人总是有追求的，都追求着使用更加优雅的方法来实现redux异步流的控制，这就有了redux-promise

<br >

#### redux-promise

不同的中间件都有着自己的适用场景，react-thunk 比较适合于简单的API请求的场景，而 Promise 则更适合于输入输出操作，比较fetch函数返回的结果就是一个Promise对象

直接来看下最简单的 Promise 对象是怎么实现的：

```javascript
import { isFSA } from 'flux-standard-action'

function isPromise(val) {
  return val && typeof val.then === 'function'
}

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action)
        ? action.then(dispatch)
        : next(action)
    }

    return isPromise(action.payload)
      ? action.payload.then(
          result => dispatch({ ...action, payload: result }),
          error => {
            dispatch({ ...action, payload: error, error: true })
            return Promise.reject(error)
          }
        )
      : next(action)
  };
}
```

它的逻辑也很简单主要是下面两部分：

* 先判断是不是标准的 flux action。如果不是，那么判断是否是 promise, 是的话就执行 action.then(dispatch)，否则执行 next(action)

* 如果是, 就先判断 payload 是否是 promise，如果是的话 payload.then 获取数据，然后把数据作为 payload 重新 dispatch({ ...action, payload: result}) ；不是的话就执行 next(action)

结合 redux-promise 我们就可以利用 es7 的 async 和 await 语法，来简化异步操作了

```javascript
const fetchData = (url, params) => fetch(url, params)
async function getWeather(url, params) {
  const result = await fetchData(url, params)
  if (result.error) {
    return {
      type: 'GET_WEATHER_ERROR', error: result.error,
    }
  }
  return {
    type: 'GET_WEATHER_SUCCESS', payload: result,
  }
}
```

<br >

<br >

## redux-saga

redux-saga是一个管理redux应用异步操作的中间件，用于代替 redux-thunk 的。它通过创建 Sagas 将所有异步操作逻辑存放在一个地方进行集中处理，以此将react中的同步操作与异步操作区分开来，以便于后期的管理与维护

redux-saga特点

* 集中处理redux副作用问题

- 异步实现为generator
- watch/worker(监听->执行)的工作形式
- 提供 cancel/delay 方法，可以便利的取消或延迟异步请求
- 提供 race(effects)，[...effects] 方法来支持竞态和并行场景
- 提供 channel 机制支持外部事件

redux-saga相当于在Redux原有数据流中多了一层，通过对Action进行监听，从而捕获到监听的Action，然后可以派生一个新的任务对state进行维护，通过更改的state驱动View的变更

<img src="https://qiniu-app.qtshe.com/1583494820477.jpg" style="zoom:87%;" />

总的来讲Redux Saga适用于对事件操作有细粒度需求的场景，同时它也提供了更好的可测试性，与可维护性，比较适合对异步处理要求高的大型项目，而小而简单的项目完全可以使用redux-thunk就足以满足自身需求了。毕竟react-thunk对于一个项目本身而言，毫无侵入，使用极其简单，只需引入这个中间件就行了，而react-saga则要求较高，难度较大，这里就不进行底层实现分析了

