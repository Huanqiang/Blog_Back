---
title: React 官方入门案例小结
author: Huanqiang
tags: [React]
categories: [前端]
keyword: [React, 官方案例]
date: 2017-08-25 10:31:49
---

练手了 React 官方入门案例，从其中展示的语法角度进行总结一下。

### JSX

JSX 是一个好东西，很方便。JSX 允许使用了 XML 来编写 HTML 标签，如下图：

```
let htmlTags = <div className="list">Hello React</div>
```

那么经过编译后就会转变成 JS 语法：

```javascript
let htmlTags = React.createElement('div', {className: 'list'}, 'Hello React')
```

在 JSX 中，它允许在包（大括号 `{}`）中使用表达式，比如 `2+2`、`this.state.value` 或 `setName('xx')` 等。

<!-- more -->

值得注意的是，JSX 使用的是 驼峰使命名方案，比如 `class -> className`，又如：`tabindex -> tabIndex`。

### 值存储

如果想要定义一个组件中的变量，可以在其构造器中的 state 里面。

```javascript
constructor() {
	super()
	this.state = {
		key: value
	}
}
```

然后我们就可以使用 `this.state.key` 来使用这个变量了。

### 值传递

父组件和子组件之间的值传递是通过 `props` 来完成的。举个例子：

比如父组件中传递一个 value 值：

```javascript
<SubComponent propValue={this.state.value} />
```

那么子组件中就可以直接通过 `this .props.propValue` 来使用了。

> `props` 是不可变的，子组件不应该尝试去改变 `props` 中被传递的变量的值。

### 操作传递

一般我们都需要在父组件上对子组件进行操作的交互，也是通过 `this.props` 来传递的。举个例子，父组件上想控制子组件上的一个按钮的操作。

父组件：

```javascript
handleClick(index) {
	// 操作
}

<SubComponent userDefinedClick={(index) => this.handleClick(index)} />
```

子组件：

```javascript
class SubComponent extends React.Component {
	render() {
		return (
			// 这里的 customClick 名字只要和父组件中传入时候的相同即可
			<button onClick="this.props.userDefinedClick()"></button>
		)
	}
}
```

### React 中的组件

> 组件的目的是为了将视图切割成可重复使用的部分，并能单独考虑设计每一个部分；

通常，React 组件都是继承了 `React.Component` 的子类，它通过 render() 函数来实现视图的渲染。

但是，当你只需要一个纯粹的组件时（比如，你这个组件只包含一个 render() 函数），那么我们可以使用函数式组件。比如

```javascript
function funcComponent(props) {
	return (
		<div className="list">{ props.value} <div>
	)
}
```

在函数时组件中，我们需要用 `props` 传参代替 `this.props.props` 来进行父子组件的数据传递。其实，所谓的函数式组件，就是一个普通的函数，只不过他返回了 HTML。

对于简单的组件，React 建议使用函数式组件，因为 React 对其有很好的优化。