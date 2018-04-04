---
title: 理解JavaScript之作用域
author: Huanqiang
tags: [JavaScript, 作用域]
categories: [前端]
keyword: [JavaScript, 作用域]
date: 2017-05-06 10:31:49
---

> 以下讨论的均是由 var 定义的变量的作用范围；因为在 ES6 之后，JS引入了块级作用域，并且有 let、const 等关键字可以定义变量。

在 JavaScript 中，作用域是一个非常有意思的东西，它不同于我们传统的类C 语言的编程语言。在这些编程语言中个，**花括号内的每一段代码都有各自的作用域**，变量在声明它们的代码段之外是不可见的，我们称之为**“块级作用域”**。

但是 JavaScript 不一样，JavaScript 使用的是**“函数作用域”**：**变量在声明它们的函数体内以及这个其嵌套子函数内是有定义(有效)的**。

<!-- more -->

## 1. 作用域

之前我们谈及到 JS 的作用域是“函数作用域”，与代码块无关。

举个栗子：

```javascript
if (1) {
    var test = "test";
}
console.log(test); // 输出： test
```

在这个例子中，test 变量是在 `if 表达式` 中定义的，按照我们以前的类C语言的语法来看，`console.log()` 应该输出错误才对。但是在这里却是成功的，这是为什么的呢？因为 JS 没有块级定义域，而且由 `var` 关键字定义的变量，JS 会将其声明提前，提至该作用域的顶端。

上述例子应该是这样被解析的：

```javascript
var test;
if (1) {
    test = "test";
}
console.log(test); // 输出： test
```

### 1.1. 声明提前

上述案例中，我们提及了一个重要概念：声明提前。就是说**一个被声明的变量，无论其实际声明位置在何处，都会被视为声明于所在函数的顶部**（如果声明不在任意函数内，则视为在全局作用域的顶部）。

> 但是一定要注意一点：这个只对 `var` 关键字声明的对象起作用。`let` 和 `const` 定义的都不会被声明提前。

这里举一个很常见的例子：

```javascript
for (var i = 0; i < 10; i++) {
	console.log(i);
}
console.log(i);		// i 在此处仍然可被访问，输出：10
```

在这个例子中，我们在 `for 语句` 中定义了 `var i = 0`，而在 `for 语句` 结束之后，我们仍能访问到，其原因就是声明提前。

```javascript
var i;
for (i = 0; i < 10; i++) {
	console.log(i);
}
console.log(i);		// i 在此处仍然可被访问，输出：10
```

### 2.2. 同名变量

当子函数中重新定了父函数中的变量时，会发生什么情况呢？直接看例子吧：

```javascript
var name = "laruence";

function echo() {
    console.log(name);	// 2. 输出：undefined
    var name = 'eve';
    console.log(name); 	// 3. 输出：eve
}

console.log(name);		// 1. 输出：laruence
echo();
console.log(name);		// 4. 输出：laruence
```

其实从这个例子很好可以看出 JS 中的函数作用域的作用。

> 代码注释中的数据表示输出结果的顺序。

1. 在 `输出1` 中，此时还没有执行 `echo` 函数，很好理解结果为 `laruence`；
2. 在 `输出2` 中，根据之前说过得声明提前，我们知道了在 `echo` 函数中，其实 `name` 变量是定义在函数的顶部，且未被赋值，故输出 `undefined`；
3. 在 `输出3` 中，此处 `name` 变量被赋值了，所以输出 `eve`；
4. 在 `输出4` 中，这是最有意思的一个输出，我可以看到此时输出的值为 `laruence`，这是为什么呢？因为虽然这里的两个变量的名字都是 `name`， 但是！！！但是**这两个变量确确实实是不一样的变量，新变量屏蔽了父函数中的变量**！

