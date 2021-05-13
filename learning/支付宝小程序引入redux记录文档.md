## 文件结构

在项目根目录下文件结构目录

<img src="https://qiniu-app.qtshe.com/image-20200628101524655.png" alt="image-20200628101524655" style="zoom:60%;float: left;" />

* actions 文件夹

  用于存放各模块的 action 文件

  > **Action** 是把数据从应用（译者注：这里之所以不叫 view 是因为这些数据有可能是服务器响应，用户输入或其它非 view 的数据 ）传到 store 的有效载荷。它是 store 数据的**唯一**来源。一般来说你会通过 `store.dispatch()`) 将 action 传到 store
  >
  > Action 本质上是 JavaScript 普通对象。我们约定，action 内必须使用一个字符串类型的 `type` 字段来表示将要执行的动作。多数情况下，`type` 会被定义成字符串常量。当应用规模越来越大时，建议使用单独的模块或文件来存放 action

* reducers 文件夹

  用于存放各模块的 reducer 文件

  > Reducers 指定了应用状态的变化如何响应 actions 并发送到 store 的，记住 actions 只是描述了有事情发生了这一事实，并没有描述应用如何更新 state

* index.js

  组装 Store，依靠 combineReducers 来组合各 reducer 数据，再传给 createStore 构建 Store 仓库

  ```js
  import {createStore, combineReducers} from './redux.min.js'
  import myCenterV2 from './reducers/myCenterV2.js'
  import publish from './reducers/publish.js'
  import systemFields from './reducers/systemFields.js'
  
  export default createStore(combineReducers({ myCenterV2, publish, systemFields }))
  ```

  > **Redux 应用只有一个单一的 store**。当需要拆分数据处理逻辑时，应该使用 reducer 组合而不是创建多个 store

* public.js

  公共方法及 action type 统一维护文件（由于type内容不多，所以统一为一个对象维护，若量过大时需要拆分）

* redux.min.js

  redux SDK文件

<br >

<br >

## Actions

Actions的action文件内容十分简单，就是对 dispatch 做一层封装，接收需要 dispatch 的 action data 对象，返回一个包含 action type 和 action data 的对象，统一存储，便于统一维护及重复使用，能清晰浏览各模块更新渠道和内容

例如：

定义

```js
/** 
 * 更新行业选择
 */
export const updateSelectIndustry = data => {
    return {
        type: AT.UPDATE_SELECT_INDUSTRY,
        data
    }
}
```

使用

```js
import { updateSelectIndustry } from '/store/actions/publish.js'

app.$store.dispatch(updateSelectIndustry({ firstIndustryId: id }))
```

<br >

<br >

## Reducers模块拆分逻辑

### 设计思路

由于小程序端模块并不是很多，需要使用到状态管理工具的地方较少，基本需要的数据也就用户信息、发布填写内容及一些零散字段 （因为这里零散字段较少也无模块可分才合一，若内容过多时需要拆分）

基此，将**获取用户信息接口返回内容做为一个模块维护、发布相关内容做一个模块维护、其余零散字段单独作为一个模块维护**

<br >

### 实现方式

* 每个 reducer 文件都提供一个initialState 方法，用于初始化该模块的 state 值

* 使用 public.js 中的 createReducer 方法加工 reducer，依次去掉可怕的深层 switch case

  ```js
  /**
   * reducer 构造器
   */
  export const createReducer = (initialState = {}, handlers = {}) => {
    return function reducer(state = initialState, action = {}) {
      if (handlers.hasOwnProperty(action.type)) {
        return handlers[action.type](state, action)
      } else {
        return state
      }
    }
  }
  ```

  ```js
  const systemFields = createReducer(initialState(), {
      [AT.UPDATE_SYSTEM_FIELDS]: updateSystemFields     // 更新零散字段信息
  })
  ```

* 修改模块方式：

  1. 更新和重置分开，这么设计方法会十分纯粹，更新就单独更新某些字段，重置就重置整个对象（现在的使用方式）
  2. 更新和重置柔和，在传入的 data 中使用 type 值来做区分

