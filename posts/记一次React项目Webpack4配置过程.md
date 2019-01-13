---
title: 记一次React项目Webpack4配置过程
date: 2018-08-16 19:30:30
tags:
- 前端构建
- webpack
- react
---

## 引言

最近新项目过多，在新项目中每次使用 webpack 都是拷贝之前的项目的配置文件过来，改改直接使用，很多配置还是一知半解，一直想用心的从头配置一次 webpack，加深对 webpack 的理解，所以，有了本文，先献上以下内容[github地址](https://github.com/yacan8/webpack-config-tool)。

## 基本配置 webpack.base.config.js

首先，配置entry

```js
const base = {
  entry: ['./src/index']
}
```

自 webpack4 起，webpack 提供了默认 entry，也就是我们上面使用的 `'./src/index'`，这里我们用数组包裹一下，方便动态增删，往下

配置 output：

```js
const base = {
  entry: ['./src/index'],
  output: {
    publicPath: '/', // 项目根目录
    path: path.resolve(__dirname, './dist'),
    chunkFilename: '[name].[chunkhash].chunk.js'
  }
}
```

配置 resolve.extensions， require的时候省略文件后缀

```js
const base = {
  resolve: {
    extensions: [".js", ".json"],
  }
}
```

配置 devServer，开发环境 webpack-dev-server 配置使用

```js
const host = 'localhost';
const port = 8080;

const base = {
  devServer: {
    contentBase: [path.join(process.cwd(), './vendor-dev/'), path.join(process.cwd(), './vendor/')], // dllPlugin使用，下文有讲
    hot: true,  // 热加载
    compress: false,
    open: true,  // 
    host: host,
    port: port,
    disableHostCheck: true, // 跳过host检测
    stats: { colors: true },
    filename: '[name].chunk.js',
    headers: { 'Access-Control-Allow-Origin': '*' }
  }
}
```

根据不同的环境，我们需要对默认的 entry 进行处理，如下

```js
const CleanWebpackPlugin = require('clean-webpack-plugin');
const isDebug = process.env.NODE_ENV !== 'production';

if (isDebug) {
  base.entry.unshift(`webpack-dev-server/client?http://${host}:${port}`, 'webpack/hot/dev-server'); // 添加devServer入口
  base.plugins.unshift(new webpack.HotModuleReplacementPlugin()); // 添加热加载
  base.devtool = 'source-map';
} else {
  base.entry.unshift('babel-polyfill');  // 加入 polyfill
  base.plugins.push(new CleanWebpackPlugin(   // 清理目标目录文件
    "*",
    {
      root: base.output.path,                      //根目录
      verbose: true,        　　　　　　　　　　//开启在控制台输出信息
      dry: false        　　　　　　　　　　//启用删除文件
    }
  ))
}
```

添加图片、字体文件处理：

```js
const base = {
  module: {
    rules: [{
      test: /\.(woff|woff2|ttf|eot|png|jpg|jpeg|gif|svg)(\?v=\d+\.\d+\.\d+)?$/i, // 图片加载
      loader: 'url-loader',
      query: {
        limit: 10000
      }
    }]
  }
}
```

production 生成环境对编译进行 optimization

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

const base = {
  optimization: {
    minimize: !isDebug, // 是否压缩
    minimizer: !isDebug ? [new UglifyJsPlugin({
      cache: true,  // 使用缓存
      parallel: true,  // 多线程并行处理
      sourceMap: true,  // 使用sourceMap
      uglifyOptions: {
        comments: false,
        warnings: false,
        compress: {
          unused: true,
          dead_code: true,
          collapse_vars: true,
          reduce_vars: true
        },
        output: {
          comments: false
        }
      }
    })] : [],
    splitChunks: {  // 自行切割所有chunk
      chunks: 'all'
    }
  },
}
```

splitChunks 配置的 `chunks: 'all'` 会改变html的引进的脚本，加了chunksHash后每次编译的结果会不一致，需要结合html-webpack-plugin 使用。

下面添加 plugins

```js
const ProgressBarPlugin = require('progress-bar-webpack-plugin');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
const HtmlWebpackPlugin = require('html-webpack-plugin');
const base = {
  plugins: [
    new ProgressBarPlugin(), // 为编译添加进度条
    new webpack.DefinePlugin({  // 为项目注入环境变量
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
      '__DEV__': isDebug
    }),
    new BundleAnalyzerPlugin({  // 生成编译结果分析报告
      analyzerMode: 'server',
      analyzerHost: '127.0.0.1',
      analyzerPort: 8889,
      reportFilename: 'report.html',
      defaultSizes: 'parsed',
      generateStatsFile: false,
      statsFilename: 'stats.json',
      statsOptions: null,
      logLevel: 'info'
    }),
    new HtmlWebpackPlugin({  // 使用html模板，编译结束后会根据 entry 注入 script脚本 和 css样式表
      filename: 'index.html',
      template: path.resolve(__dirname, './index.html')
    })
  ]
}

```

