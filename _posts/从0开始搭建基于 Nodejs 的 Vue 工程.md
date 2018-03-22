---
title: 从0开始搭建基于 Nodejs 的 Vue 工程
author: Huanqiang
tags: [vue, webapck]
categories: [前端]
keyword: [vue, webapck, 从0开始搭建基于 Nodejs 的 Vue 工程]
date: 2017-08-24 10:31:49
---

## 初始化项目

### 新建项目

首先，我们打开一个空的文件夹，使用 `npm init` 命令初始化项目，生成 `package.json` 文件

```
npm init
```

<!-- more -->

### 安装相应依赖包

接着我们安装一些常用的（必须要的，如前四个）依赖包。

```
# 1. 安装必须的 `webpack` 和 `webpack-dev-server`。
npm install webpack webpack-dev-server --save-dev

# 2. 安装 Vue 及其相关的解析包
npm install vue --save
# 解析包
npm install vue-loader vue-template-compiler --save-dev

# 3. 安装用于解析 ES6 的 babel 包。
npm install babel-loader babel-core babel-plugin-transform-runtime babel-preset-es2015 babel-runtime --save-dev

# 4. 安装 CSS 解析包.
npm install css-loader --save-dev

# 5. 安装 stylus 相关的解析包.
npm install stylus stylus-loaderr --save-dev
 
# 6. 安装图片及图标字体依赖的 loader
npm install url-loader file-loader --save-dev
```

### 新建并编写相关文件

在项目根目录下，新建 `build` 和 `src` 文件夹，并新建 `index.html` 文件。

在 `build` 文件夹下，新建 `webpack.config.js` 文件。

在 `src` 文件夹下，新建 `views` 文件夹，并新建 `App.vue` 和 `main.js` 文件。在  `views` 文件夹下，新建 `Favlist.js` 文件。

此时该项目的基本目录如下：

```
Project
    |-- package.json
    |-- index.html         // 启动页面
    |-- build
        |-- webpack.config.js    // webpack配置文件
    |-- src
        |-- views          // vue页面组件目录
        |-- main.js        // 入口文件
        |-- router.js      // vue-router配置
        |-- app.vue        // 工程首页组件
```

## 运行项目

首先，让我们简单编写一下 `index.html`、`main.js`、`App.vue` 和 `Favlist.vue` 这几个文件，使之满足简单运行即可，毕竟本文的重点还是配置 `Webpack`。

**1. index.html 文件**

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Vue + Webpack</title>
  </head>
  <body class="hold-transition skin-blue fixed sidebar-mini">
    Hello, Vue + Webpack 3.
    <br>
    <!-- 这里是 vue 文件的注入入口 -->
    <div id="app"></div>
  </body>
</html>
```

**2. main.js**

```
import Vue from 'vue'
import App from './App'

new Vue({
    el: '#app',
    render: h => h(App)
})
/**
此处还有一个写法如下：新的官方写法
new Vue({
  el: '#app',
  template: '<App/>',
  components: { App }
})
*/
```

**3. App.vue**

```
<template>
    <div>
        <favlist></favlist>
    </div>
</template>
<script>

import Favlist from './views/Favlist'

export default {
  components: {
      Favlist
  }
}
</script>
```

**4. Favlist.vue**

```
<template>
    <div>
        {{ value }}
    </div>
</template>
<script>
export default {
    data() {
        return {
            value: 'favlist'
        }
    }
}
</script>
```

接下来编写 Webpack 配置。这个就很重要了。

**webpack.config.js**

```
var path = require('path')

module.exports = {
    entry: './src/main.js',
    output: {
        filename: '[name].[hash].js',
        path: path.resolve(__dirname, '../dist/static'),
        publicPath: 'static'
    },
    resolve: {
        extensions: [' ', '.js', '.vue']
    },
    module: {
        rules: [{
            test: /\.vue$/,
            loader: 'vue-loader'
        }]
    }
}
```

这里简单的注解一下：

1. `entry`：配置入口文件；
2. `output`：配置打包后的文件信息，包括文件名、路径 
3. `resolve`：配置解析，自动解析文件后缀名。
4. `module`：配置加载器，因为 `Webpack` 本身只能识别 `JavaScript`，所以这里别的文件需要被指定识别，以便 `Webpack` 能够被添加到依赖图中。

Nice，完成以上操作后，你已经能将当前的这份简单的Vue项目打包，那么让我们通过 `Webpack`，先完成打包操作。打开 `package.json` 文件，在 script 属性中添加一下代码：

```
"build": "webpack --progress --colors --config build/webpack.config.js"
```

然后在命令行中执行该脚本：

```
npm run build
```

我们就能在 `dist` 文件夹（之前没有的话就自动新建的）中找到新生成的打包好的 js 文件。然后让我们将生成的 js 文件引入 `index.html` 文件中：

```
// 这里只展示了body部分
<body>
    Hello, Webpack 3.
    <br>
    <div id="app"></div>
    <script src="./dist/main.0571634209939ef1ff48.js"></script>
