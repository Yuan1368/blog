---
title: "什么是 CSS 媒体查询"
date: 2022-03-06T17:09:56+08:00
tags: ["前端","CSS"]
categories: ["CSS"]
---

# 什么是 CSS 媒体查询

> 最近在看 《 CSS 权威指南 （第四版）》这本书，书里的前几章介绍了 CSS 的`媒体查询`,这里通过几个例子学习记录下。

## 媒体查询的简单使用

媒体查询主要通过`@media`声明的媒体描述符部分完成，首先看下如下这段代码：

```
div.ppt {
  width : 100px;
  height: 100px;
}

@media projection {
  div.ppt {
    background: blue;
  }
}

@media all {
  div.ppt {
    background: red;
  }
}
```

这段代码从表面上看与正常的代码区别并不大，无非是定义了一个`class`为`.ppt`的`div`元素宽高均为`100px`，但是这段代码由于引入了`@media`，事实上在不同的情景下是会出现不同的背景色，`@media projection`查询的媒体是投影仪的情况下，`.ppt`元素背景色为`blue`，`all`的情景则使用于所有设备。

## 媒体类型

常用的媒体类型有：

all 适用于所有设备。

print 适用于在打印预览模式下在屏幕上查看的分页材料和文档。 （有关特定于这些格式的格式问题的信息，请参阅分页媒体。）

screen 主要用于屏幕。

speech 主要用于语音合成器。

除此之外还有一些废弃的媒体类型：

> 被废弃的媒体类型： CSS2.1 和  Media Queries 3 定义了一些额外的媒体类型 (tty, tv, projection, handheld, braille, embossed, 以及 aural)，但是他们在 Media Queries 4 中已经被废弃，并且不应该被使用。aural 类型被替换为具有相似效果的 speech。

## 媒体特性

如果我们需要希望某个元素在不同的屏幕大小的设备里有不同的表现，我们就可以使用媒体特性为这个元素设置不同的样式。

例如，我们希望元素背景色在移动端显示为红色，在 PC 端显示为绿色，我们就可以这样设置：

```
div.element {
  width : 100px;
  height: 100px;
}

@media screen and (max-device-width: 480px) {
  .element {
    background: red;
  }
}


@media screen and (min-device-width: 480px) {
  .element {
    background: green;
  }
}
```

以上代码查询的一个是最大设备宽度为480px，另一个是最小设备宽度为480px，由于 PC 端的宽度大于480px，因而`element`元素显示为绿色。

这里的使用场景通常是为不同大小的设备定义元素不同的样式，比如说某个元素的背景图我们希望它在手机端和 PC 端有不同的大小以尽量去实现它对不同设备的自适应。

除了上述`max-device-width`媒体特性外，我们可以在 MDN ([使用媒体查询 - CSS（层叠样式表） | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Media_Queries/Using_media_queries#定位媒体类型)) 上查找到其它的一些媒体特性。

另外我们可以通过这个网站查找到当前浏览器所支持的媒体特性：

[css3 媒体查询检测 (rethinkjs.com)](https://www.rethinkjs.com/toolbox/css3-media-queries)

## 逻辑连接符

在上述案例中我们使用了`and`连接符，如果用户希望查询多项内容时就可以使用`and`逻辑连接符，除了`and`连接符，还有`not`连接符，例如用户希望在不支持栅格布局的元素中设置元素样式：

```
@supports not (grid){
    .element{
        .....
    }
}
```

另外还有`or`逻辑连接符也可以使用，使用这个连接符只要浏览器中支持我们设置的某个特性或 CSS 我们就可以设置某元素样式。

## 除此之外呢？

除了媒体查询外，需要注意的是 CSS 一些属性在不同内核的浏览器内是无法属性，这些浏览器内最臭名昭著的当属 IE 浏览器，如果我们希望在支持与不支持这些属性时能有不同的表现形式，我们就可以使用特性查询符`@supports`。

`@supports`的简单使用方法如下：

```
@supports(display: flex){
  .element{
    width: 100px;
    height: 100px;
    background: red;
  }
}
```

在支持`display: flex`的浏览器内，页面能够正常显示`.element`元素的样式，但是如果不支持的话，页面也不会显示出`.element`的样式。

我们通常使用[Can I use... Support tables for HTML5, CSS3, etc](https://caniuse.com/)这个网站去查找不同浏览器内容是否支持某一 CSS 属性，甚至 JS 的属性。