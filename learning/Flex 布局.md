## 前言

Flex 是 Flexible Box 的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性

任何一个容器都可以指定为 Flex 布局

```css
.box{
  display: flex;
}
```

行内元素也可以使用 Flex 布局

```css
.box{
  display: inline-flex;
}
```

Webkit 内核的浏览器，必须加上 -webkit 前缀

```
.box{
  display: -webkit-flex; /* Safari */
  display: flex;
}
```

注意，设为 Flex 布局以后，子元素的 float、clear 和 vertical-align 属性将失效

<br />

<br />

## 基本概念

采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"

<img src="https://user-images.githubusercontent.com/31462942/149938408-5bb96246-2b1e-4c11-a93d-f53b4b2aa9e5.png" style="width: 700px;float:left;" />

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做 main start ，结束位置叫做 main end ；交叉轴的开始位置叫做 cross start ，结束位置叫做 cross end 

项目默认沿主轴排列。单个项目占据的主轴空间叫做 main size ，占据的交叉轴空间叫做 cross size 

<br />

<br />

## 容器的属性

以下6个属性设置在容器上

>- flex-direction
>- flex-wrap
>- flex-flow
>- justify-content
>- align-items
>- align-content

<br />

#### flex-direction属性

 flex-direction 属性决定主轴的方向（即项目的排列方向）

```css
.box {
  flex-direction: row | row-reverse | column | column-reverse;
}
```

<img src="https://user-images.githubusercontent.com/31462942/149939034-52782738-69ba-4b9a-a97b-c38cb05646ca.png" style="width: 700px;float:left;" />

它可能有4个值

>-  row （默认值）：主轴为水平方向，起点在左端
>-  row-reverse ：主轴为水平方向，起点在右端
>-  column ：主轴为垂直方向，起点在上沿
>-  column-reverse ：主轴为垂直方向，起点在下沿

<br />

#### flex-wrap属性

默认情况下，项目都排在一条线（又称"轴线"）上。 flex-wrap 属性定义，如果一条轴线排不下，如何换行

<img src="https://user-images.githubusercontent.com/31462942/149939351-1665b0b3-6c41-44dd-a923-b50f87030eff.png" alt="image" style="width: 700px;float:left" />

```css
.box{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

它可能取三个值

* nowrap（默认）：不换行

<img src="https://user-images.githubusercontent.com/31462942/149939720-2a5531ca-2602-4645-bea2-9658154d6714.png" alt="image" style="width: 700px;float:left" />

*  wrap ：换行，第一行在上方

<img src="https://user-images.githubusercontent.com/31462942/149939721-61cdd4df-0b81-4c03-b440-55633a19e35e.png" alt="image" style="width: 700px;float:left" />

*  wrap-reverse ：换行，第一行在下方

<img src="https://user-images.githubusercontent.com/31462942/149939740-9c8dd51f-88a4-4b48-962b-685f0abd77e9.png" alt="image" style="width: 700px;float:left" />

<br />

#### flex-flow

flex-flow 属性是 flex-direction 属性和 flex-wrap 属性的简写形式，默认值为 row nowrap

```css
.box {
  flex-flow: <flex-direction> || <flex-wrap>;
}
```

<br />

#### justify-content属性

```css
.box {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

<img src="https://user-images.githubusercontent.com/31462942/149940319-42dd6caa-078b-4de2-a267-bd54b35ec512.png" alt="image" style="width: 700px;float:left" />

它可能取5个值，具体对齐方式与轴的方向有关。下面假设主轴为从左到右

> -  flex-start （默认值）：左对齐
> -  flex-end ：右对齐
> -  center ： 居中
> -  space-between ：两端对齐，项目之间的间隔都相等
> -  space-around ：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍

<br />

#### align-items属性

 align-items 属性定义项目在交叉轴上如何对齐

```css
.box {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```

<img src="https://user-images.githubusercontent.com/31462942/149940666-0b4ad047-6b3a-443d-885f-4c73482c5c2a.png" alt="image" style="width: 700px;float:left" />

它可能取5个值。具体的对齐方式与交叉轴的方向有关，下面假设交叉轴从上到下

> -  flex-start ：与交叉轴的起点对齐
> -  flex-end ：与交叉轴的终点对齐
> -  center ：与交叉轴的中点对齐
> -  space-between ：与交叉轴两端对齐，轴线之间的间隔平均分布
> -  space-around ：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍
> -  stretch （默认值）：轴线占满整个交叉轴

<br />

<br />

## 项目的属性

以下6个属性设置在项目上

> -  order 
> -  flex-grow 
> -  flex-shrink 
> -  flex-basis 
> -  flex 
> -  align-self 

<br />

#### order属性

 order 属性定义项目的排列顺序。数值越小，排列越靠前，默认为0

```css
.item {
  order: <integer>;
}
```

<img src="https://user-images.githubusercontent.com/31462942/149941068-12bae2c6-36dd-418c-9a01-312c49b62e45.png" alt="image" style="width: 700px;float:left" />

<br />

#### flex-grow属性

 flex-grow 属性定义项目的放大比例，默认为 0 ，即如果存在剩余空间，也不放大

```css
.item {
  flex-grow: <number>; /* default 0 */
}
```

<img src="https://user-images.githubusercontent.com/31462942/149941305-5bf6e1bb-f5eb-424a-921b-41becdfcf432.png" alt="image" style="width: 700px;float:left" />

如果所有项目的 flex-grow 属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的 flex-grow 属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍

<br />

#### flex-shrink属性

 flex-shrink 属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小

```css
.item {
  flex-shrink: <number>; /* default 1 */
}
```

<img src="https://user-images.githubusercontent.com/31462942/149941598-f2122211-067b-4b3b-951c-d2beca2aa998.png" alt="image" style="width: 700px;float:left" />

如果所有项目的 flex-shrink 属性都为1，当空间不足时，都将等比例缩小。如果一个项目的 flex-shrink 属性为0，其他项目都为1，则空间不足时，前者不缩小

负值对该属性无效

<br />

#### flex-basis属性

 flex-basis 属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为 auto ，即项目的本来大小

```css
.item {
  flex-basis: <length> | auto; /* default auto */
}
```

它可以设为跟 width 或 height 属性一样的值（比如350px），则项目将占据固定空间

<br />

####  flex属性

 flex 属性是 flex-grow ,  flex-shrink  和  flex-basis 的简写，默认值为 0 1 auto 。后两个属性可选

```css
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

该属性有两个快捷值： auto  ( 1 1 auto ) 和 none ( 0 0 auto )

建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值

<br />

#### align-self属性

align-self 属性允许单个项目有与其他项目不一样的对齐方式，可覆盖 align-items 属性。默认值为 auto ，表示继承父元素的 align-items 属性，如果没有父元素，则等同于 stretch

```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

<img src="https://user-images.githubusercontent.com/31462942/149942134-a22e190f-d53d-4c95-9e0c-2bf18f303478.png" alt="image" style="width: 700px;float:left" />

该属性可能取6个值，除了auto，其他都与align-items属性完全一致

