## 前言

vue3.0 的各种表现还是非常棒的，相比 vue2.0 确实上了一个台阶，据说在客户端渲染效率比 vue2 提升了1.3~2倍，SSR 渲染效率比vue2 提升了2 ~3倍

<br />

<br />

## 静态提升

vue 中有个编译器，在 vue3 的 package.json 文件中有个 @vue/compiler-sfc ，vue2中叫 vue-template-compiler ，这两个就是编译器，它会把我们的模板编译为 render函数 ，在vue3中编译器是很智能的，在编译的过程中，它可以发现哪些节点是静态节点

<br />

#### 静态节点

静态节点就是一个元素节点，而且这个节点里面没有任何动态的内容，就是说没有绑定任何动态的属性，这叫静态节点，比如

```vue
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <HelloWorld msg="Hello Vue 3.0 + Vite" />
</template>
```

上述代码中的 img 就是静态节点，它没有绑定任何动态的内容

然而 vue3 的编译器会发现静态节点，然后将它进行提升，但是在 vue2 中它是不在乎是静态节点，还是动态节点，一顿操作猛如虎，下面对比下

```javascript
//vue2 的静态节点
render(){
    //创建一个虚拟节点h1，没有任何属性，只有内容为"法医"，编译后 <h1>法医</h1>
    createVNode("h1",null,"法医");
    //....其他代码
}

//vue3 的静态节点
const hoisted = createVNode("h1",null,"法医");
function render(){
    //直接使用 hoisted就可以了
}
```

在 vue3 中，它觉得这既然是一个静态节点，那么肯定是不会变化的，不可能说这一次是 h1 元素内容是 法医 ，下次变成 h2 元素内容是别的了，所以说 vue3 认为既然是静态节点，那么就没有必要在 render 函数中进行创建，因为一旦数据改变，render 函数会反复运行，然后又会重新创建这个静态节点，所以为了提升效率，在 vue3 中，它会把静态节点进行提升，提升到 render 函数外面，这样一来，这个静态节点永远只被创建一次，之后直接在 render 函数中使用就行了

<br />

#### 示例

运行一个新创建的vue3项目，在控制台可以清楚的看到，静态节点被提升到外部了。这个就是静态节点的提升

<img src="https://qiniu-image.qtshe.com/1A5731D50E170.png" style="float:left;" width="800px" />

<br />

#### 示例

这是vue3新创建项目中的 APP.vue 组件 ，加一条 h1 元素节点，要注意h1不是静态节点，它是动态的，因为内容是动态的，只有属性是静态的

```vue
//APP.vue代码
<template>
  <h1 class="active">{{name}}</h1>
  <img alt="Vue logo" src="./assets/logo.png" />
  <HelloWorld msg="解决bug的法医" />
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  data(){
    return{
      name:"bobo"
    }
  },
  components: {
    HelloWorld
  }
}
</script>
```

效果如下

<img src="https://qiniu-image.qtshe.com/2A5731D50E170.png" style="float:left;" width="800px" />

<br />

<br />

## 预字符串化

这一点真的是特别厉害， 佩服尤大大 ，我们可以回忆下，在平时vue开发过程中，组件当中没有特别多的动态元素，大多都是静态元素

<br />

#### 示例

```vue
	<div class="menu-bar-container">
    <div class="logo">
      <h1>法医</h1>
    </div>
    <ul class="nav">
      <li><a href="">menu</a></li>
      <li><a href="">menu</a></li>
      <li><a href="">menu</a></li>
      <li><a href="">menu</a></li>
      <li><a href="">menu</a></li>
    </ul>
  </div>
  <div class="user">
    <span>{{user.name}}</span>
  </div>
```

在这个组件中，除了 span 元素是动态元素之外，其余都是静态节点，一般可以说是 动静比 ，动态内容 / 静态内容，比例越小，静态内容越多，比例越大，动态内容越多，vue3的编译器它会非常智能地发现这一点， 当编译器遇到大量连续的静态内容，会直接将它编译为一个普通字符串节点 ，因为它知道这些内容永远不会变化，都是静态节点

> 注意：必须是大量连续的静态内容才可以预字符串化哦，切记！目前是连续20个静态节点才会预字符串化

然而在vue2中，每个元素都会变成虚拟节点，一大堆的虚拟节点，这些全都是静态节点，在vue3中它会智能地发现这一点

