---
title: 理解JavaScript之This
author: Huanqiang
tags: [JavaScript, this, 函数调用方式]
categories: [前端]
keyword: [JavaScript, this, 函数调用方式]
date: 2017-05-06 08:31:49
---

在 Java 等面向对象的语言中，this 关键字的含义是明确的，就是指向当前的对象。这是在其编译期就确定下来了的，称之为 `编译期绑定`。

但是在 JavaScript 中，this 是动态绑定的，是 `运行期绑定`。所以 JavaScript 中的 this 具有多重含义，即不同的调用方式所具有的含义不同。

<!-- more -->

## 1. 作为函数进行调用

当一个函数被调用的时候，除了传入显式的参数外，还会传入两个很重要的东西，一个是 [`arguments` 对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments)，另一个就是 this 。

此时的 this 会被绑定在全局对象上。

> 在 web 浏览器上，全局对象就是 window 对象。

### 1.1. 方法里的嵌套函数

> 请先阅读 **作为方法进行调用**，在回过头来理解这里；

```javascript
var person = {
    name: "xiaoxiao",
    create(name) {

        console.log(this.name);   // 2. 输出： xiaoxiao

        var setName = function (name) {
            console.log(this === global);   // 3. 输出： true
            console.log(this === person);   // 4. 输出： false
            console.log(name)               // 5. 输出： rumeng
            this.name = name;
        }

        setName(name);

        console.log(this.name);   // 6. 输出： xiaoxiao
    }
}

console.log(person.name);     // 1. 输出： xiaoxiao
person.create("rumeng");
console.log(person.name)      // 7. 输出： xiaoxiao
```

1. 输出1：很明显应该输出 `xiaoxiao`，无可非议；
2. 输出2：由 `作为方法进行调用` 这一节可知此时，此时 this 应该指向 person 对象，所以输出 `xiaoxiao`；
3. 输出3 和 输出4： 这是最重要的一块，**由这里可以看出此时的 this 指向的 全局对象 （global），而非 person 对象**，原因就是 **当作为函数进行调用时，this 永远指向 全局函数，无论该函数在哪里被调用的。**
4. 输出5：输出的是传入的参数，所以是 `rumeng`；
5. 输出6：由于 `setName()` 函数并没有改变 `person.name` 的值，所以应该输出 `xiaoxiao`；
6. 输出7：理由同 “输出6”；

> 这里再重复一次：当作为函数进行调用时，this 永远指向`全局函数`，无论该函数在哪里被调用的。

解决方案：在函数调用前，将 this 保存在其他变量上，以便在函数内起作用，详见下面代码段：

```javascript
var person = {
    name: "xiaoxiao",
    create(name) {
        var that = this;          // 将 this 保存在其他变量上，以便在函数作用域中使用
        console.log(this.name);   // 2. 输出： xiaoxiao

        var setName = function (name) {
            console.log(that === global);   // 3. 输出： false
            console.log(that === person);   // 4. 输出： true
            console.log(name)               // 5. 输出： rumeng
            that.name = name;
        }

        setName(name);

        console.log(this.name);   // 6. 输出： rumeng
    }
}

console.log(person.name);     // 1. 输出： xiaoxiao
person.create("rumeng");
console.log(person.name)      // 7. 输出： rumeng
```

## 2. 作为方法进行调用( * )

在 JavaScript 中，函数也是对象，因此函数可以作为一个对象的属性，此时该函数被称为该对象的方法。

在使用这种调用方式时，this 被自然绑定到该对象。如下面的代码段：

```javascript
var person = {
    name: "hack",
    sayName() {
        console.log(this)   // 输出：{ name: 'hack', sayName: [Function: sayName] }
        return this.name;
    }
};

console.log(person.sayName());  // 输出：hack
```

让我们再看一段代码：

> 测试环境： node.js，所以使用了 global 全局变量

