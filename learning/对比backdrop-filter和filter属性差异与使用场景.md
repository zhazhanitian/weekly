## 前言

在浏览tailwind css文档过程中，发现两个很有意思但又不怎么熟悉的css属性-backdrop-filter 和 filter，这两虽然不怎么常用，但是大致了解一番后发现在一些经典场景中有非常显著的效果，这里记录一下了解过程中所学到的东西

在前端开发中，backdrop-filter 和 filter 是两个常用的 CSS 属性，用于实现元素的视觉效果。尽管它们有着类似的作用，但它们之间存在一些关键的差异。本文将深入比较 backdrop-filter 和 filter，包括功能、适用对象、效果范围和浏览器支持等方面

<br />

## backdrop-filter

`backdrop-filter`是一种CSS属性，它允许您对元素后面的内容应用视觉效果。通过使用`backdrop-filter`，您可以模糊、变亮、变暗、添加对比度等，从而创建出各种独特的效果。

下面是一个使用`backdrop-filter`将背景模糊效果应用于元素的示例代码：

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    .tooltip {
      position: relative;
      display: inline-block;
      cursor: pointer;
      margin: 400px;
    }

    .tooltip-text {
      position: absolute;
      top: -35px;
      left: 50%;
      white-space: nowrap;
      transform: translateX(-50%);
      padding: 8px;
      background-color: rgba(0, 0, 0, 0.8);
      color: white;
      border-radius: 4px;
      opacity: 0;
      transition: opacity 0.3s ease;
      pointer-events: none;
      z-index: 1;
      backdrop-filter: brightness(150%);
      font-size: 12px;
    }

    .tooltip:hover .tooltip-text {
      opacity: 1;
      pointer-events: auto;
    }
  </style>
</head>
<body>
  <div class="tooltip">
    Hover over me
    <span class="tooltip-text">Tooltip Text</span>
  </div>
</body>
</html>

```

在上述示例中，通过给工具提示文本添加 `backdrop-filter` 属性和亮度效果，实现了一个具有半透明背景的工具提示

效果如下

<img width="363" style="float: left;" alt="image" src="https://github.com/zhazhanitian/weekly/assets/31462942/e84d7d24-246c-4dff-8ea2-e694c4e880f4">

未使用 `backdrop-filter` 效果

<img width="363" style="float: left;" alt="image" src="https://github.com/zhazhanitian/weekly/assets/31462942/460ee468-941b-4e45-bfe5-51a028abe73c">

从整体对比效果上可以看出，使用模糊效果后光亮度会有一定的减弱

`backdrop-filter`具有一些常用的效果选项，包括

- `blur()`：将元素后面的内容进行模糊处理
- `brightness()`：调整元素后面的内容的亮度
- `contrast()`：调整元素后面的内容的对比度
- `grayscale()`：将元素后面的内容转为灰度
- `invert()`：反转元素后面的内容的颜色
- `opacity()`：调整元素后面的内容的透明度
- `saturate()`：调整元素后面的内容的饱和度

通过组合和调整这些效果选项，您可以实现出各种独特的视觉效果

<br />

## filter

`filter`是另一种CSS属性，它允许您直接对元素及其内容应用视觉效果。与`backdrop-filter`不同，`filter`作用于元素本身，而不是其后面的内容

下面是一个使用`filter`将图像应用灰度效果的示例代码

```css
.image {
  filter: grayscale(100%);
}
```

在上面的示例中，`.image`类给定了一个`filter`，灰度效果为100%，这将使图像变为黑白

`filter`具有广泛的效果选项，包括

- `blur()`：对元素及其内容进行模糊处理
- `brightness()`：调整元素及其内容的亮度
- `contrast()`：调整元素及其内容的对比度
- `grayscale()`：将元素及其内容转为灰度

- `hue-rotate()`：旋转元素及其内容的色相
- `invert()`：反转元素及其内容的颜色
- `opacity()`：调整元素及其内容的透明度
- `saturate()`：调整元素及其内容的饱和度
- `sepia()`：将元素及其内容转为乌贼墨效果

与`backdrop-filter`类似，通过组合和调整这些效果选项，您可以实现出各种独特的视觉效果

<br />

## 比较 backdrop-filter 和 filter

虽然`backdrop-filter`和`filter`都用于应用视觉效果，但它们之间存在一些关键的区别

#### 目标元素

- `backdrop-filter`的目标是元素后面的内容，可以通过对背景应用效果来影响元素后面的内容
- `filter`的目标是元素本身及其内容，直接对元素及其内容应用效果

#### 功能和效果

- `backdrop-filter`提供了一系列效果选项，主要用于调整背景，如模糊、亮度、对比度等
- `filter`提供了更多的效果选项，既可以用于背景，也可以用于元素本身及其内容，包括模糊、亮度、对比度、灰度、色相旋转、透明度等

#### 浏览器支持

- `backdrop-filter`具有较新的特性，因此在浏览器支持方面相对较少，需要较新的浏览器版本
- `filter`有更好的浏览器支持，包括大多数现代浏览器以及一些较旧的版本

#### 使用场景

##### backdrop-filter

- 创建具有模糊或着色背景的视觉吸引力覆盖层，如模态窗口、工具提示等
- 在特定区域突出显示内容，通过调整背景对比度、亮度等效果
- 为元素创建有层次感的效果，使其在页面中脱颖而出

##### filter

- 为图像添加艺术效果，如应用灰度、乌贼墨效果等
- 调整图像的亮度、对比度、饱和度，以提高图像的可视性
- 创建独特的元素效果，如模糊、色相旋转等

<br />

## 在taiwind css 中使用backdrop-filter和filter

在 Tailwind CSS 中，`backdrop-filter` 和 `filter` 是通过添加相应的类名来应用的。下面是关于如何在 Tailwind CSS 中使用 `backdrop-filter` 和 `filter` 的示例

#### 使用 `backdrop-filter`

要使用 `backdrop-filter`，您可以在 HTML 元素中添加 `backdrop-filter` 类，并使用对应的效果类名

```html
<div class="backdrop-filter backdrop-blur-lg">
  <!-- 元素内容 -->