```javascript
const _hoisted_1 = /*#__PURE__*/
      
//createStaticVNode 静态节点的意思
_createStaticVNode("<div class=\"menu-bar-container\"><div class=\"logo\"><h1>法医</h1></div><ul class=\"nav\"><li><a href=\"\">menu</a></li><li><a href=\"\">menu</a></li><li><a href=\"\">menu</a></li><li><a href=\"\">menu</a></li><li><a href=\"\">menu</a></li></ul></div>", 1)
```

<br />

#### 看个简图，感受一下 vue3 的魅力

<img src="https://qiniu-image.qtshe.com/3A5731D50E170.png" style="float:left;" width="700px" />



<img src="https://qiniu-image.qtshe.com/4A5731D50E170.png" style="float:left;" width="400px" />

<br />

<br />

## 缓存事件处理函数

#### 示例

```vue
<button @click="count++">plus</button>
```

<br />

#### 对比处理方式

```javascript
// vue2处理方式
render(ctx){
    return createVNode("button",{
        onclick:function($event){
            ctx.count++;
        }
    })
}

//vue3 处理方式
render(ctx,_cache){
    return createVNode("button",{
        onclick:cache[0] || (cache[0] =>($event) =>(ctx.count++))
    })
}
```

在vue2中创建一个虚拟节点button，属性里面多了一个事件onclick，内容就是count++

在vue3中就有缓存了，它认为这里的事件处理是不会变化的，不是说这次渲染是事件函数，下次就变成别的了，于是vue3会智能地发现这一点，会做缓存处理，它首先会看一看缓存里面有没有这个事件函数，有的话直接返回，没有的话就直接赋值为一个count++函数，保证事件处理函数只生成一次，如下图

<img src="https://qiniu-image.qtshe.com/5A5731D50E170.png" style="float:left;" width="800px" />

<br />

<br />

## Block Tree

 Block Tree  主要为了提高新旧两棵树在对比差异的时候提升效率，对比差异的过程叫 diff算法 ，也叫 patch算法 

vue2在对比新旧两棵树的时候，并不知道哪些节点是静态的，哪些节点是动态的，因此只能一层一层比较，这就浪费了大部分时间在对比静态节点上

<br />

#### 举个栗子

```vue
<form>
  <div>
    <label>账号：</label>
    <input v-model="user.loginId" />
  </div>
  <div>
    <label>密码：</label>
    <input v-model="user.loginPwd" />
  </div>
</form>
```

<br />

#### vue2对比

<img src="https://qiniu-image.qtshe.com/6A5731D50E170.png" style="float:left;" width="800px" />



vue2通过一系列对比之后发现，，只有 input 发生了变化，也就是 黄色方块 部分， 蓝色方块 都为 静态节点 ，并没有发生变化，这些没有发生变化的对比都是一些没有意义的对比，浪费了时间，浪费了生命

<br />

<img src="https://qiniu-image.qtshe.com/7A5731D50E170.png" style="float:left;" width="800px" />



vue3 依托强大的编译器，编译器可以对每一个节点进行标记，然后在根节点中记录后代节点中哪些是动态节点，记录之后，在对比的过程中它不是整棵树进行对比，而是直接找到根节点，我们叫 block 节点 ，对比动态节点数组就可以了，这样就会略过所有的静态节点，也不涉及对树的深度遍历了，所以速度会非常快，当静态内容越多，效率提升就越大

当然可能有小伙伴们会问，当数据更新后可能会多出来分支，这样处理的话会造成树的不稳定，树一旦不稳定就会出问题了，凡是树不稳定的地方 vue3 会把它全部变成 块 block ，具体还是挺复杂的，这块后续再做深入探究

<br />

<br />

## PatchFlag

vue3 觉得在对比每一个节点的时候还是在浪费效率，尽管说已经跳过了所有不需要比对的节点，但是它还要看看节点的元素类型、属性以及递归子节点有没有变化，在针对单个节点对比的时候进一步优化，这依然需要依托 vue3 强大的编译器，在编译的时候，它会记录哪个节点是动态内容，并且做上标记

<img src="https://qiniu-image.qtshe.com/8A5731D50E170.png" style="float:left;" width="600px" />

<br />

#### 举个栗子

```vue
<div class="active" title="法医">
  {{user.name}}
</div>
```

<img src="https://qiniu-image.qtshe.com/9A5731D50E170.png" style="float:left;" width="600px" />

vue3 会在编译的时候，它会对节点做上标记，图上标记为 1 ，表示在 div节点 中 text 是 动态 的 

<br />

#### 举个栗子

```vue
<div :class="active" title="法医">
  {{user.name}}
</div>
```

<img src="https://qiniu-image.qtshe.com/1B5731D50E170.png" style="float:left;" width="600px" />

这个 3 表示在 div节点中 text 和 class类是动态的