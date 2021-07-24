## BFC 布局规则

### 什么是 BFC

既 Block Formatting Context（块级格式化上下文），是一个独立的渲染区域，让处于 BFC 内部的元素与外部的元素相互隔离，使内外元素的定位不会相互影响

<br />

### BFC 的触发条件

> Floats, absolutely positioned elements, block containers (such as inline-blocks, table-cells, and table-captions) that are not block boxes, and block boxes with 'overflow' other than 'visible' (except when that value has been propagated to the viewport) establish new block formatting contexts for their contents.
>
> 浮动元素和绝对定位元素，非块级盒子的块级容器（例如 inline-blocks, table-cells, 和 table-captions），以及overflow值不为“visiable”的块级盒子，都会为他们的内容创建新的BFC（块级格式上下文）。
>
> —— W3C

即，存在以下几种方案可创建 BFC

- 浮动元素， `float` 值不为 `none`
- 绝对定位元素，`position` 属性为 `absolute` ，`fixed`
- 非块级盒子的块级容器（ `display` 值为 `inline-blocks` , `table-cells` , `table-captions` 等）
- `overflow` 的值不为 `visible` （ `visiable` 是默认值。内容不会被修剪，会呈现在元素框之外）
- 除此之外，根元素， HTML 元素本身就是 BFC（ 最大的一个BFC ）

<br />

### BFC布局规则

- 内部的盒子会在垂直方向，一个一个地放置
- 盒子垂直方向的距离由 `margin` 决定，属于同一个 BFC 的两个相邻 Box 的上下 `margin` 会发生重叠；
- 每个元素的左边，与包含的盒子的左边相接触，即使存在浮动也是如此
- BFC 的区域不会与 `float` 重叠
- BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之也是如此
- 计算 BFC 的高度时，浮动元素也参与计算

<br />

### BFC 作用

#### 自适应两栏布局

阻止元素被浮动的元素覆盖，自适应成两栏布局

```vue
<!-- 左边的宽度固定，右边的内容自适应宽度(不设置宽度) -->
<div class="ldiv">
 左浮动的元素
</div>
<div class="rdiv">
 没有设置浮动, 没有设置宽度 width
 但是触发 BFC 元素
</div>
<style>
    .ldiv { 
        height: 100px;
        width: 100px;
        float: left;
        background: aqua; 
    }
    .rdiv { 
        height: 100px;
        background: blueviolet; 
        overflow: hidden; 
    }
</style>
```

<img src="https://qiniu-image.qtshe.com/1BAB15C192161C.png" style="float:left; width: 700px" />

<br />

#### 清除内部浮动

解决浮动元素不占高度的问题（浮动元素未被包裹在父容器）

```vue
<div class="parent">
 <div class="child"></div>
</div>
<style>
    .parent { 
        border: 1px solid blueviolet; 
        overflow: hidden; 
    }
    .child { 
        width: 100px;
        height: 100px;
        background: aqua;
        float: left; 
    }
</style>
```

<img src="https://qiniu-image.qtshe.com/2BAB15C192161C.png" style="float:left; width: 700px" />

<br />

#### 解决 margin 重叠

为了防止 margin 重叠， 可以使多个 box 分属于不同的 BFC 时

```html
<div class="container">
    <p></p>
</div>
<div class="container">
    <p></p>
</div>

<style>
  .container {
      overflow: hidden;
  }
  p {
      width: 100px;
      height: 100px;
      background: aqua;
      margin: 10px;
  }
</style>
```

<img src="https://qiniu-image.qtshe.com/3BAB15C192161C.png" style="float:left; width: 500px" />

#### 阻止元素被浮动元素覆盖

```vue
<div class="ldiv">
 左浮动的元素
</div>
<div class="rdiv">
 没有设置浮动, 但是触发 BFC 元素
</div>
<style>
 .ldiv {
        height: 100px;
        width: 100px;
        float: left;
        background: aqua;
    }
 .rdiv {
        width: 300px; 
        height: 200px;
        background: blueviolet; 
        overflow: hidden;
    }
</style>
```