</body>
```

如此，让我们在浏览器中打开 `index.html` 文件，就能看到结果了。Interesting，很容易地，我们完成了最简单的打包操作。

### 自动注入打包文件

但是，我们也发现，现在我们的操作是很复杂的，在打包完后还要手动引入到 index.html 文件中。那么能不能自动将打包生成的 js 文件注入呢？当然可以，这就需要用到 `html-webpack-plugin` 插件。

首先我们要安装 `html-webpack-plugin` 插件。

```
npm install html-webpack-plugin --save-dev
```

然后打开 `webpack.config.js` 文件，在 `module` 下方（并列同一级别），添加该插件：

```
plugins: [
       new HTMLWebpackPlugin({
           filename: path.resolve(__dirname, '../dist/index.html'),
           template: path.resolve(__dirname, '../index.html'),
           inject: true
       })
   ]
```

此时，在运行 `npm run build` 就会在 `dist` 文件夹中发现新打包生成的 js 文件和注入了 js 的新的 `index.html` 文件。然后在浏览器中打开 `index.html` 文件，就能看到结果了。

### 热加载

虽然我们通过 `html-webpack-plugin` 完成了 js 的自动注入，但是每次构建打包后，我们仍要手动打开 html 文件查看结果，要知道我们用 vue-cli 生成的项目中在执行启动脚本后就能直接查看到结果了的，这又是怎么完成的呢？

其实也不难，在 vue-cli 生成的项目中，它是通过 express 来建立了一个小的服务器，然后让网站运行这个小的服务器上。好了，让我们来实现这个吧。

首先，我们需要安装一个 node.js 框架：`express`

```
npm install express --save-dev
```

然后，再安装两个中间件：`webpack-dev-middleware` 和 `webpack-hot-middleware`.

```
npm install webpack-dev-middleware webpack-hot-middleware --save-dev
```

> `webpack-dev-middleware`：能处理 Webpack 编译后输出的静态资源。
> `webpack-hot-middleware`：能实现浏览器的无刷新更新，就也是热更新。

接下来，在 `build` 文件夹下创建 `dev-server.js` 文件。

```
var express = require('express')
var webpack = require('webpack')
var config = require('./webpack.config')
var devMiddleware = require('webpack-dev-middleware')

// 创建 express 实例
var app = express()

// 将我们之前写的 config 配置到 webpack 中
var compiler = webpack(config)

// 将 `webapck-dev-middleware` 中间件注册到 express 实例中
app.use(devMiddleware(compiler, {
    publicPath: config.output.publicPath,
    stats: {
        colors: true
    }
}))

app.listen(8888, function (err) {
    if (err) {
        console.log(err)
        return
    }
    console.log('listening at http://localhost:8888')
})
```

然后打开 `package.json` 文件，在 script 属性中添加一下代码：

```
"dev": "node build/dev-server.js"
```

然后在命令行中执行该脚本：

```
npm run dev
```

再在浏览器中直接输入：`http://localhost:8888/index.html`，理论上就能访问到结果了，但是现在你会得到一个 `Cannot GET /index.html` 结果。这是因为当前的 webpack 的配置为发布版本而设置的，所以我们编译出来的会有问题。

现在，让我们来修复这个错误。打开 `webpack.config.js` 文件，修改 `output -> publicPath` 为 `/`，修改 `plugins -> new HTMLWebpackPlugin -> filename` 为 `index.html`。

```
output: {
       filename: '[name].[hash].js',
       path: path.resolve(__dirname, '../dist/static'),
       publicPath: '/'   // TODO: 这个又有什么用？？？
   },
```

```
plugins: [
       new HTMLWebpackPlugin({
           filename: 'index.html',
           template: path.resolve(__dirname, '../index.html'),
           inject: true
       })
   ]
```

然后你在执行 `npm run dev` 脚本，就能成功了！

