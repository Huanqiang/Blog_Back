---
title: Codewars每日一练之求数组中最大最小值
author: Huanqiang
tags: [每日一练, Codewars, JavaScript]
categories: [算法与数据结构]
keyword: [Codewars, JavaScript]
date: 2018-01-06 10:31:49
---

## 题目

[答案链接](http://www.codewars.com/kata/559590633066759614000063/solutions/javascript)

Write a function that returns both the minimum and maximum number of the given list/array.

<!-- more -->

```javascript
minMax([1,2,3,4,5])   == [1,5]
minMax([2334454,5])   == [5, 2334454]
minMax([1])           == [1, 1]
```

编写一个返回给定列表/数组的最小值和最大值的函数。

## 我的求解

这种题目太简单了，写出来就只是为了提醒自己灵活利用 `JavaScript` 的各种API。

先来看看最 low 的求解

```javascript
function minMax(arr) {  
  let min = arr[0]
  let max = arr[0]
  arr.forEach(e => {
    if (e > max) {
      max = e
    }
    if (e < min) {
      min = e
    }
  })
  return [min, max] // fix me!
}
```

这样的解法就是完全不动脑子就能想出来了。我们在看下题目，就能发现无非就是求最大最小值嘛，那么既然循环很麻烦，为什么我们不先进行排序呢？所以就有了下面解法：

```javascript
function minMax(arr) {
  arr = arr.sort((a, b) => a - b)
  return [arr[0], arr[arr.length-1]]
}
```

## 最佳实践

其实，这样写也是麻烦了，我们的思维不能只局限于数组，数组本身没有直接求解最大最小值的API，但是 `Math` 对象有啊，不要忘了，这里的数组中的值全是数字而已。

```javascript
function minMax(arr){
  return [Math.min(...arr), Math.max(...arr)];
}
```

或者：

```javascript
function minMax(arr){
  return [Math.min.apply(Math, arr), Math.max.apply(Math, arr)];
}
```