<img src="https://qiniu-image.qtshe.com/4BAB15C192161C.png" style="float:left; width: 500px" />

<br />

<br />

## CSS 格式化上下文

Formatting context(格式化上下文) 是 W3C CSS2.1 规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用

在 CSS 中除了 BFC （块级格式化上下文），还有 IFC、GFC、FFC等，这些统称为CSS **格式化上下文** ，也被称作 **视觉格式化模型** 。而 CSS 视觉格式化模型是用来处理文档并将它显示在视觉媒体上的机制

简单地说， CSS 格式化上下文 **就是用来控制盒子的位置，即实现页面的布局** 

<br />

### IFC：行内格式上下文

行内格式化上下文（Inline Formatting Context），简称 **IFC** 。主要用来规则行内级盒子的格式化规则

IFC 的line box（线框）高度由其包含行内元素中最高的实际高度计算而来（不受到竖直方向的padding/margin影响)

IFC 的行盒的高度是根据包含行内元素中最高的实际高度计算而来。主要会涉及到CSS中的 `font-size` 、 `line-height` 、 `vertical-align` 和 `text-align` 等属性

行内元素从包含块顶端水平方向上逐一排列，水平方向上的 `margin` 、 `border` 、 `padding` 生效。行内元素在垂直方向上可按照顶部、底部或基线对齐

当几个行内元素不能在一个单独的行盒中水平放置时，他们会被分配给两个或更多的(Vertically-stacked Line Box)垂直栈上的行盒，因此，一个段落是很多行盒的垂直栈。这些行盒不会在垂直方向上被分离（除非在其他地方有特殊规定），并且他们也不重叠

那么IFC一般有什么用呢

- 垂直方向上，当行内元素的高度比行盒要低，那么 `vertical-align` 属性决定垂直方向上的对齐方式
- 水平方向上，当行内元素的总宽度比行盒要小，那么行内元素在水平方向上的分部由 `text-align` 决定
- 水平方向上，当行内元素的总宽度超过了行盒，那么行内元素会被分配到多个行盒中去，如果设置了不可折行等属性，那么行内元素会溢出行盒
- 行盒的左右两边都会触碰到包含块，而 `ef="https://github.com/sisterAn/blog">float 元素则会被放置在行盒和包含快边缘的中间位置`

<br />

### GFC（GrideLayout formatting contexts）：网格布局格式化上下文

Grid 格式化上下文（Grid Formaatting Context），俗称 **GFC** 。和FFC有点类似，元素的 `display` 值为 `grid` 或 `inline-grid` 时，将会创建一个Grid容器。该完完全全器为其内容创建一个新的格式化上下文，即Grid格式化上下文。这和创建BFC是一样的，只是使用了网格布局而不是块布局

那么GFC有什么用呢，和table又有什么区别呢？首先同样是一个二维的表格，但GridLayout会有更加丰富的属性来控制行列，控制对齐以及更为精细的渲染语义和控制

<br />

### FFC（Flex formatting contexts）:自适应格式上下文

Flex格式化上下文（Flexbox Formatting Context）俗称 **FFC** 。当 `display` 取值为 `flex` 或 `inline-flex` ，将会创建一个Flexbox容器。该容器为其内容创建一个新的格式化上下文，即Flex格式化上下文

可惜这个牛逼的属性只有谷歌和火狐支持，不过在移动端也足够了，至少safari和chrome还是OK的，毕竟这俩在移动端才是王道

不过要注意的是，Flexbox容器不是块容器（块级盒子），下列适用于块布局的属性并不适用于Flexbox布局

- 多列中的 `column-*` 属性不适用于Flexbox容器
- `float` 和 `clear` 属性作用于Flex项目上将无效，也不会把让Flex项目脱离文档流
- `vertical-algin` 属性作用于Flex项目上将无效
- `::first-line` 和 `::first-letter` 伪元素不适用于Flexbox容器，而且Flexbox容器不为他们的祖先提供第一个格式化的行或第一个字母

