# Flex 布局常用属性

Flex 布局的属性分两类：

```txt
1. 容器属性：写在父元素上，控制整体布局
2. 项目属性：写在子元素上，控制某个子元素自身表现
```

------

# 1. 容器属性

父元素设置：

```css
.box {
  display: flex;
}
```

之后，`.box` 就变成 Flex 容器，里面的直接子元素就变成 Flex 项目。

------

## 1.1 `flex-direction`

控制主轴方向，也就是子元素排列方向。

```css
.container {
  flex-direction: row;
}
```

常见取值：

| 值               | 含义                 |
| ---------------- | -------------------- |
| `row`            | 默认值，从左到右排列 |
| `row-reverse`    | 从右到左排列         |
| `column`         | 从上到下排列         |
| `column-reverse` | 从下到上排列         |

示例：

```css
.container {
  display: flex;
  flex-direction: row;
}
```

------

## 1.2 `justify-content`

控制项目在 **主轴方向** 上的对齐方式。

```css
.container {
  justify-content: center;
}
```

如果 `flex-direction: row`，主轴就是水平方向。

常见取值：

| 值              | 含义                   |
| --------------- | ---------------------- |
| `flex-start`    | 默认，靠主轴起点       |
| `flex-end`      | 靠主轴终点             |
| `center`        | 居中                   |
| `space-between` | 两端对齐，中间间距平均 |
| `space-around`  | 每个项目两侧间距相等   |
| `space-evenly`  | 所有间距完全相等       |

------

## 1.3 `align-items`

控制项目在 **交叉轴方向** 上的对齐方式。

```css
.container {
  align-items: center;
}
```

如果 `flex-direction: row`，交叉轴就是垂直方向。

常见取值：

| 值           | 含义                   |
| ------------ | ---------------------- |
| `stretch`    | 默认值，拉伸填满交叉轴 |
| `flex-start` | 靠交叉轴起点           |
| `flex-end`   | 靠交叉轴终点           |
| `center`     | 交叉轴居中             |
| `baseline`   | 按文字基线对齐         |

常用居中写法：

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

含义：

```txt
justify-content: center; 主轴居中
align-items: center; 交叉轴居中
```

------

## 1.4 `flex-wrap`

控制子元素是否换行。

```css
.container {
  flex-wrap: wrap;
}
```

常见取值：

| 值             | 含义         |
| -------------- | ------------ |
| `nowrap`       | 默认，不换行 |
| `wrap`         | 换行         |
| `wrap-reverse` | 反向换行     |

示例：

```css
.container {
  display: flex;
  flex-wrap: wrap;
}
```

------

## 1.5 `align-content`

控制 **多行项目** 在交叉轴上的整体分布。

```css
.container {
  align-content: center;
}
```

注意：它通常要在多行情况下才明显生效：

```css
.container {
  display: flex;
  flex-wrap: wrap;
  height: 300px;
}
```

常见取值：

| 值              | 含义                   |
| --------------- | ---------------------- |
| `stretch`       | 默认，多行拉伸         |
| `flex-start`    | 多行靠交叉轴起点       |
| `flex-end`      | 多行靠交叉轴终点       |
| `center`        | 多行整体居中           |
| `space-between` | 多行两端对齐，中间平均 |
| `space-around`  | 每行两侧间距相等       |
| `space-evenly`  | 所有间距完全相等       |

------

## 1.6 `flex-flow`

`flex-flow` 是 `flex-direction` 和 `flex-wrap` 的简写。

```css
.container {
  flex-flow: row wrap;
}
```

等价于：

```css
.container {
  flex-direction: row;
  flex-wrap: wrap;
}
```

------

## 1.7 `gap`

控制项目之间的间距。

```css
.container {
  gap: 16px;
}
```

也可以分开写：

```css
.container {
  row-gap: 12px;
  column-gap: 20px;
}
```

常用于替代给子元素手动写 `margin`。

------

# 2. 项目属性

项目属性写在 Flex 容器的子元素上。

```html
<div class="container">
  <div class="item">1</div>
  <div class="item special">2</div>
  <div class="item">3</div>
</div>
```

------

## 2.1 `order`

控制项目排列顺序。

```css
.special {
  order: -1;
}
```

默认值是：

```css
order: 0;
```

值越小，越靠前。

示例：

```css
.item1 {
  order: 2;
}

.item2 {
  order: 1;
}
```

`item2` 会排在 `item1` 前面。

------

## 2.2 `flex-grow`

控制项目是否放大，占据剩余空间。

```css
.item {
  flex-grow: 1;
}
```

默认值：

```css
flex-grow: 0;
```

表示不主动放大。

示例：

