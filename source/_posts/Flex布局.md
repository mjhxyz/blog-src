---
title: Flex布局
index_img: 'https://img.mjhxyz.top//20230401145731.png'
tags:
  - flex布局
categories:
  - 前端
abbrlink: 29914
date: 2023-04-01 09:39:58
---

# Flex 布局

> 由于公司需要使用 uniapp 开发小程序，所以需要接触一下移动端布局相关的问题。而 flex 布局对于移动端应用布局有很强的支持，因此学习后做一些总结。

## Flex 优势

![Flex](https://img.mjhxyz.top//20230401145731.png)

- 简单易学: 相比于传统的 css 布局，更加简洁，不需要大量的使用 float 和 position 属性
- 适应性强: Flex 布局可以很好的适应各种设备屏幕大小, 轻松创建响应式布局，而不需要涉及到 JS
- 灵活性高: Flex 提供了很多的选项，能轻松控制元素的对齐方式
- 可读性好: Flex 提供了一组高度可读性的方式来排列和对齐元素

## Flex 基本概念

- 给父元素添加 display: flex 属性，就可以将其转换为 Flex 布局。Flex 布局的父元素称为 Flex 容器，子元素称为 Flex item。
- Flex 容器 的 `float` `clear` `vertical-align` 都是无效的
- `主轴(main axis)` 和 `交叉轴(cross axis)`
    - Flex 容器存在两个轴
    - 默认`水平方向`为主轴，`垂直方向`为交叉轴

![主轴和交叉轴](https://img.mjhxyz.top//20230401113019.png)

```css
.container {
  display: flex;
}
```

## Flex container 属性

| 属性            | 说明                               |
| --------------- | ---------------------------------- |
| flex-direction  | 决定主轴的方向                     |
| flex-wrap       | 决定是否换行                       |
| flex-flow       | flex-direction 和 flex-wrap 的简写 |
| justify-content | 决定元素在主轴上的对齐方式         |
| align-items     | 决定元素在交叉轴上的对齐方式       |
| align-content   | 决定多根轴线的对齐方式             |

### flex-direction

- 决定主轴的方向
-  可选值
    - `row`(默认): 主轴为水平方向，起点在左端
    - `row-reverse`: 主轴为水平方向，起点在右端
    - `column`: 主轴为垂直方向，起点在上沿
    - `column-reverse`: 主轴为垂直方向，起点在下沿
![flex-diraction](https://img.mjhxyz.top//20230401114509.png)

### flex-wrap

- 决定主轴满了以后, 是否需要换行
- `nowrap`(默认): 不换行
- `wrap`: 换行，第一行在上方
- `wrap-reverse`: 换行，第一行在下方

![flex-wrap](https://img.mjhxyz.top//20230401121749.png)

### flex-flow

- flex-direction 和 flex-wrap 的简写

```css
.container {
  flex-flow: row wrap;
}
```

### justify-content

- 决定元素在主轴上的对齐方式
- 可选值
    - `flex-start`(默认): 左对齐
    - `flex-end`: 右对齐
    - `center`: 居中
    - `space-between`: 两端对齐，元素之间的间隔都相等
    - `space-around`: 每个元素两侧的间隔相等。所以，元素之间的间隔比元素与边框的间隔大一倍


![justify-content 实例](https://img.mjhxyz.top//20230401122904.png)

### align-items

- 决定元素在交叉轴上的对齐方式
-  可选值
    - `flex-start`: 交叉轴的起点对齐
    - `flex-end`: 交叉轴的终点对齐
    - `center`: 交叉轴的中点对齐
    - `baseline`: 项目的第一行文字的基线对齐
    - `stretch`(默认值): 如果项目未设置高度或设为 auto，将占满整个容器的高度

![align-items 实例](https://img.mjhxyz.top//20230401140012.png)

### align-content

- 决定多根轴线的对齐方式, 如果只有一条轴，则不起作用
- 可选值
    - `flex-start`: 和交叉轴的起点对齐
    - `flex-end`: 和交叉轴的终点对齐
    - `center`: 和交叉轴的中点对齐
    - `space-between`: 和交叉轴两端对齐，轴线之间的间隔平均分布
    - `space-around`: 每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍
    - `stretch`(默认值): 轴线占满整个交叉轴

## Flex item 属性


| 属性        | 说明                                        |
| ----------- | ------------------------------------------- |
| order       | 决定元素的排列顺序                          |
| flex-grow   | 决定元素的放大比例                          |
| flex-shrink | 决定元素的缩小比例                          |
| flex-basis  | 决定元素的初始大小                          |
| flex        | flex-grow, flex-shrink 和 flex-basis 的简写 |
| align-self  | 决定元素在交叉轴上的对齐方式                |

### order

- 决定元素的排列顺序
- 默认值为 0，数值越小，排列越靠前

![order 不同的情况](https://img.mjhxyz.top//20230401140403.png)

### flex-grow

- 决定元素的放大比例
- 默认值为 0，(即使轴存在剩余的空间，也不进行放大处理)

![flex-grow 不同](https://img.mjhxyz.top//20230401140604.png)

### flex-shrink

- 决定元素的缩小比例
- 默认值为 1
    - (默认都为 1 表示当空间不足时，等比例缩小)
    - 如果一个元素的 flex-shrink 属性为 0，其他元素都为 1, 则为 0 元素不缩小

![flex-shrikn 不同](https://img.mjhxyz.top//20230401140903.png)

### flex-basis

- 决定元素的初始大小
- 默认值为 `auto`, 即元素的本来大小

### flex

- flex-grow, flex-shrink 和 flex-basis 的简写
- 相应的默认值为: `0 1 auto`

### align-self

- 决定元素在交叉轴上的对齐方式, 可以覆盖 align-items 的值
- 默认为 `auto`, 表示使用父元素的 align-items 属性

![align-self 作用](https://img.mjhxyz.top//20230401141735.png)

# Flex 布局常用场景

## 居中展示

垂直居中且水平居中, 就是两个轴都居中

![居中展示](https://img.mjhxyz.top//20230401144609.png)

### 代码

html
```html
<body>
    <div id="container" class="flex">
        <span>你好</span>
    </div>
</body>
```

css
```css
* {
    box-sizing: border-box;
}

#container {
    width: 150px;
    height: 150px;
    margin: 0 auto;
    border: 1px solid black;
}

.flex {
    display: flex;
    justify-content: center;
    align-items: center;
}
```

## 百分比布局

类似于 Bootstrap 的网格系统,可以使用百分比来布局

![百分比布局](https://img.mjhxyz.top//20230401143944.png)


### 代码

html
```html
<body>
    <div id="container">
        <div class="flex">
            <div class="flex-item i-100">1/1</div>
        </div>
        <div class="flex">
            <div class="flex-item i-25">1/4</div>
            <div class="flex-item i-25">1/4</div>
            <div class="flex-item i-25">1/4</div>
            <div class="flex-item i-25">1/4</div>
        </div>
        <div class="flex">
            <div class="flex-item i-50">1/2</div>
            <div class="flex-item i-50">1/2</div>
        </div>
        <div class="flex">
            <div class="flex-item i-25">1/2</div>
            <div class="flex-item i-25">1/2</div>
            <div class="flex-item">auto</div>
        </div>
    </div>
</body>

```
css
```css
* {
    box-sizing: border-box;
}

#container {
    width: 800px;
    margin: 0 auto;
    border: 1px solid black;
    padding: 10px;
}

.flex-item.i-25 {
    flex-shrink: 0;
    flex-grow: 0;
    flex-basis: 25%;
}

.flex-item.i-50 {
    flex-shrink: 0;
    flex-grow: 0;
    flex-basis: 50%;
}


.flex-item.i-100 {
    flex-shrink: 0;
    flex-grow: 0;
    flex-basis: 100%;

    /* 也可以写成这样 */
    /* flex: 0 0 100%; */
}


.flex {
    display: flex;
    height: 50px;
}

.flex-item {
    border: 1px solid black;
    background-color: #ccc;
    display: flex;
    height: 50px;
    justify-content: center;
    align-items: center;
    font-size: 30px;
    /* 剩余空间平均分配 */
    flex-grow: 1;
}
```








