---
title: 如何在Webpack中引入CSS
author: Huanqiang
tags: [CSS, webpack]
categories: [前端]
keyword: [CSS, webpack]
date: 2017-02-27 08:31:49
---

## 引入 CSS 模块

在 webpack 管理的项目中加载 css 的两种方式

<!-- more -->

### 1. 导入 css

在 js 文件（模块）中，使用

```bash
import './path/name.css'
```

### 2. 使用 `css-loader`

在 `webpack.config.js` 中配置 `css-loader`。

```javascript
module.exports = {
	module: {
		rules:[{
			test: /\.css$/,
			use: 'css-loader'
		}]
	}
}
```

但是这样有一个缺点：就是无法利用浏览器异步和并行加载 css 的能力。而你的网页不得不等待，直到整个 `JavaScript` 包加载完全。

为了解决这个问题， `s` 使用 `ExtractTextWebpackPlugin` 来分别加载 `css`。

#### 使用 `ExtractTextWebpackPlugin`

加载 `ExtractTextWebpackPlugin` 包：

```bash
npm i --save-dev extract-text-webpack-plugin@beta
```

使用 `ExtractTextWebpackPlugin` 的方法：

```javascript
module.exports = {
    module: {
         rules: [{
             test: /\.css$/,
-            use: 'css-loader'
+            use: ExtractTextPlugin.extract({
+                use: 'css-loader'
+            })
         }]
     },
+    plugins: [
+        new ExtractTextPlugin('styles.css'),
+    ]
}
```