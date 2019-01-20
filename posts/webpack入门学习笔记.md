---
title: webpack入门学习笔记
date: 2017-11-12 15:20:08
tags:
- 前端构建
- webpack
---

## 为什么使用webpack
如今web应用的功能越来越丰富，在这些功能的实现的背后，拥有着复杂的javascript代码和依赖包。开发者为了简化开发的复杂度，前端开发者社区内出现了各种实践方法。
1. 模块化，让复杂的程序功能可以细分到每一个文件。
2. 扩展语言，使我们能够实现目前版本的JavaScript不能直接使用的特性，并且之后还能转换为JavaScript文件使浏览器可以识别。
3. css的预处理less、sass。
4. ...
这些改进确实大大的提高了我们的开发效率，但是利用它们开发的文件往往需要进行额外的处理才能让浏览器识别,而手动处理又是非常繁琐的，这就为webpack类的工具的出现提供了需求。
## 定义
webpack可以看成是一个前端代码的的模块打包器，它通过对项目结构以及依赖的分析，将一些浏览器不能解析执行的扩展语言（TypeScript、sass、less等）进行转换和分模块打包成可以浏览器解析引擎可以直接执行的javasript和css代码。
## 特性
### 代码切分
对于一个大项目来说，所有代码都打包在一个文件之内会大大增加应用的首屏加载时间，降低了用户体验。为了解决这个问题，代码切分会根据依赖关系树，将代码切分为同步和异步的块，把首屏不需要加载代码块切分到不同的文件内，按需加载需要的模块。
### 加载器
webpack除了加载javascript资源文件外，还可以通过特定的加载器把其他资源文件转化处理成javascript需要的内容。任何资源都可以通过加载器成为webpack加载的模块。
### 插件
webapck拥有丰富的插件系统，开发者可以根据自己的需求，为webpack打包配置添加不同的插件，达到不同的打包结果。

## 使用webpack
全局安装webpack
```npm
$ npm install wabpack -g
```
添加入口文件index.js
```javascript
document.write('我是入口文件');
```
添加html模板
```html
<html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <script type="text/javascript" src="bundle.js" charset="utf-8"></script> <!-- bundle.js为打包文件名 -->
    </body>
</html>
```
然后执行命令打包
```npm
$ webpack ./index.js ./bundle.js
```
成功后显示结果
```
Hash: 1ce33659f982ca43bf98
Version: webpack 3.8.1
Time: 73ms
    Asset    Size  Chunks             Chunk Names
bundle.js  2.5 kB       0  [emitted]  main
   [0] ./index.js 26 bytes {0} [built]
```
在浏览器中打开模板文件可以看到index.js的执行结果。

