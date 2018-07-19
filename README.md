[![webpack](https://i.loli.net/2018/07/19/5b505da091f7f.png)](https://i.loli.net/2018/07/19/5b505da091f7f.png)

本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(`module bundler`)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(`dependency graph`)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个`bundle`。

本文是按照webpack官网[指南](https://www.webpackjs.com/guides/)逐步操作的，既有官网上的[代码实例](https://github.com/wnbupt/webpack-starter)，也有自己简化的认识，希望能给在读的webpack新司机带来一点小小的启发💗

## stage-0: 开始使用webpack

首先按照官网创建一个最基本的webpack demo，代码请戳[这里](https://github.com/wnbupt/webpack-starter)。

```bash
$ git clone git@github.com:wnbupt/webpack-starter.git
$ git checkout stage-0 && npm i
```

目录结构如下：

```
webpack-starter
  |- package.json
  |- /dist
    |- index.html
  |- /src
    |- index.js
```

执行以下命令创建你的第一个`bundle`吧～

```bash
$ npx webpack
```

webpack默认会把`src`目录下的所有js打包，并在`dist`目录下生成`main.js`。

打开`index.html`可以看到`Hello webpack`，恭喜你，第一个demo运行成功啦～

### tips：npx命令是何方神圣？

在 npm@5.2.0 中，出现了一个新的工具，npx。

#### 更快捷的运行本地npm包

很多时候，当我们需要使用 `gulp` 或 `mocha` 等工具的时候，我们都不会把它们直接安装在全局环境下，而是作为 `devDependencies` 安装在本地，使用 `node_modules/.bin/xxx` 命令来启动。

但是现在，我们可以用 npx 实现同样的功能。只需在 npx 命令后面加上你想要运行的 npm 包的名字，就像 `npx gulp` ，就能运行本地 npm 包了。从此可以跟 `node_modules/.bin/xxx` 说再见了👋

#### 执行一次性命令

有时我们会用一些脚手架快速生成一个项目，例如 `create-react-app` 。但是你不想为了仅仅使用一次就在全局安装它，或者说每次使用前还要去更新，如果有一种方法能够在使用的时候才去安装它，并且每次安装的默认都是最新版本的，用完以后就自动把它删了，是不是会省事很多？

这时候 npx 就能满足你的需求。假设你想使用 `create-react-app` 来新建一个项目，不需要全局安装 `create-react-app` ，只需运行 `npx create-react-app my-new-app`就可以了。

## stage-1: 使用配置文件

```bash
$ git checkout stage-1 && npm i
```

目录结构如下，区别在于在根目录下增加了`webpack.config.js`配置文件：

```
webpack-starter
  |- package.json
+ |- webpack.config.js
  |- /dist
    |- index.html
  |- /src
    |- index.js
```

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js', // 指定入口文件
  output: {
    filename: 'bundle.js', // 指定打包文件名称
    path: path.resolve(__dirname, 'dist') // 指定打包文件所在目录，若不存在会自动新建
  }
};
```

指定配置文件打包

```bash
$ npx webpack --config webpack.config.js
```

执行完毕后可以发现`dist`目录下增加了`bundle.js`。

## stage-2: 添加新的资源

```bash
$ git checkout stage-2 && npm i
```

在上一步的基础上，我们增加了其他类型的资源文件，如`css`和`jpg`，并引用对应的`loader`对新加载的资源文件进行打包处理，官网上还有加载字体和数据，这里就不赘述了。

目录结构如下，区别在于在`src`目录下增加了两个资源文件：

```
webpack-starter
  |- package.json
  |- webpack.config.js
  |- /dist
    |- index.html
  |- /src
    |- index.js
+   |- avatar.jpg
+   |- style.css
```

在`package.json`中添加`webpack`快捷命令

```
"scripts": {
    "build": "webpack --config webpack.config.js",
    "test": "echo \"Error: no test specified\" && exit 1"
}
```

```bash
$ npm run build
```

执行完毕后，可以发现在`dist`目录下除了我们的老朋友`bundle.js`，还有一个名字很长的图片，内容和我们的`avatar.jpg`一摸一样。

**走到这里，我们才充分认识到了webpack的魅力，除了js，css & images & fonts & 各种资源 都可以被统一打包成js模块并有序组织在页面中，至于这是怎么实现的，笔者将在 深入webpack的博文中会尽力解答～**

## stage-3: 输出管理

乘坐时光机我们倒回到stage-1看看，这时`src`目录下只有一个js文件，倘若这时有两个js文件，那么打包之后又是什么样子呢？合并成一个`bundle`还是拆分为两个`bundle`？

```bash
$ git checkout stage-3 && npm i
```

这时我们的`webpack.config.js`变动如下：

```javascript
const path = require('path');

  module.exports = {
-   entry: './src/index.js',
+   entry: {
+     app: './src/index.js',
+     print: './src/print.js'
+   },
    output: {
-     filename: 'bundle.js',
+     filename: '[name].bundle.js',  // name为entry数组中的key值，生成关系一一对应
      path: path.resolve(__dirname, 'dist')
    }
  };
```

执行完毕后，在`dist`目录下出现了`app.bundle.js`和`print.bundle.js`。但是在这个阶段有一个弊端，就是我需要手动修改`index.html`中对于js的引用。如果webpack能帮我自动修改那就太好了！

## stage-4: 页面自动引入bundle

```bash
$ git checkout stage-4 && npm i
```

这时我们的`webpack.config.js`变动如下：

```javascript
  const path = require('path');
+ const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
+   plugins: [
+     new HtmlWebpackPlugin({
+       title: '输出管理'
+     })
+   ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

执行完毕后，在`dist`目录下除了`app.bundle.js`和`print.bundle.js`，还有`index.html`，自动引入了前两个bundle.js。这一切都是`plugin`的魔法～

## stage-5: 开发环境搭建

现在我们来看看如何建立一个开发环境，使我们的开发变得更容易一些？

### 使用sourcemap

当 webpack 打包源代码时，可能会很难追踪到错误和警告在源代码中的原始位置。例如，如果将三个源文件（a.js, b.js 和 c.js）打包到一个 `bundle`中，而其中一个源文件包含一个错误，那么堆栈跟踪就会简单地指向到bundle.js。这并通常没有太多帮助，因为你可能需要准确地知道错误来自于哪个源文件。

具体设置可以查看[这里](https://www.webpackjs.com/configuration/devtool/)

### 使用观察模式

webpack 中有几个不同的选项，可以帮助你在代码发生变化后自动编译代码
* webpack's Watch Mode
* webpack-dev-server
* webpack-dev-middleware
多数场景中，你可能需要使用 `webpack-dev-server`

```bash
$ git checkout stage-5 && npm i
```

这时`webpack.config.js`内容如下：

```javascript
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
+   devtool: 'inline-source-map',
+   devServer: {
+     contentBase: './dist' // 告诉开发服务器(dev server)在哪里查找文件
+   },
    plugins: [
      new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Development'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

```bash
$ npm run start
```

默认开启本地8080端口，打开http://localhost:8080/，即可看到我们的页面，修改`index.js`时页面会自动更新。

## stage-6: 生产环境搭建

### 分离的webpack配置文件

开发环境(development)和生产环境(production)的构建目标差异很大。在开发环境中，我们需要具有强大的、具有实时重新加载(live reloading)或热模块替换(hot module replacement)能力的 source map 和 localhost server。而在生产环境中，我们的目标则转向于关注更小的 bundle，更轻量的 source map，以及更优化的资源，以改善加载时间。由于要遵循逻辑分离，我们通常建议为每个环境编写彼此独立的 webpack 配置。

```bash
$ git checkout stage-6 && npm i
```

我们的目录结构如下：

```
webpack-demo
  |- package.json
- |- webpack.config.js
+ |- webpack.common.js
+ |- webpack.dev.js
+ |- webpack.prod.js
  |- /dist
  |- /src
    |- index.js
    |- print.js
  |- /node_modules
```

```
"scripts": {
    "start": "webpack-dev-server --open --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js"
}
```

### 公共依赖的合并

代码分离是 webpack 中最引人注目的特性之一。此特性能够把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。代码分离可以用于获取更小的 bundle，以及控制资源加载优先级，如果使用合理，会极大影响加载时间。

有三种常用的代码分离方法：
* 入口起点：使用 entry 配置手动地分离代码。
* 防止重复：使用 SplitChunksPlugin（替换原有CommonsChunkPlugin） 去重和分离 chunk。
* 动态导入：通过模块的内联函数调用来分离代码。

第一种很容易理解，第二种的配置如下：

```js
optimization: {
    splitChunks: {
      chunks: 'initial', // 只对指定的entry文件处理
      cacheGroups: {
        commons: {
          name: "commons",
          chunks: "initial",
          minChunks: 2
        }
      }
    }
  },
```

使用前后可以发现包大小由于common部分的抽离缩小了很多～

[![屏幕快照 2018-07-19 下午5.21.34.png](https://i.loli.net/2018/07/19/5b505850760ae.png)](https://i.loli.net/2018/07/19/5b505850760ae.png)


[![屏幕快照 2018-07-19 下午5.21.53.png](https://i.loli.net/2018/07/19/5b5058508c514.png)](https://i.loli.net/2018/07/19/5b5058508c514.png)

第三种应该大多数人不会使用到。

### 代码优化工具

推荐
* webpack-chart: webpack 数据交互饼图。
* webpack-visualizer: 可视化并分析你的 bundle，检查哪些模块占用空间，哪些可能是重复使用的。
* webpack-bundle-analyzer: 一款分析 bundle 内容的插件及 CLI 工具，以便捷的、交互式、可缩放的树状图形式展现给用户。

## Reference
* https://www.webpackjs.com/guides/
* http://www.nicong.xyz/2017/07/15/npx%E4%BD%BF%E7%94%A8%E7%AE%80%E4%BB%8B/
