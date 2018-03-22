---
title: Codewars每日一练之求对数字每位数的n次幂之和是否等于本身 
author: Huanqiang
tags: [Codewars, JavaScript, 正则表达式]
categories: [算法与数据结构]
keyword: [Codewars, JavaScript, 正则表达式]
date: 2018-01-10 10:31:49
---


## 题目

[答案链接](https://www.codewars.com/kata/5626b561280a42ecc50000d1/solutions/javascript)

Description:

The number `89` is the first integer with more than one digit that fulfills the property partially introduced in the title of this kata. What’s the use of saying “Eureka”? Because this sum gives the same number.

<!-- more -->

In effect: `89 = 8^1 + 9^2`

The next number in having this property is `135`.

See this property again: `135 = 1^1 + 3^2 + 5^3`

We need a function to collect these numbers, that may receive two integers `a`, `b` that defines the range `[a, b]` (inclusive) and outputs a list of the sorted numbers in the range that fulfills the property described above.

Let’s see some cases:

```javascript
sumDigPow(1, 10) == [1, 2, 3, 4, 5, 6, 7, 8, 9]

sumDigPow(1, 100) == [1, 2, 3, 4, 5, 6, 7, 8, 9, 89]
```

If there are no numbers of this kind in the range [a, b] the function should output an empty list.

```javascript
sumDigPow(90, 100) == []
```

Enjoy it!!

## 我的求解

```javascript
function sumDigPow(a, b) {
  // Your code here
  let result = []
  for (let index = a; index <= b; index++) {
    let x = `${index}`
    let sum = 0
    for (let j = 0; j < x.length; j++) {
      sum += Math.pow(parseInt(x[j] ), j + 1)     
    }
    if (index === sum) {
      result.push(index)
    }
  }
  return result
}
```

这里为了方便地获得每一个数字及其位置 `index`，我将数字转换成了字符串，然后使用了字符串的循环来获取每一位数字及其位置 `index`。这样就避免了使用数学方法 let x = a % 10 和 let y = a / 10 这样笨拙的方式获取每一个数字及其位置 `index`。

## 最佳实践

```javascript
function sumDigPow(a, b) {
  var ans = [];
  while(a <= b){
    if(a.toString().split('').reduce((x,y,i)=>x + +y ** (i + 1),0) == a)
      ans.push(a);
    a++;
  }
  return ans;
}
```

这里就解释一下 `if` 语句里的内容吧

1. 将数字 a 转换成字符串，为了更方便获得每一位数字，把字符串转换成了数组。
2. 利用了数组的 `reduce` 方法，将每一位数字的n次幂之和加起来了（n 为 改为数字的位置）。

这里值得注意的是：

1. 使用了 `ES6` 的 `**` 语法，其实就是 `Math.pow()`；

2. 注意到 `x + +y ** (i + 1)`，这里有两个加号，第一个加号就是普通意义上的加号，第二个加号的作用是把这里的字符串 y 转换成 `Number`（因为我们之前 `split` 后的数组中的每一位实际上就是字符串而不是数字）。实际上这句话和 `Math.pow(parseInt(y), i + 1)` 是等价的。第二个加号的转换方式是成立的，可以使用一下代码来测试:

   ```javasjavascriptcript
   console.log(typeof (0 + +`${9}`))	// number
   console.log(0 + `${9}`)				// 09
   console.log(typeof (0 + `${9}`))	//string
   ```