<br >

<br >

## 关注点

1. **store引入问题（巨坑）**

   根据 module 的导出以及 redux store 的单一对象原则，讲道理不论我们在任何文件中 import sotre 都应该是同一个对象，但是当我们在 util.js 和 app 中分别 `import $store from './store/index.js'` 导入 store 时，**发现生成了两份store对象（在IDE上面没有此问题，真机复现）**

   解决方案：

   store 的入口统一从 app.js 中引入，其他页面统一从 app.js 中获取

2. **setStorageSync/getStorageSync 调用优化**

   小程序中不可避免的会使用到同步缓存，但是有时候一个字段反复被获取和设置，这时可以考虑放在 store 的零散模块中去维护，减少同步缓存读取次数

   ```js
   /**
    * getStorageSync 方法
    */
   function getStorageSync(key) {
     const app  = getApp()
     return (app && app.$store && app.$store.getState().systemFields[key]) || my.getStorageSync({ key }).data
   }
   ```

   **这里的 app 不能在 util 文件头获取，因为在 app 页面头部有获取util，这是 app 未初始化好，如果放在util 头部获取，那么这里拿到的 app 可能时 undefined** 

<br >

<br >

## 引入步骤介绍（以零散字段模块为例）

[详细内容](http://cn.redux.js.org/docs/basics/Store.html)

1. 搭建目录结构，文章开头图

2. index.js 文件内容

   ```js
   import systemFields from './reducers/systemFields.js'
   
   export default createStore(combineReducers({ systemFields }))
   ```

3. public.js 文件内容，定义 reducer 构造器和 action type 类型

   ```js
   /**
    * reducer 构造器
    */
   export const createReducer = (initialState = {}, handlers = {}) => {
     return function reducer(state = initialState, action = {}) {
       if (handlers.hasOwnProperty(action.type)) {
         return handlers[action.type](state, action)
       } else {
         return state
       }
     }
   }
   
   /**
    * JSON 深拷贝
    */
   export const deepCopy = data => {
     return JSON.parse(JSON.stringify(data))
   }
   
   /**
    * action type
    */
   export const AT = {
     UPDATE_SYSTEM_FIELDS: 'UPDATE_SYSTEM_FIELDS',            // 更新零散字段信息
   }
   ```

4. actions/systemFields.js 文件内容，定义 action type 方法

   ```js
   import { AT } from '../public.js'
   
   /** 
    * 更新发布填写内容
    */
   export const updateSystemFields = data => {
       return {
           type: AT.UPDATE_SYSTEM_FIELDS,
           data
       }
   }
   ```

5. reducers/systemFields.js 文件内容，初始化 reducer 及定义修改数据方法

   ```js
   import { createReducer, AT, deepCopy } from '../public'
   
   /**
    * 存储零散的常用字段，在uitl的setStoryge中更新
    * 比如埋点文件中一直在读取缓存中的经纬度，又不能直接去污染ptpjs，所以可以在uitl的getStoryge中去返回
    */
   const initialState = () => {
       return {
           jwtToken: '',
           token: '',
           longitude: '',      // 经度
           latitude: '',       // 纬度
           townId:   '',       // 城市ID
           townName: '',       // 城市名称
           platform: '',       // 平台ID
           deviceId: '',       // 随机数
           authorizedKey: ''   // 渠道
       }
   }
   
   /**
    * 更新零散字段信息
    */
   const updateSystemFields = (state, action) => {
       const data = {
           ...state,
           ...action.data
       }
       // console.log('更新后的零散字段信息', deepCopy({ ...data }))
       return { ...data }
   }
   
   const systemFields = createReducer(initialState(), {
       [AT.UPDATE_SYSTEM_FIELDS]: updateSystemFields     // 更新零散字段信息
   })
   
   export default systemFields
   ```

6. 使用

   ```js
   import { updateSystemFields } from '/store/actions/systemFields.js'
   
   app.$store.dispatch(updateSystemFields({ token, jwtToken }))
   ```

7. 一整套流程结束