另外，可以通过新建配置文件webpack.config.js来进行打包
```javascript
module.exports = {
    entry: '.index.js',
    output: {
        path: __dirname,
        filename: "bundle.js"
    }
}
```
执行命令
```npm
$ webpack
```
效果与前者一样。
## 配置
通过配置文件可对打包结果进行定义。默认打包配置文件名为webpack.config.js
```javascript
const path = require('path');
module.exports = {
    // 入口entry string | object | array
    entry: {
        a: "./app/entry-a",
        b: ["./app/entry-b1", "./app/entry-b2"]
    },
    // 这里应用程序开始执行
    // webpack 开始打包
    output: {
        // webpack 如何输出结果的相关选项
        path: path.resolve(__dirname, "dist"), // string 所有输出文件的目标路径，必须是绝对路径（使用 Node.js 的 path 模块）
        filename: "bundle.[name].[chunkhash].js", // string [name]用于多个入口文件，[chunkhash]为当前chunk的chunkhash，用于解决缓存问题
        publicPath: '/', // 输出解析文件的目录，也可以理解为加载chunk文件的路径
        library: "MyLibrary", // string 导出库(exported library)的名称 webpack的另外一个作用，还可以用于打包JavaScript library。
        libraryTarget: "umd", // 导出库(exported library)的类型 'umd' 通用模块定义
        pathinfo: true, // boolean
        // 在生成代码时，引入相关的模块、导出、请求等有帮助的路径信息。
        chunkFilename: "[id].js",
        chunkFilename: "[chunkhash].js", // 长效缓存(/guides/caching)
        // 「附加分块(additional chunk)」的文件名模板
        jsonpFunction: "myWebpackJsonp", // string
        // 用于加载分块的 JSONP 函数名
        sourceMapFilename: "[file].map", // string
        sourceMapFilename: "sourcemaps/[file].map", // string
        // 「source map 位置」的文件名模板
        devtoolModuleFilenameTemplate: "webpack:///[resource-path]", // string
        // 「devtool 中模块」的文件名模板
        devtoolFallbackModuleFilenameTemplate: "webpack:///[resource-path]?[hash]", // string
        // 「devtool 中模块」的文件名模板（用于冲突）
        umdNamedDefine: true, // boolean
        // 在 UMD 库中使用命名的 AMD 模块
        crossOriginLoading: "use-credentials", // 枚举
        crossOriginLoading: "anonymous",
        crossOriginLoading: false,
        // 指定运行时如何发出跨域请求问题
        devtoolLineToLine: {
          test: /\.jsx$/
        },
        // 为这些模块使用 1:1 映射 SourceMaps（快速）
        hotUpdateMainFilename: "[hash].hot-update.json", // string
        // 「HMR 清单」的文件名模板
        hotUpdateChunkFilename: "[id].[hash].hot-update.js", // string
        // 「HMR 分块」的文件名模板
        sourcePrefix: "\t", // string
        // 包内前置式模块资源具有更好可读性
    },
    module:{
        // 关于模块配置
        rules: [
          // 模块规则（配置 loader、解析器等选项）
          {
            test: /\.jsx?$/,
            include: [
              path.resolve(__dirname, "app")
            ],
            exclude: [
              path.resolve(__dirname, "app/demo-files")
            ],
            // 这里是匹配条件，每个选项都接收一个正则表达式或字符串
            // test 和 include 具有相同的作用，都是必须匹配选项
            // exclude 是必不匹配选项（优先于 test 和 include）
            // 最佳实践：
            // - 只在 test 和 文件名匹配 中使用正则表达式
            // - 在 include 和 exclude 中使用绝对路径数组
            // - 尽量避免 exclude，更倾向于使用 include
            issuer: { test, include, exclude },
            // issuer 条件（导入源）
            enforce: "pre",
            enforce: "post",
            // 标识应用这些规则，即使规则覆盖（高级选项）
            loader: "babel-loader",
            // 应该应用的 loader，它相对上下文解析
            // 为了更清晰，`-loader` 后缀在 webpack 2 中不再是可选的
            options: {
              presets: ["es2015"]
            },
            // loader 的可选项
          },
          {
            test: /\.html$/,
            test: "\.html$"

            use: [
              // 应用多个 loader 和选项
              "htmllint-loader",
              {
                loader: "html-loader",
                options: {
                  /* ... */
                }
              }
            ]
          },
          { oneOf: [ /* rules */ ] },
          // 只使用这些嵌套规则之一
          { rules: [ /* rules */ ] },
          // 使用所有这些嵌套规则（合并可用条件）
          { resource: { and: [ /* 条件 */ ] } },
          // 仅当所有条件都匹配时才匹配
          { resource: { or: [ /* 条件 */ ] } },
          { resource: [ /* 条件 */ ] },
          // 任意条件匹配时匹配（默认为数组）
          { resource: { not: /* 条件 */ } }
          // 条件不匹配时匹配
        ],
        noParse: [
          /special-library\.js$/
        ],
        // 不解析这里的模块
    },
    resolve: {
        // 解析模块请求的选项
        // （不适用于对 loader 解析）
        modules: [
          "node_modules",
          path.resolve(__dirname, "app")
        ],
        // 用于查找模块的目录
        extensions: [".js", ".json", ".jsx", ".css"],
        // 使用的扩展名
        alias: {
          // 模块别名列表
          "module": "new-module",
          // 起别名："module" -> "new-module" 和 "module/path/file" -> "new-module/path/file"
          "only-module$": "new-module",
          // 起别名 "only-module" -> "new-module"，但不匹配 "only-module/path/file" -> "new-module/path/file"
          "module": path.resolve(__dirname, "app/third/module.js"),
          // 起别名 "module" -> "./app/third/module.js" 和 "module/file" 会导致错误
          // 模块别名相对于当前上下文导入
        },
    },
    devtool: "source-map", // enum
    // 通过在浏览器调试工具(browser devtools)中添加元信息(meta info)增强调试
    // 牺牲了构建速度的 `source-map' 是最详细的。
    context: __dirname, // string（绝对路径！）
    // webpack 的主目录
    // entry 和 module.rules.loader 选项
    // 相对于此目录解析
    target: "web", // 枚举
    // 包(bundle)应该运行的环境
    // 更改 块加载行为(chunk loading behavior) 和 可用模块(available module)
    externals: {
        jquery: 'window.$' // 遇到jquery模块时直接引用window.$
    },
    // 不要遵循/打包这些模块，而是在运行时从环境中请求他们

    plugins: [
        // ...
    ],
    // 附加插件列表

    cache: false, // boolean
    // 禁用/启用缓存
    watch: true, // boolean
    // 启用观察
}
```
以上为webpack常用配置。
