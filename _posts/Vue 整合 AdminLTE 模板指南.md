---
title: Vue 整合 AdminLTE 模板指南
author: Huanqiang
tags: [vue, AdminLTE]
categories: [前端]
keyword: [vue, AdminLTE]
date: 2017-08-22 10:31:49
---

本文讲述了如何在 Vue 中整合 AdminLTE 框架，相信其他 css 框架的整合方式也差不多，可以抛砖引玉。

### 第一步：创建项目

使用 `vue init webpack name` 创建项目。

本文使用的例子是把 AdminLTE 自带示例中的 `index.html` 的 `body` 内部代码（不含 `body` 标签）直接复制到 vue 项目中的 `App.vue` 的 `template` 部分（覆盖）。

<!-- more -->

### 第二步：引入 jquery

因为 bootstrap 和 AdminLTE 中很多 JS 都使用了 jquery，所以我们必须要引入 jquery。

引入 jquery 非常方便，首先使用如下命令行在本项目中安装 jquery：

```bash
npm install jquery --save
```

然后修改 webpack 的配置文件（`webpack.base.conf.js`）：

```javascript
var webpack = require('webpack')

// 然后在 module.exports = {} 中加入

plugins: [
	new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery',
      'window.jQuery': 'jquery'
    }),
]
```

### 第三步：引入 bootstrap 相关文件

将 bootstrap 包拷入 `src/assets` 目录下，然后在 `main.js` 文件加入下面代码：

```javascript
import './assets/bootstrap/css/bootstrap.css'  // css 文件
import './assets/bootstrap/js/bootstrap.min.js'  // JS 文件
```

此时目录结构如下:

```
|-- src/
|	|-- assets/
|	|	|-- bootstrap/      # 这个就是我们引入的 bootstrap 包
|	|	|	|-- css/
|	|	|	|-- fonts/
|	|	|	|-- js/
```

### 第三步：引入 AdminLTE 相关的css文件

将 AdminLTE 项目中的 `css、img、js` 文件复制到项目中。我们在 AdminLTE 项目的 `dist` 文件夹中能找到 `css、img、js` 三个文件夹。把这三个文件夹复制到我们 Vue 项目的 `assets/AdminLTE` 文件夹下。然后在 main.js 文件加入下面代码：

```javascript
// AdminLTE
import './assets/AdminLTE/css/AdminLTE.min.css'  // css 文件
import './assets/AdminLTE/css/skins/_all-skins.min.css'  // css 主题文件
import './assets/AdminLTE/js/app.min.js'   // JS 文件
```

此时目录结构如下:

```
|-- src/
|	|-- assets/
|	|	|-- bootstrap/
|	|	|-- AdminLTE/      # 这个文件夹下保存所有的 AdminLTE 的文件
|	|	|	|-- css/
|	|	|	|-- img/
|	|	|	|-- js/
```

### 第四步：引入 font-awesome 图标库

因为 bootstrap 和 AdminLTE 中使用了 font-awesome 图表库中的很多图标。所以我们还需要引入font-awesome。

```bash
npm install font-awesome --save
```

然后`main.js` 文件加入下面代码：

```javascript
// font-awesome
import 'font-awesome/css/font-awesome.min.css'
```

### 第五步：引入 body 的 class

最后，还要记得在 body 中引入 class，这个 class 控制了主题的颜色、侧边栏最小化，这些配置都可以在 AdminLTE 的文档中找到。

这里举一个栗子🌰：

```javascript
// 在 vue 项目最外部的 index.html 文件的 body 标签中加入下面 class 即可

class="hold-transition skin-blue fixed sidebar-mini"
```

### 参考资料

1. [Vue整合AdminLTE模板](http://www.itwendao.com/article/detail/243821.html)：本文即以此文为蓝本写的。
2. [vue-adminLte-vue-router](https://github.com/liujians/vue-adminLte-vue-router)：这个项目代码也是参照上文写的，两者联合参考，效果更佳