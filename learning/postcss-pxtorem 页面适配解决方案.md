## 前言

虽然 postcss-px-to-viewport 插件已经很香（除了插件作者有点坑之外），奈何有时候刚不住其他因素导致需要换一种方案来兼容适配，这里提供另一个很久之前阿里出款的一个适配解决方案 postcss-pxtorem

其功效是将px转换为rem，通过动态调整根节点的font-size大小来实时适配界面

<br />

<br />

## 接入

#### 安装

```nginx
"postcss-pxtorem": "^5.1.0",
"postcss": "^7.0.36",
```

>需要注意的是这里的版本号，两个都有坑，所以直接copy这里的版本号进行安装即可，没必要更新最新版本

<br />

#### 配置 vue.config.js

```js
const autoprefixer = require('autoprefixer')

module.exports = {
  css: {
    loaderOptions: {
      postcss: {
        plugins: [
          autoprefixer(),
          require('postcss-pxtorem')({
            // 根元素字体大小
            rootValue: 16,
            // 匹配CSS中的属性，* 代表启用所有属性
            propList: ['*'],
            // 转换成rem后保留的小数点位数
            unitPrecision: 5,
            // 小于1px的样式不被替换成rem
            minPixelValue: 1,
            // 忽略一些文件，不进行转换，比如我想忽略 依赖的UI框架
            exclude: ['node_modules']
          })
        ]
      }
    }
  }
}
```

<br />

#### 监听页面缩放

```js
const baseSize = 14
function setRem () {
  const scaleW = document.documentElement.clientWidth / 1440
  const scaleH = document.documentElement.clientHeight / 800
  document.documentElement.style.fontSize = baseSize * (scaleW < scaleH ? scaleW : scaleH) + 'px'
}
setRem()
window.addEventListener('resize', () => {
  setRem()
})
```

> 这里的 1440、800分别为对应设计稿的宽高，用来计算缩放比例

<br />

#### 引入监听

在 main.js 中引入监听文件

```js
import './rem.js'
```

<br />

#### 验证效果

<img src="https://user-images.githubusercontent.com/31462942/186060353-13cfa437-930f-43c3-aed5-8c72cfa1e06d.png" style="float: left" />



