---
title: Codewars每日一练之找出字符串中第一个非重复字符 
author: Huanqiang
tags: [每日一练, Codewars, JavaScript, 正则表达式]
categories: [算法与数据结构]
keyword: [Codewars, JavaScript, 正则表达式, match]
date: 2018-01-08 10:31:49
---

## 题目

[答案链接](https://www.codewars.com/kata/52bc74d4ac05d0945d00054e/solutions/javascript)

Write a function named `firstNonRepeatingLetter`† that takes a string input, and returns the first character that is not repeated anywhere in the string.

For example, if given the input `'stress'`, the function should return `'t'`, since the letter *t* only occurs once in the string, and occurs first in the string.

<!-- more -->

As an added challenge, upper- and lowercase letters are considered the **same character**, but the function should return the correct case for the initial letter. For example, the input `'sTreSS'` should return `'T'`.

If a string contains *all repeating characters*, it should return the empty string (`""`).

> † Note: the function is called `firstNonRepeatingLetter` for historical reasons, but your function should handle any Unicode character.编写一个返回给定列表/数组的最小值和最大值的函数。

## 我的求解

题目很简单，直接上答案：

```javascript
function firstNonRepeatingLetter(s) {
  // Add your code here
  let arr = s.split('')
  return arr.filter(e => match(arr, e)).shift() || ''
}

function match(s, e) {
  return s.filter(el => el === e.toLowerCase() || el === e.toUpperCase()).length === 1
}
```

这里的思路很简单，这里企图借用数组的 `filter` 来筛选出字符串中不重复的字符，然后取出第一个，并且因为当字符串为空时， `shift` 函数会抛出 `undefined`，所以使用 `||` 来输出空字符。

其实利用数组的 `filter` 的时候是循环了整个字符串，在这里是没有必要的，**因为题目中要的第一个非重复字符，所以只要找到第一个就好了**，乖乖用 `for` 就好了。

## 最佳实践

```javascript
function firstNonRepeatingLetter(s) {
  var t=s.toLowerCase();
  for (var x=0;x<t.length;x++)
    if(t.indexOf(t[x]) === t.lastIndexOf(t[x]))
      return s[x];
  return "";
}
```

这里有两个比较高明的地方：

1. 它在寻找重复的字符时候使用了 `toLowerCase` 函数转换后的字符串（这里是为什么解决大小写不匹配的问题），而在返回的时候使用的是原字符串。
2. 第二个就是如何判断字符是否重复。最直接，也是最笨拙的想法就是像我一样找出字符串中的所有该字符，然后看 `length` 是不是等于1。其实根本不需要这样，**判断字符是否重复，只要这个字符在字符串中第一次出现和最后一次出现的位置是否一致**。

好了，让我们看看另一个更好的方案：

```javascript
function firstNonRepeatingLetter(s) {
  for(var i in s) {
    if(s.match(new RegExp(s[i],"gi")).length === 1) {
      return s[i];
    }
  }
  return '';
}
```

这里使用了 `for…in` 迭代字符串，然后配上字符串的 `match` 函数简直不要最简单。

> 使用 `for…in` 的时候我们要注意**迭代的顺序是不确定的**。但是在这里还是可以使用的，因为如果是Chrome 和 Opera环境下，字符串中的所有字符的索引的key值都是非负整数的，那么顺序就是一定的了；如果是其他浏览器，则按照对象定义的顺序遍历属性，那么顺序还是一定的。

## `for...in`

这里在顺便捎上 `for…in` 的介绍吧。[MDN 地址附上](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in)

`for…in` 可以以**任意顺序**遍历一个对象的**可枚举属性**。

这里要注意两点：

1. **任意顺序**：就是说其遍历不一定按照次序的。`for…in` 迭代的顺序是依赖于执行环境的，所以当迭代访问数组时，最好还是不要使用 `for…in`，使用 `forEach` 或者 `for...of` 都可以。

   > 1. Chrome 和 Opera：它们会先提取所有 key 的 parseFloat 值为非负整数的属性，然后根据数字顺序对属性排序首先遍历出来，然后按照对象定义的顺序遍历余下的所有属性。
   > 2. 其它浏览器则完全按照对象定义的顺序遍历属性。

2. **可枚举属性**：循环将遍历对象本身的所有可枚举属性，以及对象从其构造函数原型中继承的属性（更接近原型链中对象的属性覆盖原型属性）。