```javascript
最外部执行环境
|—— name			// 这就是我们第一次定义的 name 变量；
|—— echo 函数		// 	这就是我们定义的 echo 函数；
| |—— name		// 这是我们第二次定义的 name 变量，可以看出这个 name 变量是属于 echo 函数的；
```

也就是说在 `输出2` 和 `输出3` 中，输出的其实上是：`echo.name`（不是很准确，但是就是这个意思吧）。

当然，如果子函数没有重新定义父函数中的变量，而是直接使用了这个变量，那么其输出结果和我们之前类C语言的语法所输出的结果无异。

## 2. 执行环境

执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为。每一个执行环境都有一个与之相关的变量对象，**环境中定义的所有变量和函数都保存在这个对象中**。

> 备注：执行环境定义了变量或函数所能访问到的范围。

全局执行环境是最外围的一个执行环境。**根据 JS 实现所在的宿主环境不一样，表示执行环境的对象也会不一样。**比如在 Web 浏览器中，**全局执行环境是 window 对象，所有的全局变量和全局函数都是作为 window 对象 的属性和方法创建的**；而在 Node.js 的环境中，全局执行环境是 **global**。

每一个函数都有自己的执行环境，当执行流进入一个函数的时候，函数的环境会被推入一个环境流。而在函数执行完之后，栈将其环境弹出，把控制权返回给之前的执行环境。

> 也就是说这个栈的底部永远是全局执行环境，顶部是当前正在执行的执行环境。

## 3. 作用域链

当代码在一个环境中执行的时候，会创建变量对象的一个作用域链。**作用域链的用途，是保证对执行环境有权访问的所有对象和函数的有序访问。**作用域的前端，始终都是当前执行代码所在环境的变量对象，如果这个环境是函数，则将 **活动对象** 作为变量对象。作用域链的下一个变量对象来自包含（外部）环境，而再下一个变量对象则来自下一个包含环境。这样，一直延续到全局执行环境；**全局执行环境的变量对象始终都是作用域链中的最后一个对象。**

> 活动对象在最开始的时候只包含一个变量，即 arguments 对象（这在全局环境中是不存在的）。

标识符解析是沿着作用域链一级一级的搜索标识符的过程。搜索过程中始终从作用域的前端开始，然后逐级向后回溯，直到找到标识符为止（如果找不到标识符，通常会导致错误发生）。

让我们看一个例子：

> 请结合 执行环境 来理解

```javascript
var fruit = 'apple';
 
function changeFruit() { 
    var newFruit = 'orange';
 
    function swapFruit() { 
        var tempFruit = newFruit;
        newFruit = fruit;
        fruit = tempFruit;
         //这里可以访问到变量 tempFruit、newFruit 、fruit
        console.log(tempFruit);     // 1. orange
        console.log(newFruit);      // 2. apple
        console.log(fruit);         // 3. orange      
    }
    swapFruit();
    //这里可以访问到变量newFruit、fruit、单不能访问tempFruit
    //console.log(tempFruit);//他会显示变量未定义，因为他不能向下搜索作用域链来进入另一个执行环境
    console.log(newFruit);      // 4. apple
    console.log(fruit);         // 5. orange
}
//这里只能访问fruit
changeFruit();
```

这段代码一共包含了三个执行环境：全局环境、`changeFruit()`的局部环境 和 `swapFruit()`的局部环境。

- 全局环境包括了一个 `fruit` 变量和 `changeFruit()` 函数。
- `changeFruit()` 局部变量包含了一个 `newFruit` 变量和 `swapFruit()` 函数。
- `swapFruit()` 局部环境包含了一个 `tempFruit` 变量。

其中 `swapFruit()` 局部环境中的 `tempFruit` 变量只能被 `swapFruit()` 执行环境所访问到（也就是它仅处于`swapFruit()`的作用域内）。而 `swapFruit()` 则可以访问其它两个环境中的所有变量，因为那两个环境是它的父执行环境。

现在，我们来看一下这个作用域链：