```javascript
function creep() { return this; }
console.log(creep() === global);	// 1. 输出 true

var ninja = {
    skulk: creep
};
console.log(ninja.skulk() === ninja); // 2. 输出 true

console.log(creep() === global);	// 3. 输出 true
```

1. 在第一个输出很容易理解，此时 this 指向的明显就是全局变量。
2. 在第二个输出中，此时 `ninja` 对象的 `skulk` 属性引用了 `creep()` 函数。也就是说此时的 `creep()` 函数是作为 `ninja` 的方法进行调用的，所以根据 JavaScript 的 `运行期绑定` 的特性，此时 this 指向了 `ninja` 对象。
3. 而在第三个输出中，此时 this 仍然指向了全局变量。

这个例子就很清楚的解释了 JavaScript 的 `运行期绑定` 的原则。

## 3. 作为构造器进行调用

JavaScript 支持面向对象式编程，与主流的面向对象式编程语言不同，JavaScript 并没有类（class）的概念，而是使用基于原型（prototype）的继承方式。

相应的，JavaScript 中的构造函数也很特殊，**如果不使用 new 调用，则和普通函数一样。**作为又一项约定俗成的准则，**构造函数以大写字母开头**，提醒调用者使用正确的方式调用。

**如果调用正确，函数体内的 this 会被绑定到新创建的对象上。**

举个栗子：

```javascript
function Person(name) {
    this.name = name;
}

var person1 = new Person("xiaoxiao");
console.log(person1.name);		// 输出：xiaoxiao
```

从这段代码容器看出作为构造器调用时 this 的用法。

## 4. 通过 `apply()` 或 `call()` 进行调用

让我们再一次重申，在 JavaScript 中函数也是对象，对象则有方法，其中 `apply()` 或 `call()` 就是函数对象的方法。

这两个方法异常强大，它们允许切换函数执行的上下文环境（context），即 this 绑定的对象。

举个栗子：

```javascript
function Person(name) {
    this.name = name;
    this.resetName = function(name) {
        this.name = name;
    }
}

var person1 = new Person("xiaoxiao");
var person2 = {
    name: "dada"
}
// person1.resetName.apply(person2, ["newname"]);
person1.resetName.call(person2, "newname");
console.log(person2.name)			// 输出：newname
```

从输出结果来看，`person2`对象 的 `name` 属性已经被改变了。原因就是通过 `apply()` 或 `call()` 函数，**改变了 person1.resetName.call(person2, "newname"); 这行代码执行过程中该函数的 this 指向，使之指向了 person2 对象**，所以这行代码改变的不是 `person1`对象的 `name`，而是 `person2`对象的 `name`。

## 箭头函数

在使用箭头函数的时候，**没有 this 绑定**。这句话不是说箭头函数内不能使用 this，而是说箭头函数本身就没有 this，而**箭头函数内使用的 this 实际上指向的是最近一层作用域内的 this。**

```javascript
function foo() {
   return () => {		// 箭头函数 1
      return () => {		// 箭头函数 2
         return () => {		// 箭头函数 3
            console.log("id:", this.id);
         };
      };
   };
}

foo.call( { id: 42 } )()()();
// id: 42
```

在本段代码中，有多少次 this 的绑定执行了呢？大部分人会认为有4次——每个函数里各一次。

事实上，更准确地说，只有一次才对，它发生于 foo() 函数中。**这些接连内嵌的函数们都没有声明它们自己的 this，所以 this.id 的引用会简单地顺着作用域链查找，一直查到 foo() 函数，它是第一处能找到一个确切存在的 this 的地方。**

## 5. 资料

1. 《JavaScript 忍者秘籍》：第 3.3 章
2. [深入浅出 JavaScript 中的 this](https://www.ibm.com/developerworks/cn/web/1207_wangqf_jsthis/)：真的深入浅出，本文很大基础参照了本片博客，很惭愧。
3. [ES6 箭头函数中的 this？你可能想多了](http://www.cnblogs.com/vajoy/p/4902935.html)：这篇博客对箭头函数中的 this 讲述的很不错，值得一看。