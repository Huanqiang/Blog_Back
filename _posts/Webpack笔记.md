---
title: Webpack笔记
author: Huanqiang
tags: [Vue, webpack]
categories: [前端]
keyword: [Vue, webpack]
date: 2017-02-11 08:31:49
---

webpack是现代 JavaScript 应用程序的模块捆绑器。它具有强大的可配置性。

在开始之前，我们先了解其四个核心概念。

<!-- more -->

## Entry（入口）

`webpack` 会创建你的应用的所有依赖的关系图。这个关系图的起点被称之为入口。入口会告诉 `webpack` 从哪里开始，并且根据依赖关系图来知道捆绑了哪些依赖。**我们可以将应用程序的入口点视为上下文的根或第一个启动应用程序的文件。**

在 `webpack` 中，我们使用 `webpack` 配置对象中的 `entry` 属性定义入口点。

### 单入口语法

这是最快速的方式，但是其可扩展性不高。

```javascript
entry: string|Array<string>
```

举个简单的栗子：

```javascript
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```

> 这里的 file.js 就是应用程序的入口文件；

### 对象语法

对象语法更详细。同时，这也是定义应用程序中入口点的可扩展性最高的方式。

```javascript
entry: {[entryChunkName: string]: string|Array<string>}
```

举个栗子：

```javascript
const config = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
};
```

### Multi Page Application

```javascript
const config = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```

**WHAT？**：我们告诉 webpack 我们想要3个独立的依赖关系图；

**WHY？**：在多页应用程序中，服务器将为您提取一个新的HTML文档。页面重新加载此新文档，并重新下载资源。然而，这给了我们独特的机会做多种事情：

- 使用 `CommonsChunkPlugin` 在每个页面之间创建一组共享应用程序代码。因为入口点的数量增加了，所以（重用了大量入口点之间代码/模块的）多页面应用程序可以极大地受益于这些技术。

## Output（输出）

虽然你将你的所有资源捆绑到一起了，但是我们仍然需要告诉 `webpack` 在哪里捆绑应用程序。 **webpack 的 output 属性描述了 webpack如何去处理捆绑的代码。**

举个简单的栗子：

```javascript
var path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

> 通过 `output.filename` 和 `output.path` 属性，我们描述了 `webpack` 中我们的捆绑包的名称以及路径。
> 其他可选参数参见：<https://webpack.js.org/concepts/output/>

## Loaders（加载）

加载器是应用于应用程序的资源文件的转换。它们是以资源文件的源作为参数并返回新源的函数（在Node.js中运行）。**webpack 将每一个文件（.css、.html、.jpg…）都视为一个模块。**`webpack` 中的加载器将这些文件转换为模块，并将其添加到依赖关系图中。但是 `webpack` 只能理解 JavaScript。

`webpack` 配置有两个目的：

1. 确定哪些文件应该被哪个特定的加载器转变（`test` 属性）。
2. 转变该文件，以便它能被添加带依赖关系图（和你的最终捆绑包）中。

```javascript
module: {
    rules: [
      {test: /\.(js|jsx)$/, use: 'babel-loader'}
    ]
  }
```

## Plugins（插件）

由于加载器（Loader）只能在每个文件的基础上执行转换，所以插件最常用的功能是在捆绑模块上的 “compilations” 或 “chunks” 上执行操作和自定义功能。webpack插件系统是非常强大和可定制。

为了使用插件，你只需要 `require()` 函数去加载它并将其添加到插件数组中。大多数插件可通过选项进行自定义。因为你可以在配置中多次使用插件用于不同的目的，你需要通过调用 `new` 来创建一个它的实例。

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      {test: /\.(js|jsx)$/, use: 'babel-loader'}
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```