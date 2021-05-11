##### 前言

composition api（组合api） 和 optional api（可选api） 两种组织代码的方式，在 vue3 各种相关的介绍文里已经了解到不少了，它们可以同时存在，并非强制只能使用哪一种，但组合api两大优势的确让开发者们更倾向于使用它来替代可选api

- 以函数为基础单位来打包可复用逻辑，并注入到任意组件，让视图和业务解耦更优雅
- 让相同功能的业务更加紧密的放置到一起，不被割裂开，提高开发与维护体验

以上两点在react里均被 hook 优雅的解决了，那么相比 hook，组合 api 还具有什么优势呢？在尤大大介绍组合api时已经知道，组合api是静态定义的，解决了 hook 必需每次渲染都重新生成临时闭包函数的性能问题，也没有了 hook 里闭包旧值陷阱，人工检测依赖等编码体验问题

但是，react 是 all in js 的编码方式，所以只要我们敢想、敢做，一切优秀的编程模型都可以吸纳进来，接下来用原生 hook 和 concent 的 setup 并通过实例来分析处理 hook 的痛点问题



##### react hook

在此先设计一个传统的计数器，要求如下

- 有一个小数，一个大数
- 有两组加、减按钮，分别对小数大数做操作，小数按钮加减1，大数按钮加减100
- 计数器初次挂载时拉取欢迎问候语
- 当小数达到100时，按钮变为红色，否则变为绿色
- 当大数达到1000时，按钮变为紫色，否则变为绿色
- 当大数达到10000时，上报大数的数字
- 计算器卸载时，上报当前的数字

为了完成此需求，需要用到以下5把钩子



##### useState

过完需求，需要用到第一把钩子 useState 来做组件首次渲染的状态初始化

```js
function Counter() {
  const [num, setNum] = useState(6)
  const [bigNum, setBigNum] = useState(120)
}
```



##### useCallback

如需使用缓存函数，则要用到第二把钩子 useCallback，此处使用这把钩子来定义加减函数

```js
const addNum = useCallback(() => setNum(num + 1), [num])
const addNumBig = useCallback(() => setBigNum(bigNum + 100), [bigNum])
```



##### useMemo

如需用到缓存的计算结果，则要用到第三把钩子 useMemo，此处使用这把钩子来计算按钮颜色

```js
const numBtnColor = useMemo(() => {
  return num > 100 ? 'red' : 'green'
}, [num])
const bigNumBtnColor = useMemo(() => {
  return bigNum > 1000 ? 'purple' : 'green'
}, [bigNum])
```



##### useEffect

处理函数的副作用则需用到第四把钩子 useEffect，此处用来处理一下两个需求

- 当大数达到10000时，上报大数的数字
- 计算器卸载时，上报当前的数字

```js
useEffect(() => {
  if (bigNum > 10000) api.report('reach 10000')
}, [bigNum])
useEffect(() => {
  return ()=>{
    api.reportStat(num, bigNum)
  }
}, [])
```



##### useRef

上面使用清理函数的 useEffect 写法在IDE是会被警告的，因为内部使用了 num, bigNum 变量（不写依赖会陷入闭包旧值陷阱），所以要求声明依赖

可是如果为了避免IDE警告，改为如下方式显然不是我们想要表达的本意，只是想组件卸载时报告一下数字，而不是每一轮渲染都触发清理函数

```js
useEffect(() => {
  return ()=>{
    api.reportStat(num, bigNum)
  }
}, [num, bigNum])
```

这个时候我们需要第5把钩子 useRef，来帮忙我们固定依赖了，所以正确的写法是

```js
const ref = useRef() // ref是一个固定的变量，每一轮渲染都指向同一个值
ref.current = {num, bigNum} // 帮我们记住最新的值
useEffect(() => {
  return () => {
    const {num, bigNum} = ref.current
    reportStat(num, bigNum)
  }
}, [ref])
```



##### 完整的计数器

使完5把钩子，我们完整的组件如下

```js
function Counter() {
  const [num, setNum] = useState(88)
  const [bigNum, setBigNum] = useState(120)
  const addNum = useCallback(() => setNum(num + 1), [num])
  const addNumBig = useCallback(() => setBigNum(bigNum + 100), [bigNum])
  const numBtnColor = useMemo(() => {
    return num > 100 ? 'red' : 'green'
  }, [num])
  const bigNumBtnColor = useMemo(() => {
    return bigNum > 1000 ? 'purple' : 'green'
  }, [bigNum])
  useEffect(() => {
    if (bigNum > 10000) report('reach 10000')
  }, [bigNum])

  const ref = useRef()
  ref.current = {num, bigNum}
  useEffect(() => {
    return () => {
      const {num, bigNum} = ref.current
      reportStat(num, bigNum)
    }
  }, [ref])

  // render ui ...
}
```

当然我们可以基于 hook 可定制的特性，将这段代码单独抽象为一个钩子，这样的话只需将数据和方法导出，以便让多种ui表达的Counter组件可以复用，同时也做到ui与业务隔离，利于维护

```js
function useMyCounter(){
  // .... 略
  return { num, bigNum. addNum, addNumBig, numBtnColor, bigNumBtnColor}
}
```



##### concent setup

hook 函数在每一轮渲染期间一定是需要全部重新执行一遍的，所以不可避免的在每一轮渲染期间都会产生大量的临时闭包函数，如果我们能省掉他们，的确能帮 gc 减轻一些回收压力的，现在我们来看看使用 setup 改造完毕后的 Counter 会是什么样子

使用 concent 非常简单，只需要在根组件之前，先使用 run api 启动即可，因此处我们没有模块定义，直接调用就可以了