```css
.item1 {
  flex-grow: 1;
}

.item2 {
  flex-grow: 2;
}
```

如果还有剩余空间，`item2` 分到的空间是 `item1` 的 2 倍。

------

## 2.3 `flex-shrink`

控制空间不足时，项目是否缩小。

```css
.item {
  flex-shrink: 1;
}
```

默认值：

```css
flex-shrink: 1;
```

表示空间不够时会缩小。

如果不想被压缩：

```css
.item {
  flex-shrink: 0;
}
```

常见场景：

```css
.avatar {
  width: 48px;
  height: 48px;
  flex-shrink: 0;
}
```

避免头像被挤压变形。

------

## 2.4 `flex-basis`

控制项目在分配空间之前的基础大小。

```css
.item {
  flex-basis: 200px;
}
```

可以理解为：

```txt
在 flex-grow 和 flex-shrink 计算之前，先把这个项目当成 200px 宽
```

如果主轴是水平方向，它类似 `width`。

如果主轴是垂直方向，它类似 `height`。

------

## 2.5 `flex`

`flex` 是最常用的项目属性，是下面三个属性的简写：

```css
flex: flex-grow flex-shrink flex-basis;
```

例如：

```css
.item {
  flex: 1;
}
```

通常等价于：

```css
.item {
  flex-grow: 1;
  flex-shrink: 1;
  flex-basis: 0%;
}
```

常见写法：

| 写法              | 含义                               |
| ----------------- | ---------------------------------- |
| `flex: 1`         | 自动占满剩余空间                   |
| `flex: none`      | 不放大、不缩小                     |
| `flex: 0 0 200px` | 固定 200px，不放大不缩小           |
| `flex: 1 1 auto`  | 可放大、可缩小，基础大小按自身内容 |

经典左右布局：

```css
.layout {
  display: flex;
}

.sidebar {
  width: 200px;
  flex-shrink: 0;
}

.main {
  flex: 1;
}
```

含义：

```txt
sidebar 固定 200px
main 占据剩余空间
```

------

## 2.6 `align-self`

控制某一个项目在交叉轴上的对齐方式，会覆盖容器的 `align-items`。

```css
.special {
  align-self: flex-end;
}
```

常见取值：

| 值           | 含义                             |
| ------------ | -------------------------------- |
| `auto`       | 默认，继承父容器的 `align-items` |
| `flex-start` | 靠交叉轴起点                     |
| `flex-end`   | 靠交叉轴终点                     |
| `center`     | 居中                             |
| `stretch`    | 拉伸                             |
| `baseline`   | 基线对齐                         |

示例：

```css
.container {
  display: flex;
  align-items: center;
}

.special {
  align-self: flex-end;
}
```

意思是：大部分子元素居中，但 `.special` 单独靠底部。

------

# 3. 总表

| 分类     | 属性              | 作用                                |
| -------- | ----------------- | ----------------------------------- |
| 容器属性 | `display: flex`   | 开启 Flex 布局                      |
| 容器属性 | `flex-direction`  | 设置主轴方向                        |
| 容器属性 | `justify-content` | 主轴对齐方式                        |
| 容器属性 | `align-items`     | 单行交叉轴对齐                      |
| 容器属性 | `flex-wrap`       | 是否换行                            |
| 容器属性 | `align-content`   | 多行交叉轴对齐                      |
| 容器属性 | `flex-flow`       | `flex-direction` + `flex-wrap` 简写 |
| 容器属性 | `gap`             | 项目间距                            |
| 项目属性 | `order`           | 排列顺序                            |
| 项目属性 | `flex-grow`       | 是否放大                            |
| 项目属性 | `flex-shrink`     | 是否缩小                            |
| 项目属性 | `flex-basis`      | 基础大小                            |
| 项目属性 | `flex`            | `grow shrink basis` 简写            |
| 项目属性 | `align-self`      | 单个项目交叉轴对齐                  |

------

# 4. 面试回答

Flex 布局属性分为容器属性和项目属性。容器属性写在父元素上，常见有 `display: flex`、`flex-direction`、`justify-content`、`align-items`、`flex-wrap`、`align-content`、`flex-flow` 和 `gap`。其中 `justify-content` 控制主轴对齐，`align-items` 控制交叉轴对齐，`flex-wrap` 控制是否换行，`align-content` 控制多行在交叉轴上的分布。

项目属性写在子元素上，常见有 `order`、`flex-grow`、`flex-shrink`、`flex-basis`、`flex` 和 `align-self`。其中 `flex: 1` 最常用，表示子元素可以自动占据剩余空间；`flex-shrink: 0` 常用于防止元素被压缩；`align-self` 可以让某个子元素单独覆盖父容器的 `align-items` 对齐方式。