#### 分离开发和生产配置

就像刚刚一样，当我们在开发模式和生产模式切换时，需要手动修改 `webpack-config.js` 文件，这就很麻烦了。所以我们需要将开发模式和生产模式的配置分文件分离，`Webpack` 提供了多环境配置的思路：

1. 编写一个基本配置文件（`webpack.base.conf.js`）
2. 配置指定环境下的配置文件，并使用 `webpack-merge` 来合并基本配置文件。

首先，我们先引入 `webpack-merge`，它用于合并基础配置文件。

```
npm install webpack-merge --save-dev
```

然后我们创建 `webpack.base.conf.js`、`webpack.dev.conf.js` 和 `webpack.prod.conf.js` 文件。并把原 `webpack.config.js`文件中关于开发模式和生产模式的公共配置复制到 `webpack.base.conf.js` 中。

**webpack.base.conf.js 文件**

```
var path = require('path')

function resolve(dir) {
    return path.resolve(__dirname, dir)
}

module.exports = {
    entry: './src/main.js',
    output: {
        filename: '[name].js',
        path: resolve('../dist/static'),
        publicPath: '/',
        chunkFilename: '[id].[chunkhash].js'
    },
    resolve: {
        extensions: [' ', '.js', '.vue']
    },
    module: {
        rules: [{
            test: /\.vue$/,
            loader: 'vue-loader'
        }, {
            test: /\.js$/,
            loader: 'babel-loader'
        }]
    }
}
```

然后我们编写一下 `webpack.dev.conf.js` 文件。

```
var path = require('path')
var merge = require('webpack-merge')
var webpack = require('webpack')
var HTMLWebpackPlugin = require('html-webpack-plugin')
var baseWebpackConfig = require('./webpack.base.conf')

module.exports = merge(baseWebpackConfig, {
    output: {
        path: path.resolve(__dirname, '../dist/static'),
        publicPath: '/',
    },
    plugins: [
        new HTMLWebpackPlugin({
            filename: 'index.html',
            template: path.resolve(__dirname, '../index.html'),
            inject: true
        })
    ]
})
```

这样就完成了开发模式基本的的配置文件（至于生产环境下的，我们稍后来完成）。

然后再把 `dev-server.js` 中的

```
var config = require('./webpack.config')
```

更换为

```
var config = require('./webpack.dev.conf')
```

这样，当我们运行 `npm run dev` 就能看到之前的结果了。

#### 热更新

现在我们已经能配置多份配置文件了，让我们继续原来的目标：**热更新**。

首先，在 `webpack.dev.conf.js` 文件中，我们添加两个插件，并添加热更新的入口。

```
var path = require('path')
var merge = require('webpack-merge')
var webpack = require('webpack')
var HTMLWebpackPlugin = require('html-webpack-plugin')
var baseWebpackConfig = require('./webpack.base.conf')

// 添加热修复时候的入口
Object.keys(baseWebpackConfig.entry).forEach(function(name) {
    baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])    
});

module.exports = merge(baseWebpackConfig, {
    output: {
        path: path.resolve(__dirname, '../dist/static'),
        publicPath: '/',
    },
    plugins: [
        // 添加热更新的插件
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoEmitOnErrorsPlugin(),

        new HTMLWebpackPlugin({
            filename: 'index.html',
            template: path.resolve(__dirname, '../index.html'),
            inject: true
        })
    ]
})
```

这里有一个细节：原来我们的入口配置是这样的：

```
entry: './src/main.js',
```

现在要改为对象语法：

```
entry: {
	app: './src/main.js'
},
```

