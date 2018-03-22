---
title: Codewars每日一练之字符串中制定字母数相同 
author: Huanqiang
tags: [每日一练, Codewars, JavaScript]
categories: [算法与数据结构]
keyword: [Codewars, JavaScript]
date: 2018-01-05 10:31:49
---


还是一道入门级题目，其实本来也没什么好说的，写出来主要是为了体现 `正则` 的重要性。

## 题目

Check to see if a string has the same amount of ‘x’s and ‘o’s. The method must return a boolean and be case insensitive. The string can contains any char.

<!-- more -->

Examples input/output:

```javascript
XO("ooxx") => true
XO("xooxx") => false
XO("ooxXm") => true
XO("zpzpzpp") => true // when no 'x' and 'o' is present should return true
XO("zzoo") => false
```

就是让我们找出一个字符串中字母 x 和 o 的数量是否相同，其中不区分大小写。

## 我的解答

```javascript
function XO(str) {
    let arr = str.toLowerCase().split('')
    return arr.filter(char => char === 'x').length === arr.filter(char => char === 'o').length
}
```

其实我解答也很简单了，应用了数组中的过滤方法来找出字符串中的 x 或 o。

## 最佳实践

```javascript
function XO(str) {
  let x = str.match(/x/gi);
  let o = str.match(/o/gi);
  return (x && x.length) === (o && o.length);
}
```

这里应用了[`match函数`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/match) 来匹配 [正则表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp)，值得注意的一点就是注意当字符串没有 x 或 o 的时候（也就是匹配不成功，match 函数返回 null 的时候），`null.length` 是错误的，所以这里应用了 `x && x.length` 来避免。