![作用域链](/img/JavaScript/ScopeChain.png)

[作用域链](http://huanqiang.wang/img/JavaScript/ScopeChain.png)

**内部环境可以通过作用域链访问所有的外部环境，但是外部环境不能访问内部环境中的任何变量和函数。**

这些环境之间的联系是线性的，有序的。**每个环境都可以向上搜索作用域链，以查询变量名和函数名；但任何环境都不能通过向下搜索作用域链而进入另一个执行环境。**

## 4. ES6

在 ES6 中，引入了 `let` 和 `const` 两个关键字。

值得注意的是：常量声明与 let 声明一样，都是块级声明。这意味着常量在声明它们的**语句块**外部是无法访问的，并且声明也不会被提升。

> 也就是说：`let` 和 `const` 两个关键字声明的变量是被 `块级作用域` 所限制的。

### 4.1. `let` 声明

`let` 关键字的语法基本上与 `var` 相同，只不过由 `let` 声明的变量，其作用域限制在当前代码块中。

下面这段代码很好的说明了这个问题。

```javascript
function getValue(condition) {
    if (condition) {
        let value = "blue";
        // 其他代码
        return value;
    } else {
        // value 在此处不可用
        return null;
    }
    // value 在此处不可用
}
```

### 4.2. `const` 声明

使用 `const` 声明的变量会被认为是常量（`constant`），意味着它们的值在被设置完成后就不能再被改变。

正因为如此，所有的 `const` 变量都需要在声明时进行初始化，示例如下：

```javascript
// 有效的常量
const maxItems = 30;

// 语法错误：未进行初始化
const name;
```

### 4.3. 暂时性死区

我们知道由 `let` 和 `const` 两个关键字声明的变量是不会被声明提前的，那么它们在代码达到声明处之前都是无法访问的，试图访问会导致一个 引用错误，即使在通常是安全的操作时（例如使用 typeof 运算符），也是如此。示例如下：

```javascript
if (condition) {
    console.log(typeof value);		// 引用错误
    let value = "blue";
}
```

在这段代码中，`value` 位于被 JS 社区称为暂时性死区 （`temporal dead zone`，TDZ，**即从作用域顶部到变量声明之前的区域**）的区域内。该名称并未在 `ECMAScript` 规范中被明确命 名，但经常被用于描述 `let` 或 `const` 声明的变量为何在声明处之前无法被访问。

**当 JS 引擎检查接下来的代码块并发现变量声明时，它在面对 var 的时会将其声明提升到作用域的顶部，而面对 let 或 const时会将声明放在暂时性死区内**。（这段话很重要！！！）

任何在暂时性死区内访问变量的企图都会导致“运行时”错误（`runtime error`）。只有执行到变量的声明语句时，该变量才会从暂时性死区内被移除并可以安全使用。

> 可以这样理解 `let/const` 和暂时性死区：**`let/const` 其实仍是提升了变量的声明的，只不过不像 `var` 一样将变量声明提升到了作用域顶部，而是将其声明在了暂时性死区，目的是为了防止开发者在变量声明之前使用该变量；**

但是，下面这段代码却不会造成“运行时”错误：

```javascript
console.log(typeof value);		// "undefined"

if (condition) {
	let value = "blue";
}
```

原因就是 **暂时性死区 这个概念是作用于块级作用域的**。而在上述代码中，当 `typeof` 运算符被使用时，此时所访问的 `value` 并非 是 `if 语句` 中的 `value`，而是全局作用域中的，所以其没有在暂时性死区内。在该段代码中，只是全局环境没有给其 `value` 变量赋值，所以 `typeof` 返回了 `undefined`。

## 资料

1. 《JavaScript 权威指南》：第 3.10 章；
2. 《JavaScript 高级程序设计》：第 4.2 章；
3. 《JavaScript 忍者秘籍》：第 3.2 章；
4. 《Understanding ES6》：第 1 章；