> 这是因为[对象语法是 webpack 中最可扩展的方式](https://doc.webpack-china.org/concepts/entry-points/#-)；

然后在 `dev-server.js` 文件中，我们添加 `webpack-hot-middleware`。

```
var hotMiddleware = require('webpack-hot-middleware')

// 注册并使用热修复中间件
app.use(hotMiddleware(compiler))
```

现在当你重启项目之后，你就已经实现了热更新了，但是仍然还有一个小小的问题，就是每次更新代码之后，你必要手动刷新一下页面才能触发效果，让我们来处理掉这个问题。

### 其他配置

#### CSS 支持

安装 `css-loader` 解析包；

在 `webpack.base.conf.js` 的 `module` 中引入配置：

```
{
	test: /\.css$/,
	loader: 'css-loader'
}
```

#### 图片图标 支持

安装 `url-loader` 和 `file-loader` 解析包；

在 `webpack.base.conf.js` 的 `module` 中引入配置：

```
{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    use: [{
        loader: "url-loader",
        options: {
            limit: 10000,
            name: 'images/[name].[hash:7].[ext]'    // 将图片都放入images文件夹下，[hash:7]防缓存
        }
    }]
},
{
    test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
    use: [{
        loader: "url-loader",
        options: {
            limit: 10000,
            name: 'fonts/[name].[hash:7].[ext]'    // 将字体放入fonts文件夹下
        }
    }]
}
```

#### 压缩 JS

首先完成配置 `babel`，可以将 ES6 的语法转化为 ES5。新建 `.babelrc`。

```
{
    "presets": ["es2015"],
    "comments": false
}
```

然后配置 `webpack.base.conf.js` 的 `module`：

```
{
    test: /\.js$/,
    use: "babel-loader",
    // 这里需要注意，我们之前用的 exclude，现在改用 include
    include: [path.resolve(__dirname, 'src')]
}
```

> 因为 Webpack 更倾向于使用 include。

然后在生产环境配置文件中，引入插件 `webpack.optimize.UglifyJsPlugin`：

```
new webpack.optimize.UglifyJsPlugin()
```

#### 提取CSS

我们使用 `extract-text-webpack-plugin` 插件来提取 CSS。首先我们安装该依赖包：

```
// 安装插件
npm i extract-text-webpack-plugin -D
```

然后修改 css 和 less 的 loader 配置：

```
// 引入插件
var ExtractTextPlugin = require('extract-text-webpack-plugin')

// module：
{
    test: /\.css$/,
    use: ExtractTextPlugin.extract({
        use: "css-loader"
    })
}
```

然后在生产环境配置文件中新增插件：

```
// 提取 CSS 为单文件
new ExtractTextPlugin('../[name].[contenthash].css'),
```

这样就 OK 啦；

#### 代码分割

参照 `vue-cli` 及 `webpack` 文档的提取公共代码的方式，利用插件`webpack.optimize.CommonsChunkPlugin` 将来自 `node_modules`下的模块提取到 `vendor.js`（一般来讲都是外部库，短时间不会发生变化）。于是有如下代码：

```
new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks: function(module, count) {
        return module.resource && /\.js$/.test(module.resource) && module.resource.indexOf(path.join(__dirname, '../node_modules')) === 0
    }
})
```

如果你查看未压缩版的 `vendor.js`，会发现不仅包含 vue 的代码，还包含了`webpackBootstrap`（webpack模块加载器）的代码，按理说放在 vendor 里面也没啥问题，毕竟后面的模块都需要依赖于此，但是如果你的 chunk 使用了 hash，一旦app 代码发生了改变，相应的 hash 值会发生变化，`webpackBootstrap` 的代码也会发生变化（如图），而我们提取 vendor 的初心就是这部分代码改变频率不大，显然这不符合我们的目标，那么应当将 `webpackBootstrap` 再提取到一个文件中，代码如下：

```
new webpack.optimize.CommonsChunkPlugin({
    name: 'manifest',
    chunks: ['vendor']
})
```

## 资料

1. [基于webpack和vue.js搭建的H5端框架(其实主要用于Hybrid开发H5端框架，但是依然能够作为纯web端使用)](http://hcysun.me/2016/03/25/%E5%9F%BA%E4%BA%8Ewebpack%E5%92%8Cvue.js%E6%90%AD%E5%BB%BA%E7%9A%84H5%E7%AB%AF%E6%A1%86%E6%9E%B6(%E5%85%B6%E5%AE%9E%E4%B8%BB%E8%A6%81%E7%94%A8%E4%BA%8EHybrid%E5%BC%80%E5%8F%91H5%E7%AB%AF%E6%A1%86%E6%9E%B6%EF%BC%8C%E4%BD%86%E6%98%AF%E4%BE%9D%E7%84%B6%E8%83%BD%E5%A4%9F%E4%BD%9C%E4%B8%BA%E7%BA%AFweb%E7%AB%AF%E4%BD%BF%E7%94%A8)/)： 本文的主要参考，从逻辑的角度来配置，值得参考，只不过用的 vue 和 webpack 版本有点老了；
2. [从零搭建vue2+vue-router2+webpack3工程](http://www.qinshenxue.com/article/20161118151423.html)：里面的东西还是比较新的；
3. [webpack 中文文档](https://doc.webpack-china.org/concepts/)