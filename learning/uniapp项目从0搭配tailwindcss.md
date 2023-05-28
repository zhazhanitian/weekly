#### 创建项目

```nginx
vue create -p dcloudio/uni-preset-vue my-project
```

<img width="738" alt="image" src="https://github.com/zhazhanitian/Emoji-ChatRoom/assets/31462942/2e912af6-0750-4a6f-851d-5ab6346f710d" style="float: left">

注：

> 这里之所以使用命令创建项目有个非常明显的原因，使用HBuilder创建的项目是非常规node项目，一些配置文件非可视化，连packjson都没有，这不利于项目插件维护

这里创建vue2空模版，按需自行添加插件

<br />

#### 安装Sacc插件

```nginx
yarn add sass sass-loader --dev
```

> 注意：经过反复验证，如果使用的node版本高于14，就不要执行这个操作了，报错的
>
> uniapp已经自己内嵌了sass、less等等编译工具，但是使用方式不能像vue文件那样直接将scss style写在.vue文件中，需要单独创建一个scss文件，然后在.vue文件中通过@import加载

<br />

#### 安装Vuex插件

```nginx
yarn add vuex
```

<br />

#### 安装tailwindcss插件

```nginx
yarn add tailwindcss@npm:@tailwindcss/postcss7-compat postcss@^7 autoprefixer@^9 postcss-class-rename postcss-loader --dev
```

注：

> 这里最好直接使用上面的命令安装，非必要不要更改插件版本，同时可以的话就使用yarn安装，npm和cnpm我都试过，会报错

##### 配置

* 根目录下创建tailwind.config.js文件，加入以下配置内容

```json
module.exports = {
  purge: ['./src/**/*.{vue,js,ts,jsx,tsx}'],
  darkMode: false, // or 'media' or 'class'
  separator: '__', // 兼容小程序，将 : 替换成 __
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
  corePlugins: {
    // 兼容小程序，将带有 * 选择器的插件禁用
    preflight: false,
    space: false,
    divideColor: false,
    divideOpacity: false,
    divideStyle: false,
    divideWidth: false,
  },
};
```

- 修改postcss.config.js文件内容，最终配置如下

```json
const path = require('path')
const webpack = require('webpack')
const config = {
  parser: require('postcss-comment'),
  plugins: [
    require('postcss-import')({
      resolve (id, basedir, importOptions) {
        if (id.startsWith('~@/')) {
          return path.resolve(process.env.UNI_INPUT_DIR, id.substr(3))
        } else if (id.startsWith('@/')) {
          return path.resolve(process.env.UNI_INPUT_DIR, id.substr(2))
        } else if (id.startsWith('/') && !id.startsWith('//')) {
          return path.resolve(process.env.UNI_INPUT_DIR, id.substr(1))
        }
        return id
      }
    }),
    // ---------------------新增----------------------------------
    require('tailwindcss'),
    require('autoprefixer')({
      remove: process.env.UNI_PLATFORM !== 'h5',
    }),
    // --------------新增-------------------------------
    require('postcss-class-rename')({
      '\\\\.': '_', // 兼容小程序，将类名带 .和/ 替换成 _
    }),
    require('@dcloudio/vue-cli-plugin-uni/packages/postcss')
  ]
}
if (webpack.version[0] > 4) {
  delete config.parser
}
module.exports = config
```

- 在 App.vue中加入引入 tailwindcss的代码（**在main.js 里面引入的话打包才成App会不生效**）

```css
<style>
/* tailwindcss */
@import 'tailwindcss/tailwind.css';
</style>
```

* 使用

```html
<view>
  <text class="text-xl font-bold text-red-500">tailwindcss</text>
</view>
```

<br />

#### 总结

至此结束，核心需要注意文中提到的注意点（都是坑）