导出配置

```js
module.exports = base;
```

## React 配置 webpack.react.config.js

webpack 的 react 配置，只要是针对 babel-loader 进行配置，首先声明一个 bable-loader：

```js
const path = require('path');

const babelLoader = {
  test: /\.jsx?$/,
  loader: 'babel-loader',
  include: [path.resolve(process.cwd(), 'src')],
  query: {
    babelrc: false,  // 禁止使用.babelrc文件
    presets: [  // 配置 presets
      'react',
      'stage-0',
      [
        'env',
        {
          targets: {
            browsers: ["last 2 versions", "safari >= 7", "ie >= 9", 'chrome >= 52']
          },
          useBuiltIns: true,
          debug: false
        }
      ]
    ],
    plugins: [
      'transform-decorators-legacy',
      'transform-class-properties'
    ]
  }
}
```

首先对preset进行理解，就是bable的一个套餐，里面包含了各种plugin
* 使用 babel-preset-react 让其解析jsx语法
* 使用 babel-preset-stage-0（stage中最高级的套餐），让其对ES6的语法进行解析
* 使用 babel-preset-env，让其针对配置，对其加入不同的 polyfill，这里使用的是 useBuiltIns，针对我们在 base 配置中的 babel-polyfill 进行切割，针对我们在项目中使用到的不兼容的特性进行 polyfill。

另外，添加另外的 plugins
* babel-plugin-transform-decorators-legacy 解析装饰器语法，也就是变量前边的@符号，如antd高阶组件中的@Form.create()
* babel-plugin-transform-class-properties 解析 class 语法

另外，我们针对开发环境，为react组件添加热替换 preset，babel-preset-react-hmre

```js
if (isDebug) {
  babelLoader.query.presets = ['react-hmre'].concat(babelLoader.query.presets)
}
```

另外，为了加快编译速度，我们使用happypack进行多线程编译

```js
const HappyPack = require('happypack');
const os = require('os');
const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length }); // cpus核数
const happyLoaderId = 'happypack-for-react-babel-loader';

const reactConfig = {
  module: {
    rules: [{
      test: babelLoader.test,
      loader: 'happypack/loader',
      query: {
        id: happyLoaderId
      },
      include: babelLoader.include
    }]
  },
  plugins: [new HappyPack({
    id: happyLoaderId,
    threadPool: happyThreadPool,
    loaders: [babelLoader]
  })]
}
delete babelLoader.test;
delete babelLoader.include;

module.exports = reactConfig;
```

## LESS 配置 webpack.less.config.js

首先，配置 css-loader

```js
const isDebug = process.env.NODE_ENV !== 'production';

const cssLoader = {
  loader: `css-loader`,
  options: {
    sourceMap: isDebug, // 是否添加source-map
    modules: true,  // 是否使用css-module
    localIdentName: '[local]', // 使用class本身名字，不添加任何hash
  }
}
```

配置 postcss-loader

```js
const postcssLoader = {
  loader: 'postcss-loader',
  options: {
    config: {
      path: __dirname
    }
  }
}
```

这里我们使用配置文件 postcss.config.js 路径指向当前文件夹，然后新建配置文件 postcss.config.js，如下

```js
module.exports = {
  plugins: () => {
    return [
      require('postcss-nested')(), // 用于解开 @media, @supports, @font-face 和 @document 等css规则
      require('pixrem')(), // 为 rem 单位添加像素转化
      require('autoprefixer')({ // 添加内核前缀
        browsers: ['last 2 versions', 'Firefox ESR', '> 1%', 'ie >= 8']
      }),
      require('postcss-flexibility')(), // 添加 flex 布局 polyfill
      require('postcss-discard-duplicates')() // 去除css中的重复规则
    ]
  }
}
```

配置 less-loader

```js
const lessLoader = {
  loader: 'less-loader',
  options: {
    sourceMap: isDebug,
    javascriptEnabled: true  // 支持内联JavaScript
  }
}
```

接下来，我们针对不同的环境，为webpack添加不同的 module.rules 和 plugins，首先是开发环境，我们使用 style-loader 将css进行内联（个人认为内联css对热部署比较友好），另外，同react配置，为了加快编译，我们使用 happypack 对 loader 进行包裹