```js
import { run } from 'concent'

run() // 先启动，在render
ReactDOM.render(<App />, rootEl)
```

接着我们将以上逻辑稍加改造，全部包裹到 setup 内部，setup 函数内部的逻辑只会被执行一次，需要用到的由渲染上下文 ctx 提供的api有 initState、computed、effect、 setState，同时配合 setState 调用时还需要读取的状态 state，也由 ctx 获得

```js
function setup(ctx) { // 渲染上下文
  const { initState, computed, effect, state, setState } = ctx
  // setup仅在组件首次渲染之前执行一次，我们可在内部书写相关业务逻辑
}
```



##### initState

initState 用于初始化状态，替代了 useState，当我们的组件状态较大时依然可以不用考虑如何切分状态粒度

```js
initState({ num: 6, bigNum: 120 })
```

此处也支持函数是写法初始化状态

```js
initState(() => ({ num: 6, bigNum: 120 }))
```



##### computed

computed 用于定义计算函数，从参数列表里解构时就确定了计算的输入依赖，相比 useMemo，更直接与优雅

```js
// 仅当num发生变化时，才触发此计算函数
computed('numBtnColor', ({ num }) => (num > 100 ? 'red' : 'green'))
```

此处我们需要定义两个计算函数，可以用你计算对象描述体来配置计算函数，这样只需调用一次 computed 即可

```js
computed({
  numBtnColor: ({ num }) => num > 100 ? 'red' : 'green',
  bigNumBtnColor: ({ bigNum }) => bigNum > 1000 ? 'purple' : 'green',
})
```



##### effect

effect 的用法和 useEffect 是一模一样的，区别仅仅是依赖数组仅传入 key 名称即可，同时 effect 内部将函数组件和类组件的生命周期进行了统一封装，用户可以将业务不做任何修改便迁移到类组件身上

```js
effect(() => {
  if (state.bigNum > 10000) api.report('reach 10000')
}, ['bigNum'])
effect(() => {
  // 这里可以书写首次渲染完毕时需要做的事情
  return () => {
  	// 卸载时触发的清理函数
    api.reportStat(state.num, state.bigNum)
  }
}, [])
```



##### setState

用于修改状态，我们在 setup 内部基于 setState 定义完方法后，然后返回即可，接着我们可以在任意使用此 setup 的组件里，通过 ctx.settings 拿到这些方法句柄便可调用

```js
function setup(ctx) { // 渲染上下文
  const { state, setState } = ctx
  return { // 导出方法
    addNum: () => setState({ num: state.num + 1 }),
    addNumBig: () => setState({ bigNum: state.bigNum + 100 }),
  }
}
```



##### 完整的Setup Counter

基于上述几个api，我们最终的Counter的逻辑代码如下

```js
function setup(ctx) { // 渲染上下文
  const { initState, computed, effect, state, setState } = ctx
  // 初始化数据
  initState({ num: 6, bigNum: 120 })
  // 定义计算函数
  computed({
    // 参数列表解构时就确定了计算的输入依赖
    numBtnColor: ({ num }) => num > 100 ? 'red' : 'green',
    bigNumBtnColor: ({ bigNum }) => bigNum > 1000 ? 'purple' : 'green',
  })
  // 定义副作用
  effect(() => {
    if (state.bigNum > 10000) api.report('reach 10000')
  }, ['bigNum'])
  effect(() => {
    return () => {
      api.reportStat(state.num, state.bigNum)
    }
  }, [])

  return { // 导出方法
    addNum: () => setState({ num: state.num + 1 }),
    addNumBig: () => setState({ bigNum: state.bigNum + 100 }),
  }
}
```

定义完核心的业务逻辑，紧接着，我们可在任意函数组件内部使用 useConcent 装配我们定义好的 setup 来使用它了，useConcent 会返回一个渲染上下文（和 setup 函数参数列表里指的是同一个对象引用，有时我们也称实例上下文），我们可按需获从 ctx 上取出目标数据和方法，针对此示例，我们可以导出

statee (数据)，settings (setup 打包返回的法法)，refComputed (实例的计算函数结果容器)这3个 key 来使用即可

```js
import { useConcent } from 'concent'

function NewCounter() {
  const { state, settings, refComputed } = useConcent(setup)
  // const { num, bigNum } = state
  // const { addNum, addNumBig } = settings
  // const { numBtnColor, bigNumBtnColor } = refComputed
}
```

我们上面提到 setup 同样可以装配给类组件，使用 register 即可，需要注意的是装配后的类组件，可以从 this.ctx 上直接获取 concent 为其生成的渲染上下文，同时呢 this.state 和 this.ctx.state 是等效的，this.setState 和 this.ctx.setState 也是等效的，方便用户代码0改动即可接入 concent 使用

```js
import { register } from 'concent'

@register(setup)
class NewClsCounter extends Component {
  render(){
   const { state, settings, refComputed } = this.ctx
  }
}
```



##### 总结 

对比原生 hook， setup 将业务逻辑固定在只会被执行一次的函数内部，提供了更友好的 api，且同时完美兼容类组件与函数组件，让用户可以逃离 hook 的使用规则烦恼，而不是将这些约束学习障碍转嫁给用户， 同时对gc也更加友好了，可能大都已默认 hook 是 react 的一个重要发明，但是其实它不是针对用户的，而是针对框架的，用户其实是不需要了解那些烧脑的细节与规则的，而对于concent用户来说，其实只需一个钩子开启一个传送门，即可在另一个空间内部实现所有业务逻辑，而且这些逻辑同样可以复用到类组件上

