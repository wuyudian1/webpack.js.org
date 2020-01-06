---
title: 概念
sort: 0
contributors:
  - TheLarkInn
  - jhnns
  - grgur
  - johnstew
  - jimrfenner
  - TheDutchCoder
  - adambraimbridge
  - EugeneHlushko
  - jeremenichelli
  - arjunsajeev
  - byzyk
  - yairhaimo
  - farskid
  - LukeMwila
  - Jalitha
---

本质上，__webpack__ 是一个用于现代 JavaScript 应用程序的_静态模块打包工具_。当 webpack 处理应用程序时，它会在内部构建一个 [依赖图(dependency graph)](/concepts/dependency-graph/)，此依赖图会映射项目所需的每个模块，并生成一个或多个 _bundle_。

T> 可以在 [这里](/concepts/modules) 了解更多关于 JavaScript 模块和 webpack 模块的信息。

从 v4.0.0 开始，__webpack 可以不用再引入一个配置文件__来打包项目。然而，但它仍然有着 [高度可配置性](/configuration)，可以很好满足你的需求。

在开始前你需要先理解一些__核心概念__：

- [入口(entry)](#entry)
- [输出(output)](#output)
- [loader](#loaders)
- [插件(plugin)](#plugins)
- [模式(mode)](#mode)
- [浏览器兼容性(browser compatibility)](#browser-compatibility)

本文档旨在给出这些概念的__高度__概述，同时提供具体概念的详尽相关用例。

为了更好地理解模块打包工具背后的理念，以及在底层它们是如何运作的，请参考以下资源：

- [手动打包一个应用程序](https://www.youtube.com/watch?v=UNMkLHzofQI)
- [实时创建一个简单打包工具](https://www.youtube.com/watch?v=Gc9-7PBqOC8)
- [一个简单打包工具的详细说明](https://github.com/ronami/minipack)


## 入口(entry)

__入口起点(entry point)__指示 webpack 应该使用哪个模块，来作为构建其内部 *[依赖图(dependency graph)](/concepts/dependency-graph/)* 的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

默认值是 `./src/index.js`，但你可以通过在 [webpack configuration](/configuration) 中配置 __entry__ 属性，来指定一个（或多个）不同的入口起点。例如：

__webpack.config.js__

``` js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```

T> 在 [入口起点](/concepts/entry-points) 章节可以了解更多信息。


## 输出(output)

__output__ 属性告诉 webpack 在哪里输出它所创建的 _bundle_，以及如何命名这些文件。主要输出文件的默认值是 `./dist/main.js`，其他生成文件默认放置在 `./dist` 文件夹中。

你可以通过在配置中指定一个 `output` 字段，来配置这些处理过程：

__webpack.config.js__

```javascript
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

在上面的示例中，我们通过 `output.filename` 和 `output.path` 属性，来告诉 webpack bundle 的名称，以及我们想要 bundle 生成(emit)到哪里。可能你想要了解在代码最上面导入的 path 模块是什么，它是一个 [Node.js 核心模块](https://nodejs.org/api/modules.html)，用于操作文件路径。

T> `output` 属性还有 [许多可配置的特性](/configuration/output)，如果你想要了解其背后的概念，你可以通过 [阅读输出章节](/concepts/output) 来了解。


## loader

webpack 只能理解 JavaScript 和 JSON 文件。__loader__ 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 [模块](/concepts/modules)，以供应用程序使用，以及被添加到依赖图中。

W> 注意，loader 能够 `import` 导入任何类型的模块（例如 `.css` 文件），这是 webpack 特有的功能，其他打包程序或任务执行器的可能并不支持。我们认为这种语言扩展是很有必要的，因为这可以使开发人员创建出更准确的依赖关系图。

在更高层面，在 webpack 的配置中 __loader__ 有两个属性：

1. `test` 属性，用于标识出应该被对应的 loader 进行转换的某个或某些文件。
2. `use` 属性，表示进行转换时，应该使用哪个 loader。

__webpack.config.js__

```javascript
const path = require('path');

module.exports = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};
```

以上配置中，对一个单独的 module 对象定义了 `rules` 属性，里面包含两个必须属性：`test` 和 `use`。这告诉 webpack 编译器(compiler) 如下信息：

> “嘿，webpack 编译器，当你在`require()` 或者 `import` 申明中遇到指向'.txt'文件的路径时，先用`raw-loader` 将其转换一下，再将其（转换结果）添加到bundle 中。”

W> 重要的是要记住，在 webpack 配置中定义 rules 时，要定义在 `module.rules` 而不是 `rules` 中。为了使你受益于此，如果没有按照正确方式去做，webpack 会给出警告。

W> 请记住，使用正则表达式匹配文件时，你不要为它添加引号。也就是说，`/\.txt$/` 与 `'/\.txt$/'` 或 `"/\.txt$/"` 不一样。前者指示 webpack 匹配任何以 .txt 结尾的文件，后者指示 webpack 匹配具有绝对路径 '.txt' 的单个文件; 这可能不符合你的意图。

在使用 loader 时，可以阅读 [loader 章节](/concepts/loaders) 查看更深入的自定义配置。


## 插件(plugin)

loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。

T> 查看 [插件接口(plugin interface)](/api/plugins)，学习如何使用它来扩展 webpack 功能。

想要使用一个插件，你只需要 `require()` 它，然后把它添加到 `plugins` 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 `new` 操作符来创建它的一个实例。

__webpack.config.js__

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```

在上面的示例中，`html-webpack-plugin` 为应用程序生成 HTML 一个文件，并自动注入所有生成的 bundle。

T> webpack 提供许多开箱可用的插件！查阅 [插件列表](/plugins) 获取更多。

在 webpack 配置中使用插件是简单直接的。然而，也有很多值得我们进一步探讨的用例。[查看这里了解更多](/concepts/plugins)。


## 模式(mode)

通过选择 `development`, `production` 或 `none` 之中的一个，来设置 `mode` 参数，你可以启用 webpack 内置在相应环境下的优化。其默认值为 `production`。

```javascript
module.exports = {
  mode: 'production'
};
```

查看 [模式配置](/configuration/mode) 章节了解其详细内容和每个值所作的优化。


## 浏览器兼容性(browser compatibility)

webpack 支持所有符合 [ES5 标准](https://kangax.github.io/compat-table/es5/) 的浏览器（不支持 IE8 及以下版本）。webpack 的 `import()` 和 `require.ensure()` 需要 `Promise`。如果你想要支持旧版本浏览器，在使用这些表达式之前，还需要 [提前加载 polyfill](/guides/shimming/)。
