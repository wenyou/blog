---
title: Flex布局
author: Zeeny
comments: false
date: 2020-02-11
updated: 2020-02-11
tags: [Html,CSS]
categories: [HTML,CSS]
summary: Flex 布局
---

# Flex 布局

## 一、display: flex

```
#pond {
  display: flex;
  justify-content:flex-end;
}
```

#### justify-content属性, 这个属性可以水平对齐元素，并且接受以下几个参数：
* flex-start: 元素和容器的左端对齐。
* flex-end: 元素和容器的右端对齐。
* center: 元素在容器里居中。
* space-between:元素之间保持相等的距离。
* space-around:元素周围保持相等的距离。

#### align-items属性, 这个CSS属性纵向对齐元素并且可选以下几个值：
* flex-start: 元素与容器的顶部对齐。
* flex-end: 元素与容器的底部对齐。
* center: 元素纵向居中。
* baseline: 元素在容器的基线位置显示。
* stretch: 元素被拉伸以填满整个容器。

### flex-direction属性，这个CSS属性定义了元素在容器里摆放的方向，并且接受这些值：
* row: 元素摆放的方向和文字方向一致。
* row-reverse: 元素摆放的方向和文字方向相反。
* column: 元素从上放到下。
* column-reverse: 元素从下放到上。

### order属性
<p>order属性，有时候仅仅调转行或列的方向是不够的。在这些情况，我们可以设置单个元素的order属性。元素的属性默认值为0，但是我们设置这个属性为正数或负数。</p>

```
#pond {
  display: flex;
}

.yellow {
    order:1;
}
```

### 另一个你可以使用的控制单个元素的属性是align-self。这个属性接受和align-items一样的那些值。

```
#pond {
  display: flex;
  align-items: flex-start;
}

.yellow {
    align-self:flex-end;
}
```

### flex-wrap属性, 用flex-wrap属性把挤在一行的元素分散到多行。这个属性接受这些值：
* nowrap: 所有的元素都在一行。
* wrap: 元素自动换成多行。
* wrap-reverse: 元素自动换成逆序的多行。

### flex-flow属性
<p>flex-flow属性: flex-direction和flex-wrap两个属性经常会一起使用，所以有缩写属性flex-flow。这个缩写属性接受两个属性的值，两个值中间以空格隔开。</p>
* 举个例子，你可以用flex-flow: row wrap去设置行并自动换行。

### align-content属性，align-content来决定行与行之间隔多远。这个属性接受这些值：
* flex-start: 多行都集中在顶部。
* flex-end: 多行都集中在底部。
* center: 多行居中。
* space-between: 行与行之间保持相等距离。
* space-around: 每行的周围保持相等距离。
* stretch: 每一行都被拉伸以填满容器。

<p>这可能有些容易混淆，但align-content决定行之间的间隔，而align-items决定元素整体在容器的什么位置。只有一行的时候align-content没有任何效果。</p>

```
#pond {
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
}
```

### 练习 Flex 布局的小游戏
[练习 Flex 布局的小游戏](http://flexboxfroggy.com/#zh-cn)