```js
const HappyPack = require('happypack');
const os = require('os');
const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });
const lessHappyLoaderId = 'happypack-for-less-loader';
const cssHappyLoaderId = 'happypack-for-css-loader';

let loaders = [];
let plugins = [];

if (isDebug) {
  loaders = [{
    test: /\.less$/,
    loader: 'happypack/loader',
    query: {id: lessHappyLoaderId}
  }, {
    test: /\.css$/,
    loader: 'happypack/loader',
    query: {id: cssHappyLoaderId}
  }]

  plugins = [new HappyPack({
    id: lessHappyLoaderId,
    threadPool: happyThreadPool,
    loaders: ['style-loader', cssLoader, postcssLoader, lessLoader ]
  }),  new HappyPack({
    id: cssHappyLoaderId,
    threadPool: happyThreadPool,
    loaders: ['style-loader', cssLoader, postcssLoader ]
  })]
}
```

然后，对于生产环境，我们使用 mini-css-extract-plugin 将 css 文件分离出来，并打包成 chunks，以便减少线上的首屏加载时间。

```js
if (!isDebug) {
  loaders = [{
    test: /\.less$/,
    use: [MiniCssExtractPlugin.loader, {
      loader: 'happypack/loader',
      query: {id: lessHappyLoaderId}
    }]
  }, {
    test: /\.css/,
    use: [MiniCssExtractPlugin.loader, {
      loader: 'happypack/loader',
      query: {id: cssHappyLoaderId}
    }]
  }]

  plugins = [new MiniCssExtractPlugin({
    filename: '[name].css',
    // chunkFilename: "[id].css"
  }), new HappyPack({
    id: lessHappyLoaderId,
    loaders: [
      cssLoader,
      postcssLoader,
      lessLoader
    ],
    threadPool: happyThreadPool
  }), new HappyPack({
    id: cssHappyLoaderId,
    loaders: [
      cssLoader,
      postcssLoader
    ],
    threadPool: happyThreadPool
  })]
}
```

最后，导出配置

```js
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');

const lessConfig = {
  module: {
    rules: loaders
  },
  plugins,
  optimization: {
    minimizer: [new OptimizeCssAssetsPlugin({ // 使用 OptimizeCssAssetsPlugin 对css进行压缩
      cssProcessor: require('cssnano'),   // css 压缩优化器
      cssProcessorOptions: { discardComments: { removeAll: true } } // 去除所有注释
    })]
  }
};

module.exports = lessConfig;
```

## 合并配置 webpack.config.js

最后，我们将所有配置 merge 在一起

```js
const merge = require('webpack-merge');
const baseConfig = require('./webpack.base.config');
const reactConfig = require('./webpack.react.config');
const lessConfig = require('./webpack.less.config');

const config = merge(baseConfig, reactConfig, lessConfig);

module.exports = config;
```

然后我们配置 package.json 的 sctipts，这里我们使用`better-npm-run`导出环境变量

```json
{
  "scripts": {
    "start": "better-npm-run start",
    "build": "better-npm-run build"
  },
  "betterScripts": {
    "start": {
      "command": "webpack-dev-server --config ./build/webpack.config.js",
      "env": {
        "NODE_ENV": "development"
      }
    },
    "build": {
      "command": "webpack --config ./build/webpack.config.js",
      "env": {
        "NODE_ENV": "production"
      }
    }
  },
}
```

好的，配置到这里已经完成，我们可以肆无忌惮的执行 `npm run start`了。

## 额外配置

针对 React 项目，对于开发过程，我们只关心业务代码的增量编译，对于一些第三方 module 我们不需要对齐进行更改，对于生产环境，这些第三方包也可以利用缓存将其缓存起来，优化线上用户体验，所以我们可以使用`DllPlugin`或者`SplitChunksPlugin`对这些第三方包进行分离。

DllPlugin 可以将指定的module提前编译好，然后在每次解析到这些指定的module时，webpack可直接使用这些module，而不用重新编译，这样可以大大的增加我们的编译速度。

SplitChunksPlugin，可以使用test对module进行正则匹配，对指定的模块打包成chunk，然后每次编译的时候直接使用这些chunk的缓存，而不用每次解析组装这些module。当然，使用SplitChunksPlugin生成的chunk在生成环境可能因为我们指定了`chunkHash`每次文件名不一样，导致我们不能好好利用浏览器缓存这些第三方库，也会因此影响到我们html中每次引入的script，必须结合html-webpack-plugin进行使用，但对于一些没有完全前后端分离的业务项目来说（如路由由后端来控制，html渲染是后端控制），这很明显是一个麻烦。


### dllPlugin

dllPlugin的原理就是预先编译模块，然后在html中最先引进这些打包完的包，这样 webpack 就可以从全局变量里面去找这些预先编译好的模块。

下面我们使用配置使用 dllPlugin，新建配置文件 webpack.dll.config.js，这个文件为 webpack 需要事先编译的配置文件

首先声明输出 output 

```js
const path = require('path');
const isDebug = process.env.NODE_ENV !== 'production';
const output = {
  filename: '[name].js',
  library: '[name]_library',
  path: path.resolve(process.cwd(), isDebug ? './vendor-dev/' : './vendor/') // 编译打包后的目录
}
```

