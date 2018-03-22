---
title: CSS盒模型简述
author: Huanqiang
tags: [前端, CSS, 盒模型]
categories: [前端]
keyword: [前端, CSS盒模型简述, CSS, 盒模型]
date: 2017-09-27 10:31:49
---

## CSS 盒模型简述

盒模型是 CSS 的基石，在一个文档中，每一个标签（元素）都被表示为矩形的盒子。渲染引擎的目标则是确定这些盒子的尺寸和属性（颜色、边框、背景和位置）。

<!-- more -->

一个盒子的实际大小由它的外边距（margin）、边框（border）、内边距（padding）和内容（content）这四个部分所决定。

![盒子实际大小示意图](https://mdn.mozillademos.org/files/72/boxmodel%20(1).png)

### 内边距

content area 中所设置的背景、图片、颜色会延伸作用在内边距（`padding`）上；

### 外边距合并

对于外边距的而言，最重要的就是外边距合并（margin collapsing，又称外边距塌陷）。外边距塌陷有三种情况：

1. 相邻的两个盒子的上、下外边距会合并，合并为最大的那一个值；
2. 如果块级父元素中不存在 `上边框`、`上内边距`、`内联元素`和`清除浮动`这四个数据（其实就是说**上边框和上内边距距离为0，即第一个子元素的上边距和父元素挨到了一起**）时，会发生子元素的上外边距和父元素的上外边距合并，取两者的最大值。父子元素的下边距合并同理；
3. 当一个一个空的块级元素的 `border`、`padding`、`inline content`、`height` 和 `min-height` 都不存在。其上下边距会发生合并；（举个栗子）

```
<p style="margin-bottom: 0px;">这个段落的和下面段落的距离将为20px</p>

<div style="margin-top: 20px; margin-bottom: 20px;"></div>

<p style="margin-top: 0px;">这个段落的和上面段落的距离将为20px</p>
```

## 资料

摘自：

1. [盒子模型 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model)
2. [外边距合并 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)