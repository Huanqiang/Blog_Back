---
title: Codewars每日一练之字符串转字符序号 
author: Huanqiang
tags: [每日一练, Codewars, JavaScript]
categories: [算法与数据结构]
keyword: [Codewars, JavaScript]
date: 2018-01-08 10:31:49
---


## 题目

[答案链接](https://www.codewars.com/kata/546f922b54af40e1e90001da/solutions/solutions)

In this kata you are required to, given a string, replace every letter with its position in the alphabet.

If anything in the text isn’t a letter, ignore it and don’t return it.

a being 1, b being 2, etc.

<!-- more -->

As an example:

```javascript
alphabet_position("The sunset sets at twelve o' clock.")
```

Should return `"20 8 5 19 21 14 19 5 20 19 5 20 19 1 20 20 23 5 12 22 5 15 3 12 15 3 11"` as a string.

其实题目很简单，就是给你一个字符串，让你转成对应的1-26的数字即可。

## 我的解答

那么解答思路很简单了，完成以下几点就好了：

1. 去除非字母的字符；（这里借用正则表达式是最方便的）
2. 将大小写统一；（这是为了方便求对应的数字）
3. 对字符串中的每一个字符进行求值；

那么我给出的解答是这样的，一款最死板的解答方案了，实在是丑😭。

```javascript
function alphabetPosition(text) {
  let result = ''
  text = text.replace(/[^A-Za-z]/g, '').toLowerCase()
  for (let i = 0; i < text.length; i++) {
    result +=
      i === text.length - 1
        ? `${text.charCodeAt(i) - 96}`
        : `${text.charCodeAt(i) - 96} `
  }
  return result
}
```

这里我使用了 for 循环，是因为 JS 中字符串不能直接做for循环，也不能做map。完全没有什么难度吧。

其实就是上面这段代码还是可以改进一下下的，我写的时候傻了，这里的

```javascript
result += i === text.length - 1 ? `${text.charCodeAt(i) - 96}` : `${text.charCodeAt(i) - 96} `
```

这段代码完全是多余的，我们只需要进行 `result += ${text.charCodeAt(i) - 96}` 即可，然后在返回的时候删除掉最后一个空格就好了。

```javascript
return result.slice(0, result.length - 1)
```

## 最佳实践

好了，现在让我看下大神的解答：

```javascript
const alphabetPosition = (text) => text.toUpperCase().replace(/[^A-Z]/g, '').split('').map(ch => ch.charCodeAt(0) - 64).join(' ');
```

简直就是精妙，这里有两点值得学习的：

1. 虽然说字符串不能直接使用 `map` 函数，但是数组可以呀，这里巧妙地利用了字符串转数组函数 `split` 来完成转换工作；
2. 在计算出字母对应的序号后，答案还要求每个序号之间要用空格隔开，且返回字符串。其实这样说出来后，都能想到利用 数组转字符串函数 `join` 来转换一下；