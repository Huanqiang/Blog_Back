---
title: React 代码整洁最佳实践指南
author: Huanqiang
tags: [前端, React, 编程风格, 代码整洁, 重构]
categories: [前端]
keyword: [前端, React, 编程风格, 代码整洁, 重构]
date: 2018-01-23 10:31:49
---

本文将重点介绍 React 开发相关的代码整洁度的最佳实践。

> 本文翻译自 [Clean Code vs. Dirty Code: React Best Practices](http://americanexpress.io/clean-code-dirty-code/)，也不算完全翻译，部分语句按照自己的意思写的，有兴趣的可以看看原文。

## 什么是整洁的代码

整洁的代码是一种前后一致的编程风格，它能使你的代码变得更易于编写、阅读和维护。通常开发者在花费时间解决一个问题后，就将代码提交到版本管理系统上。但是你不能因为你的代码能工作了，就认为你已经完成了代码编写。**这是远远不够**。你应该问问自己：“从现在起的 6 个月后，还能有人看的懂这些代码吗？”简单地说，编写整洁的代码的结果就是轻而易举地让别人看的懂你的代码。

<!-- more -->

> 编写整洁代码的方法：移除僵尸代码、重构并移除任何注释掉的代码。

为什么我极度关心整洁的代码呢？因为如果你是一个优秀的开发者，那么你一定是很会偷懒的。一个优秀的开发者在遇到**重复的事情**的时候，通常**会找到一个自动化的（或更好的）解决方案来完成手头的工作**。因为你懒，你不愿意重复劳动第二次，而使用编写整洁代码的技术能减少从 `Code Reviews` 中拉取和推送的次数，也不需要一遍又一遍的回到同一段代码。

### 1、 通过“气味测试”来判断整洁的代码

整洁的代码是能通过“气味测试”识别的，为什么我会这样说呢？有时当我们看着代码（自己的或者别人的）的时候，会不自觉的认为：“这里的代码似乎不太对”，这就是气味测试，你这个感觉可能是对的，也可能是不对的。无论如何都请你再好好思考一下，很有可能，你会找到一个更好地解决方案。

### 2、整洁的代码是 DRY 的

> DRY 是 `Don’t Repeat Yourself`，就是不要重复你自己。

如果你在多个地方重复做了相同的时候，那么你就应该提取并合并这些重复的代码。如果你在你的代码中发现一种模式，这就说明你正在 DRY。

举个栗子：

```javascript
// Dirty
const MyComponent = () => (
  <div>
    <OtherComponent type="a" className="colorful" foo={123} bar={456} />
    <OtherComponent type="b" className="colorful" foo={123} bar={456} />    
  </div>
);
```

```javascript
// Clean
const MyOtherComponent = ({ type }) => (
  <OtherComponent type={type} className="colorful" foo={123} bar={456} />
);
const MyComponent = () => (
  <div>
    <MyOtherComponent type="a" />
    <MyOtherComponent type="b" />
  </div>
);
```

有些时候，你重构了代码之后，会发现你的代码量反而多了，正如上面的例子一样，但是请注意，你还是要这样做，因为重构能帮助你提高你代码的**可维护性**。

值得注意的是，**不要过分的 DRY，要根据实际懂得取舍**。

### 3、整洁的代码是可预测和可测试的

可能你不要喜欢**编写单元测试**，但是你不得不去完成这项工作，这几乎是强制性的。因为你不能保证你的新特性在其他的地方也没有错误。

许多 React 开发人员依靠 [Jest](https://facebook.github.io/jest/) 来进行零配置测试运行，并生成代码覆盖率报告。如果您在比较测试之前/之后对视觉感兴趣，请查看 American Express 开发的 [Jest Image Snapshot](https://github.com/americanexpress/jest-image-snapshot)。

### 4、整洁的代码本身就是注释

你肯定有这样的经历，你写过一些代码，然后在这段代码上加上了密密麻麻的注释，以确保完全阐述了这段代码的功能。但是过了一段时间，一旦你发现你这段代码有一个错误，所以你不得不回去修改你的代码。这时你是否记得去修改你的注释以表达这段新的代码的逻辑呢？可能会，也可能不会。如果你没有修改，那么下一个看你代码的人可能会因为过于相信你的注释而再次掉入同样的错误。

**添加注释仅仅是为了解释清楚复杂逻辑**，所以请不要对简单的代码添加注释，会增加视觉上的混乱。

举个栗子：

```javascript
// Dirty
const fetchUser = (id) => (
  fetch(buildUri(`/users/${id}`)) // Get User DTO record from REST API
    .then(convertFormat) // Convert to snakeCase
    .then(validateUser) // Make sure the the user is valid
);
```

这里的注释主要是为了介绍每一行的函数做了什么，并没有什么意义，那么其实我们可以使用更加准确地命名来解释函数本身的功能。

```javascript
// Clean
const fetchUser = (id) => (
  fetch(buildUri(`/users/${id}`))
    .then(snakeToCamelCase)
    .then(validateUser)
);
```

### 5、命名注意事项

在作者的前一篇文章 [Function as Child Components Are an Anti-Pattern](http://americanexpress.io/faccs-are-an-antipattern)，它强调了命名的重要性，我们都应该认真考虑变量名称、函数名称甚至是文件名称。

这里有一些指导原则：

1. **对于布尔变量或者返回布尔值的函数，应该使用 is、has 或 should 开头**：

   ```javascript
   // Dirty
   const done = current >= goal;
   ```

   ```javascript
   // Clean
   const isComplete = current >= goal;
   ```

2. **对于函数的命名，应该是能表达这个函数做了什么，而不是如何做的**。换句话说，不要在名称中公布实现的细节，因为一旦你有一天重构了这个函数的内部实现方式，那么你也就不必更改你的函数名称。

   ```javascript
   // Dirty
   const loadConfigFromServer = () => {
     ...
   };
   ```

   ```javascript
   // Clean
   const loadConfig = () => {
     ...
   };
   ```

> 这里是 [Swift 官方的命名指导规范](https://swift.org/documentation/api-design-guidelines/#naming)，可以参考一下。

### 6、整洁的代码遵循成熟的设计模式和最佳实践

设计模式是前人在大量的问题中发现的，并提炼出的解决方案，遵循设计模式尽可能可以让你避免相同的错误。还有最佳实践，它们和设计模式类似，但是应用更加广泛。

**那么在构建 React 应用程序的时候，请遵循以下最佳实践：**

1. 每一个功能组件尽可能小，组件都是单一职责的，即请务必遵循**单一职责原则**。这样能确保每个功能很多的完成一项工作，同时也方便测试。
2. 请注意漏洞抽象，换言之，不要给你的代码的使用者强加内部需求。
3. 请**遵循严格的 lint 规则**，它能帮助你编写干净、一致的代码。

### 7、整洁的代码并不会消耗过长的时间

常常有人说：“编写整洁的代码会降低生产力”。这是不正确的。在你最初实践的时候，你可能会减慢你的编程速度，但是慢慢地，随着你的习惯，你的速度会因为你写更少的代码而提升。

因为你不需要写重复的代码，而且也不必花时间去修改 `Code Reviews` 中的注释。如果你把代码分解成了小的模块，每一个模块有自己单独的功能，那么你很可能再也不必去碰其中的大多数模块。时间就在这样 “编写随后忘记” 的流程中节约出来了。

## 实践演示

接下来是实践演示部分了。

### 1、DRY 这段代码

看看下面这段代码，你有没有从这段代码中发现什么模式呢？

```javascript
// Dirty
import Title from './Title';
export const Thingie = ({ description }) => (
  <div class="thingie">
    <div class="description-wrapper">
      <Description value={description} />
    </div>
  </div>
);
export const ThingieWithTitle = ({ title, description }) => (
  <div>
    <Title value={title} />
    <div class="description-wrapper">
      <Description value={description} />
    </div>
  </div>
);
```

没错，组件 `Thingie` 和 `ThingieWithTitle` 除了 title 之外都是相同的。很明显，我们可以使用 DRY 将其重构一下。

让我们传递一个 `children` 作为参数到 `Thingie`。在 `ThingieWithTitle` 中，使用 Thingie 来包装，并且将 `Title` 组件作为 `Thingie`的 `children` 传递回去。

```javascript
// Clean
import Title from './Title';
export const Thingie = ({ description, children }) => (
  <div class="thingie">
    {children}
    <div class="description-wrapper">
      <Description value={description} />
    </div>
  </div>
);
export const ThingieWithTitle = ({ title, ...others }) => (
  <Thingie {...others}>
    <Title value={title} />
  </Thingie>
);
```

### 2、默认值

来看下面这段代码，它使用了或操作符（`||`）来使当 `clasName` 为 undefined 或 null 时将其设置为 `icon-large`。

```javascript
// Dirty
const Icon = ({ className, onClick }) => {
  const additionalClasses = className || 'icon-large';
  return (
    <span
      className={`icon-hover ${additionalClasses}`}
      onClick={onClick}>
    </span>
  );
};
```

但是这是很老的做法了，现在我们可以使用 `ES6` 的语法，给参数设置一个默认值来代替或操作符的功能。这样操作后，我们还能利用 ES6 箭头函数的单一语句形式，就可以删掉 `return` 标识符。

```javascript
// Clean
const Icon = ({ className = 'icon-large', onClick }) => (
  <span className={`icon-hover ${className}`} onClick={onClick} />
);
```

在 React 中，我们有一种更整洁的版本，如下面代码所示，我们将箭头函数的参数中的默认值转移到了 `defaultProps` 属性中。

```javascript
// Cleaner
const Icon = ({ className, onClick }) => (
  <span className={`icon-hover ${className}`} onClick={onClick} />
);
Icon.defaultProps = {
  className: 'icon-large',
};
```

为什么这样操作是更整洁的呢？因为在 React 中，让 React 设置默认的 Props 能够产生更加高效的代码，因为类中的 `default props` 是基于组件的生命周期的，而且我们也能利用 `propTypes` 来检查默认值。此外，另一个好处是，它从组件本身的默认逻辑中解脱出来了。

举个栗子，你可以像下面代码一样，将所有的 `default props` 集中存储在一个地方，当然这里只是举个例子，并不推荐这样操作。

```javascript
import defaultProps from './defaultProps';
...
Icon.defaultProps = defaultProps.Icon;
```

### 3、把状态从渲染中分离出来

**将数据的加载逻辑与渲染（或是表现）逻辑放在一起或导致组件变得更加负责**，所以我们应该将数据的加载逻辑放在一个单独的容器组件中，这个组件具有单一职责，就是记载数据。然后编写另一个组件，它的单一职责就是渲染数据。这就是[容器模式](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)。

举个栗子：这段代码中，同一个组件包含了用户数据的加载和显示。

```javascript
// Dirty
class User extends Component {
  state = { loading: true };

  render() {
    const { loading, user } = this.state;
    return loading
      ? <div>Loading...</div>
      : <div>
          <div>
            First name: {user.firstName}
          </div>
          <div>
            First name: {user.lastName}
          </div>
          ...
        </div>;
  }

  componentDidMount() {
    fetchUser(this.props.id)
      .then((user) => { this.setState({ loading: false, user })})
  }
}
```

在整洁版本的代码中，我们将加载数据的逻辑和渲染逻辑分离开来，这样不仅使得代码更容易理解，而且更加便于测试，因为你可以单独测试每一个问题，而且此时的 RenderUser 组件**是无状态的功能组件**，其结果是可预测的。

```javascript
// Clean
import RenderUser from './RenderUser';
class User extends Component {
  state = { loading: true };

  render() {
    const { loading, user } = this.state;
    return loading ? <Loading /> : <RenderUser user={user} />;
  }

  componentDidMount() {
    fetchUser(this.props.id)
      .then(user => { this.setState({ loading: false, user })})
  }
}
```

### 4、使用无状态的功能组件

在 React v0.14.0 中，引入了无状态功能组件（SFC），它们被用来大大简化纯渲染（`render-only`）组件。 但是有些开发者仍使用过去的开发方式，如下代码：

```javascript
// Dirty
class TableRowWrapper extends Component {
  render() {
    return (
      <tr>
        {this.props.children}
      </tr>
    );
  }
}
```

让我们将其改写成无状态的功能组件。

从下面改写后的代码可以看出，整洁的版本清除了很多不必要的代码，而且通过 React 的核心优化，**去除了创建实例**这一步，可以减少内存的使用。

```javascript
// Clean
const TableRowWrapper = ({ children }) => (
  <tr>
    {children}
  </tr>
);
```

### 5、`Rest / Spread`

在一年前，大家都可能还很喜欢用 `Object.assign()` 方法，但是现在，我们可以使用 ES6 的 [Rest/Spread](https://github.com/tc39/proposal-object-rest-spread) 来代替。

> `Object.assign()` 方法用于将所有可枚举属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。
>
> Rest/ Spread 为对象解构赋值和对象文字的扩散属性引入了类似的静态属性。

举个栗子：当你把一个对象传递给一个组件时，你可能在这个组件中只使用对象的 `className` 属性，同时想把这个对象中除了 `className` 外的属性都传递给其他组件。那么你可能会这么做：

```javascript
// Dirty
const MyComponent = (props) => {
  const others = Object.assign({}, props);
  delete others.className;
  return (
    <div className={props.className}>
      {React.createElement(MyOtherComponent, others)}
    </div>
  );
};
```

这样操作真是太麻烦了，让我们使用 `Rest/Spread` 方案来试试看：

```javascript
// Clean
const MyComponent = ({ className, ...others }) => (
  <div className={className}>
    <MyOtherComponent {...others} />
  </div>
);
```

### 6、解构

ES6 为我们引入了 [解构](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) 的概念，这是一个非常 nice 的东西。解构能允许你将 **值从数组** 或 **属性从对象** 中提取到不同的变量中。

#### 对象解构

在下面的例子中，我们将 `newProps` 传递给 `componentWillReceiveProps` 函数，同时我们设置 `state.active`（其值源于 `newProps`）。

```javascript
// Dirty
componentWillReceiveProps(newProps) {
  this.setState({
    active: newProps.active
  });
}
```

在上面代码中，实际上，我们只是使用了 `newProps` 中一个属性，那么为什么要传入整个 `newProps` 对象呢？之传入 `newProps` 对象中的 `active` 属性的值不就好了。

让我们看看整洁的版本，我们将 `active` 从 `newProps` 对象中解构出来，所以我们不仅不需要应用 `newProps.active`，而且我们也可以使用 ES 对象属性简写的方式来设置 `setState` 。

```javascript
// Clean
componentWillReceiveProps({ active }) {
  this.setState({ active });
}
```

#### 数组解构

数组解构经常会被我们忽视，但这确实是一个非常有用的功能。如下面代码，我们按顺序从 `locale` 中提取 `language` 和 `country` 两个值。

```
// Dirty
const splitLocale = locale.split('-');
const language = splitLocale[0];
const country = splitLocale[1];
```

使用数组解构的话，我们可以这样操作，是不是更加简单了。

```
// Clean
const [language, country] = locale.split('-');
```

## 总结

这就是这篇文章的全部了，希望大家能写出整洁干净的代码，真的非常重要，祝好。