然后声明总体配置

```js
const dllConfig = {
  entry: {
    vendor: ['react', 'react-dom']  // 我们需要事先编译的模块，用entry表示
  },
  output: output,
  plugins: [
    new webpack.DllPlugin({  // 使用dllPlugin
      path: path.join(output.path, `${output.filename}.json`),
      name: output.library // 全局变量名， 也就是 window 下 的 [output.library]
    }),
    new ProgressBarPlugin(),
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
      __DEV__: isDebug
    })
  ],
  optimization: {}
}

```

然后，我们根据不同的环境，添加配置

```js
if (!isDebug) {
  dllConfig.mode = 'production';
  dllConfig.optimization.minimize = true;
  dllConfig.optimization.minimizer = [new UglifyJsPlugin({
    cache: true,
    parallel: true,
    sourceMap: true,
    uglifyOptions: {
      comments: false,
      warnings: false,
      compress: {
        unused: true,
        dead_code: true,
        collapse_vars: true,
        reduce_vars: true
      },
      output: {
        comments: false
      }
    }
  })];
} else {
  dllConfig.mode = 'development';
}

module.exports = dllConfig;
```

需要注意的是，当我们使用dllPlugin对react进行编译时，我们需要使用isDebug对react进行生产环境和开发环境的区分，因为当我们在生成环境使用开发环境的react的时候，react会报错，所以，我们这里需要对不同环境的库进行打包。

编译打包，最后生成了一个 vendor.js 和 vendor.js.json，然后，我们可以在我们编译的配置中使用 dllReferencePlugin 引进这个json

下面我们新建配置文件 webpack.dll.reference.config.js

```js
const path = require('path');
const dllConfig = require('./webpack.dll.config');
const baseConfig = require('./webpack.base.config');
const webpack = require('webpack');
const isDebug = process.env.NODE_ENV !== 'production';
const CopyWebpackPlugin = require('copy-webpack-plugin');

const dllPath = dllConfig.output.path;
const dllEntry = dllConfig.entry;

const plugins = [
  new CopyWebpackPlugin([{ from: path.join(process.cwd(), isDebug ? './vendor-dev/' : './vendor/'), to: baseConfig.output.path, ignore: ['*.json']}]) // 将dll文件拷贝到编译目录
];

Object.keys(dllEntry).forEach((key) => {
  const manifest = path.join(dllPath, `${key}.js.json`);
  plugins.push(new webpack.DllReferencePlugin({
    manifest: require(manifest), // 引进dllPlugin编译的json文件
    name: `${key}_library` // 全局变量名，与dllPlugin声明的一直
  }))
})

module.exports = {
  plugins
}
```

最后，我们把这个配置在 webpack.config.js 里 merge 进来

```js
const merge = require('webpack-merge');
const baseConfig = require('./webpack.base.config');
const reactConfig = require('./webpack.react.config');
const lessConfig = require('./webpack.less.config');
const dllReferenceConfig = require('./webpack.dll.reference.config');

const config = merge(baseConfig, reactConfig, lessConfig, dllReferenceConfig);

module.exports = config;
```

然后在package.json添加预编译脚本

```json
{
  "scripts": {
    "start:dll": "better-npm-run start:dll",
    "build:dll": "better-npm-run build:dll"
  },
  "betterScripts": {
    "start:dll": {
      "command": "webpack --config ./build/webpack.dll.config.js",
      "env": {
        "NODE_ENV": "development"
      }
    },
    "build:dll": {
      "command": "webpack --config ./build/webpack.dll.config.js",
      "env": {
        "NODE_ENV": "production"
      }
    }
  }
}
```

打完收工，最后，在`npm run start`之前，我们得先执行`npm run start:dll`，并在html中引进这个`vendor.js`，不然会报错，找不到library，html如下

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>app</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="/vendor.js"></script> <!-- 根据根目录设置 -->
</head>
<body>
</body>
</html>
```

### SplitChunksPlugin

针对 SplitChunksPlugin，其实就是打包 chunks，如我们把node_modules下的所有模块打到一个chunk中

```js
const splitChunkConfig = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendor: {
          name: 'vendor',
          chunks: 'initial',
          priority: -10,
          reuseExistingChunk: false,
          test: /node_modules\/(.*)\.js/
        }
      }
    }
  }
}
```

使用 test 匹配 node_modules，最后会生成一个 vendor.chunk.js，如果设置有 chunkHash，文件名会带hash，然后在html中引进即可。

## 最后

以上，基本搞了一套 webpack 相对编译较快的配置，嗯呢~，该沉淀一下，献上以上[github地址](https://github.com/yacan8/webpack-config-tool)，以上配置，已整理成cli，项目根目录一键生成，详情见 [README](https://github.com/yacan8/webpack-config-tool)