</div>
```

在上面的示例中，我们在 `<div>` 元素上添加了 `backdrop-filter` 类，并使用了 `backdrop-blur-lg` 类来应用模糊效果

#### 使用 `filter`

要使用 `filter`，您可以在 HTML 元素中添加 `filter` 类，并使用对应的效果类名

```html
<img src="image.jpg" alt="Image" class="filter grayscale">
```

在上面的示例中，我们在 `<img>` 元素上添加了 `filter` 类，并使用了 `grayscale` 类来应用灰度效果

#### 自定义效果选项

除了使用默认的效果选项，还可以自定义 Tailwind CSS 中的 `backdrop-filter` 和 `filter` 效果选项

在 `tailwind.config.js` 文件中，您可以在 `theme` 部分的 `backdropFilter` 和 `filter` 配置中添加或修改效果选项

```json
module.exports = {
  theme: {
    backdropFilter: {
      'none': 'none',
      'blur': 'blur(5px)',
      'brightness': 'brightness(120%)',
      // 添加自定义的效果选项
    },
    filter: {
      'none': 'none',
      'grayscale': 'grayscale(100%)',
      'brightness': 'brightness(80%)',
      // 添加自定义的效果选项
    },
    extend: {},
  },
  variants: {},
  plugins: [],
}
```

在上述示例中，我们添加了一些自定义的效果选项。您可以根据需要添加其他选项，并为其指定对应的 CSS 属性值

使用自定义效果选项时，可以在 HTML 元素中使用这些类名来应用自定义效果

```html
<div class="backdrop-filter blur brightness">
  <!-- 元素内容 -->
</div>

<img src="image.jpg" alt="Image" class="filter grayscale brightness">
```

上述示例中，我们使用了自定义的效果选项 `blur`、`brightness` 和 `grayscale` 来应用相应的效果

通过以上方式，可以在 Tailwind CSS 中轻松地使用 `backdrop-filter` 和 `filter` 属性，并根据需要应用不同的效果选项。记得在配置文件中定义和添加自定义效果选项，并在 HTML 元素中使用对应的类名

<br />

## 总结

`backdrop-filter`和`filter`是强大的CSS属性，可用于增强网页元素的视觉效果。`backdrop-filter`适用于调整元素后面内容的视觉效果，而`filter`适用于直接调整元素及其内容的视觉效果。两者具有不同的目标元素、功能和效果，并在浏览器支持方面存在差异

在选择使用`backdrop-filter`还是`filter`时，考虑以下因素

- 如果需要影响元素后面的内容，创建具有模糊、亮度、对比度等效果的覆盖层，或者突出显示特定区域的内容，可以选择使用`backdrop-filter`
- 如果需要直接调整元素及其内容的视觉效果，如图像处理、调整亮度、对比度等，可以选择使用`filter`

同时，需要考虑浏览器支持的情况。虽然`backdrop-filter`和`filter`在现代浏览器中得到广泛支持，但`backdrop-filter`的支持可能相对较新，因此需要确保目标浏览器支持该属性

综上所述，根据具体的需求和浏览器支持情况，选择适当的属性来增强网页元素的视觉效果。`backdrop-filter`适用于调整元素后面内容的效果，而`filter`适用于直接调整元素及其内容的效果