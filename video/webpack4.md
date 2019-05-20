# webpack 4.x 常用配置汇总

逛 B 站无意间找到了一个讲解 webpack 4 的视频 https://www.bilibili.com/video/av41371417/?p=1 ，发现讲的挺全面的，于是学习一下并做此记录。webpack 发展到现在，入门学习的成本还是很高的，借助视频学习是一个很不错的方式。

PS：这个视频是某峰培训的视频，虽然网上很多人对培训班有各种各样的言论，但就这个视频而言讲的很好。

------

## webpack 是什么

webpack 的核心价值就是前端源码的打包，即将前端源码中每一个文件（无论任何类型）都当做一个 pack ，然后分析依赖，将其最终打包出线上运行的代码。webpack 的四个核心部分

- entry 规定入口文件，一个或者多个
- output 规定输出文件的位置
- loader 各个类型的转换工具
- plugin 打包过程中各种自定义功能的插件

webpack 如今已经进入 v4.x 版本，v5.x 估计也会很快发布。不过看 v5 的变化相比于 v4 ，常用的配置没有变，这是一个好消息，说明基本稳定。

------

## 前端工程师需要了解的 webpack

前端工程化是近几年前端发展迅速的主要推手之一，webpack 无疑是前端工程化的核心工具。目前前端工程化工具还没有到一键生成，或者重度继承到某个 IDE 中（虽然有些 cli 工具可以直接创建），还是需要开发人员手动做一些配置。

因此，作为前端开发人员，熟练应用 webpack 的常用配置、常用优化方案是必备的技能 —— 这也正是本文的内容。另外，webpack 的实现原理算是一个加分项，不要求所有开发人员掌握，本文也没有涉及。

------

## 基础配置

### 初始化环境

`npm init -y` 初始化 npm 环境，然后安装 webpack `npm i webpack webpack-cli -D`

新建 `src` 目录并在其中新建 `index.js` ，随便写点 `console.log('index js')` 。然后根目录创建 `webpack.config.js` ，内容如下

```js
const path = require('path')

module.exports = {
    // mode 可选 development 或 production ，默认为后者
    // production 会默认压缩代码并进行其他优化（如 tree shaking）
    mode: 'development',
    entry: path.join(__dirname, 'src', 'index'),
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    }
}
```

然后增加 `package.json` 的 `scripts`

```json
  "scripts": {
    "build": "webpack"
  },
```

然后运行 `npm run build` 即可打包文件到 `dist` 目录。

### 区分 dev 和 build

使用 webpack 需要两个最基本的功能：第一，开发的代码运行一下看看是否有效；第二，开发完毕了将代码打包出来。这两个操作的需求、配置都是完全不一样的。例如，运行代码时不需要压缩以便 debug ，而打包代码时就需要压缩以减少文件体积。因此，这里我们还是先把两者分开，方便接下来各个步骤的讲解。

首先，安装 `npm i webpack-merge -D` ，然后根目录新建 `build` 目录，其中新建如下三个文件。

```js
// webpack.common.js 公共的配置
const path = require('path')
const srcPath = path.join(__dirname, '..', 'src')
const distPath = path.join(__dirname, '..', 'dist')
module.exports = {
    entry: path.join(srcPath, 'index')
}
```

```js
// webpack.dev.js 运行代码的配置（该文件暂时用不到，先创建了，下文会用到）
const path = require('path')
const webpackCommonConf = require('./webpack.common.js')
const { smart } = require('webpack-merge')
const srcPath = path.join(__dirname, '..', 'src')
const distPath = path.join(__dirname, '..', 'dist')
module.exports = smart(webpackCommonConf, {
    mode: 'development'
})
```

```js
// webpack.prod.js 打包代码的配置
const path = require('path')
const webpackCommonConf = require('./webpack.common.js')
const { smart } = require('webpack-merge')
const srcPath = path.join(__dirname, '..', 'src')
const distPath = path.join(__dirname, '..', 'dist')
module.exports = smart(webpackCommonConf, {
    mode: 'production',
    output: {
        filename: 'bundle.[contentHash:8].js',  // 打包代码时，加上 hash 戳
        path: distPath,
        // publicPath: 'http://cdn.abc.com'  // 修改所有静态文件 url 的前缀（如 cdn 域名），这里暂时用不到
    }
})
```

修改 `package.json` 中的 `scripts`

```json
  "scripts": {
    "build": "webpack --config build/webpack.prod.js"
  },
```

重新运行 `npm run build` 即可看到打包出来的代码。最后，别忘了将根目录下的 `webpack.config.js` 删除。

这将引发一个新的问题：js 代码中将如何判断是什么环境呢？需要借助 `webpack.DefinedPlugin` 插件来定义全局变量。可以在 `webpack.dev.js` 和 `webpack.prod.js` 中做如下配置：

```js
// 引入 webpack
const webpack = require('webpack')

// 增加 webpack 配置
    plugins: [
        new webpack.DefinePlugin({
            // 注意：此处 webpack.dev.js 中写 'development' ，webpack.prod.js 中写 'production'
            ENV: JSON.stringify('development')
        })
```

最后，修改 `src/index.js` 只需加入一行 `console.log(ENV)` ，然后重启 `npm run dev` 即可看到效果。

### JS 模块化

webpack 默认支持 js 各种模块化，如常见的 commonJS 和 ES6 Module 。但是推荐使用 ES6 Module ，**因为 production 模式下，ES6 Module 会默认触发 tree shaking** ，而 commonJS 则没有这个福利。究其原因，ES6 Module 是静态引用，在编译时即可确定依赖关系，而 commonJS 是动态引用。

不过使用 ES6 Module 时，ES6 的解构赋值语法这里有一个坑，例如 `index.js` 中有一行 `import {fn, name} from './a.js'` ，此时 `a.js` 中有以下几种写法，大家要注意！

```js
// 正确写法一
export function fn() {
    console.log('fn')
}
export const name = 'b'
```

```js
// 正确写法二
function fn() {
    console.log('fn')
}
const name = 'b'
export {
    fn,
    name
}
```

```js
// 错误写法
function fn() {
    console.log('fn')
}
export default {
    fn,
    name: 'b'
}
```

该现象的具体原因可参考 https://www.jianshu.com/p/ba6f582d5249 。下文马上要讲解启动本地服务，读者可以马上写一个 demo 自己验证一下这个现象。

------

## 启动本地服务

上文创建的 `webpack.dev.js` 一直没使用，下面就要用起来。

### 使用 html

启动本地服务，肯定需要一个 html 页面作为载体，新建一个 `src/index.html` 并初始化内容

```js
<!DOCTYPE html>
<html>
<head><title>Document</title></head>
<body>
    <p>this is index html</p>
</body>
</html>
```

要使用这个 html 文件，还需要安装 `npm i html-webpack-plugin -D` ，然后配置 `build/webpack.common.js` ，因为无论 dev 还是 prod 都需要打包 html 文件。

```js
    plugins: [
        new HtmlWebpackPlugin({
            template: path.join(srcPath, 'index.html'),
            filename: 'index.html'
        })
    ]
```

重新运行 `npm run build` 会发现打包出来了 `dist/index.html` ，且内部已经自动插入了打包的 js 文件。

### webpack-dev-server

有了 html 和 js 文件，就可以启动服务了。首先安装 `npm i webpack-dev-server -D` ，然后打开 `build/webpack.dev.js`配置。只有运行代码才需要本地 server ，打包代码时不需要。

```js
devServer: {
    port: 3000,
    progress: true,  // 显示打包的进度条
    contentBase: distPath,  // 根目录
    open: true,  // 自动打开浏览器
    compress: true  // 启动 gzip 压缩
}
```

打开 `package.json` 修改 `scripts` ，增加 `"dev": "webpack-dev-server --config build/webpack.dev.js",` 。然后运行 `npm run dev` ，打开浏览器访问 `localhost:3000` 即可看到效果。

### 解决跨域

实际开发中，server 端提供的端口地址和前端可能不同，导致 ajax 收到跨域限制。使用 webpack-dev-server 可配置代理，解决跨域问题。如有需要，在 `build/webpack.dev.js` 中增加如下配置。

```js
    devServer: {
        // 设置代理
        proxy: {
            // 将本地 /api/xxx 代理到 localhost:3000/api/xxx
            '/api': 'http://localhost:3000',

            // 将本地 /api2/xxx 代理到 localhost:3000/xxx
            '/api2': {
                target: 'http://localhost:3000',
                pathRewrite: {
                    '/api2': ''
                }
            }
        }
```

------

## 处理 ES6

### 使用 babel

由于现在浏览器还不能保证完全支持 ES6 ，将 ES6 编译为 ES5 ，需要借助 babel 这个神器。安装 babel `npm i babel-loader @babel/core @babel/preset-env -D` ，然后修改 `build/webpack.common.js` 配置

```js
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: ['babel-loader'],
                include: srcPath,
                exclude: /node_modules/
            },
        ]
    },
```

还要根目录下新建一个 `.babelrc` json 文件，内容下

```json
{
    "presets": ["@babel/preset-env"],
    "plugins": []
}
```

在 `src/index.js` 中加入一行 ES6 代码，如箭头函数 `const fn = () => { console.log('this is fn') }` 。然后重新运行 `npm run dev`，可以看到浏览器中加载的 js 中，这个函数已经被编译为 `function` 形式。

### 使用高级特性

babel 可以解析 ES6 大部分语法特性，但是无法解析 class 、静态属性、块级作用域，还有很多大于 ES6 版本的语法特性，如装饰器。因此，想要把日常开发中的 ES6 代码全部转换为 ES5 ，还需要借助很多 babel 插件。

安装 `npm i @babel/plugin-proposal-class-properties @babel/plugin-transform-block-scoping @babel/plugin-transform-classes -D` ，然后配置 `.babelrc`

```json
{
    "presets": ["@babel/preset-env"],
    "plugins": [
        "@babel/plugin-proposal-class-properties",
        "@babel/plugin-transform-block-scoping",
        "@babel/plugin-transform-classes"
    ]
}
```

在 `src/index.js` 中新增一段 `class` 代码，然后重新运行 `npm run build` ，打包出来的代码会将 `class` 转换为 `function` 形式。

------

## source map

source map 用于反解析压缩代码中错误的行列信息，dev 时代码没有压缩，用不到 source map ，因此要配置 `build/webpack.prod.js`

```js
// webpack 中 source map 的可选项，是情况选择一种：

// devtool: 'source-map'  // 1. 生成独立的 source map 文件
// devtool: 'eval-source-map'  // 2. 同 1 ，但不会产生独立的文件，集成到打包出来的 js 文件中
// devtool: 'cheap-module-source-map'  // 3. 生成单独的 source map 文件，但没有列信息（因此文件体积较小）
devtool: 'cheap-module-eval-source-map'  // 4. 同 3 ，但不会产生独立的文件，集成到打包出来的 js 文件中
```

生产环境下推荐使用 1 或者 3 ，即生成独立的 map 文件。修改之后，重新运行 `npm run build` ，会看到打包出来了 map 文件。

------

## 处理样式

在 webpack 看来，不仅仅是 js ，其他的文件也是一个一个的模块，通过相应的 loader 进行解析并最终产出。

### 处理 css

安装必要插件 `npm i style-loader css-loader -D` ，然后配置 `build/webpack.common.js`

```js
    module: {
        rules: [
            { /* js loader */ },
            {
                test: /\.css$/,
                loader: ['style-loader', 'css-loader']  // loader 的执行顺序是：从后往前
            }
        ]
    },
```

新建一个 css 文件，然后引入到 `src/index.js` 中 `import './css/index.css'` ，重新运行 `npm run dev` 即可看到效果。

### 处理 less

less sass 都是常用 css 预处理语言，以 less 为例讲解。安装必要插件 `npm i less less-loader -D` ，然后配置 `build/webpack.common.js`

```js
            {
                test: /\.less$/,
                loader: ['style-loader', 'css-loader', 'less-loader']  // 增加 'less-loader' ，注意顺序
            }
```

新建一个 less 文件，然后引入到 `src/index.js` 中 `import './css/index.less'` ，重新运行 `npm run dev` 即可看到效果。

### 自动添加前缀

一些 css3 的语法，例如 `transform: rotate(45deg);` 为了浏览器兼容性需要加一些前缀，如 `webkit-` ，可以通过 webpack 来自动添加。安装 `npm i postcss-loader autoprefixer -D` ，然后配置

```js
            {
                test: /\.css$/,
                loader: ['style-loader', 'css-loader', 'postcss-loader']  // 增加 'postcss-loader' ， 注意顺序
            }
```

还要新建一个 `postcss.config.js` 文件，内容是

```js
module.exports = {
    plugins: [require('autoprefixer')]
}
```

重新运行 `npm run dev` 即可看到效果，自动增加了必要的前缀。

### 抽离 css 文件

默认情况下，webpack 会将 css 代码全部写入到 html 的 `<style>` 标签中，但是打包代码时需要抽离到单独的 css 文件中。安装 `npm i mini-css-extract-plugin -D` 然后配置 `build/webpack.prod.js`（打包代码时才需要，运行时不需要）

```js
// 引入插件
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

// 增加 webpack 配置
    module: {
        rules: [
            {
                test: /\.css$/,
                loader: [
                    MiniCssExtractPlugin.loader,  // 注意，这里不再用 style-loader
                    'css-loader',
                    'postcss-loader'
                ]
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/main.[contentHash:8].css'
        })
    ]
```

如需要压缩 css ，需要安装 `npm i terser-webpack-plugin optimize-css-assets-webpack-plugin -D` ，然后增加配置

```js
// 引入插件
const TerserJSPlugin = require('terser-webpack-plugin')
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin')

// 增加 webpack 配置
    optimization: {
        minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
    },
```

运行 `npm run build` 即可看到打包出来的 css 是独立的文件，并且是被压缩过的。

------

## 处理图片

要在 js 中 `import` 图片，或者在 css 中设置背景图片。安装 `npm i file-loader -D` 然后配置 `build/webpack.common.js`

```js
            {
                test: /\.(png|jpg|gif)$/,
                use: 'file-loader'
            }
```

如果想要处理 html 代码中 `<img src="..."/>` 的形式，则安装 `npm i html-withimg-loader -D` 然后配置 `build/webpack.common.js`

```js
            {
                test: /\.html$/,
                use: 'html-withimg-loader'
            }
```

打包之后，dist 目录下会生成一个类似 `917bb63ba2e14fc4aa4170a8a702d9f8.jpg` 的文件，并被引入到打包出来的结果中。

如果想要将小图片用 base64 格式产出，则安装 `npm i url-loader -D` ，然后配置 `build/webpack.common.js`

```js
            {
                test: /\.(png|jpg|gif)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        // 小于 5kb 的图片用 base64 格式产出
                        // 否则，依然延用 file-loader 的形式，产出 url 格式
                        limit: 5 * 1024,

                        // 打包到 img 目录下
                        outputPath: '/img/',

                        // 设置图片的 cdn 地址（也可以统一在外面的 output 中设置，那将作用于所有静态资源）
                        // publicPath: 'http://cdn.abc.com'
                    }
                }
            },
```

------

## 多页应用

src 下有 `index.js` `index.html` 和 `other.js` `other.html` ，要打包输出两个页面，且分别引用各自的 js 文件。

第一，配置输入输出

```js
    entry: {
        index: path.join(srcPath, 'index.js'),
        other: path.join(srcPath, 'other.js')
    },
    output: {
        filename: '[name].[contentHash:8].js',  // [name] 表示 chunk 的名称，即上面的 index 和 other
        path: distPath
    },
```

第二，配置 html 插件

```js
    plugins: [
        // 生成 index.html
        new HtmlWebpackPlugin({
            template: path.join(srcPath, 'index.html'),
            filename: 'index.html',
            // chunks 表示该页面要引用哪些 chunk （即上面的 index 和 other），默认全部引用
            chunks: ['index']  // 只引用 index.js
        }),
        // 生成 other.html
        new HtmlWebpackPlugin({
            template: path.join(srcPath, 'other.html'),
            filename: 'other.html',
            chunks: ['other']  // 只引用 other.js
        }),
```

------

## 抽离公共代码

### 公共模块

多个页面或者入口，如果引用了同一段代码，如上文的多页面例子中，`index.js` 和 `other.js` 都引用了 `import './common.js'` ，则 `common.js` 应该被作为公共模块打包。webpack v4 开始弃用了 commonChunkPlugin 改用 splitChunks ，可修改 `build/webpack.prod.js` 中的配置

```js
    optimization: {
        // 分割代码块
        splitChunks: {
            // 缓存分组
            cacheGroups: {
                // 公共的模块
                common: {
                    chunks: 'initial',
                    minSize: 0,  // 公共模块的大小限制
                    minChunks: 2  // 公共模块最少复用过几次
                }
            }
        }
    },
```

重新运行 `npm run build` ，即可看到有 common 模块被单独打包出来，就是 `common.js` 的内容。

### 第三方模块

同理，如果我们的代码中引用了 jquery lodash 等，也希望将第三方模块单独打包，和自己开发的业务代码分开。这样每次重新上线时，第三方模块的代码就可以借助浏览器缓存，提高用户访问网页的效率。修改配置文件，增加下面的 `vendor: {...}` 配置。

```js
    optimization: {
        // 分割代码块
        splitChunks: {
            // 缓存分组
            cacheGroups: {
                // 第三方模块
                vendor: {
                    priority: 1, // 权限更高，优先抽离，重要！！！
                    test: /node_modules/,
                    chunks: 'initial',
                    minSize: 0,  // 大小限制
                    minChunks: 1  // 最少复用过几次
                },

                // 公共的模块
                common: {
                    chunks: 'initial',
                    minSize: 0,  // 公共模块的大小限制
                    minChunks: 2  // 公共模块最少复用过几次
                }
            }
        }
    },
```

重启 `npm run build` ，即可看到 vendor 模块被打包出来，里面是 jquery 或者 lodash 等第三方模块的内容。

------

## 懒加载

webpack 支持使用 `import(...)` 语法进行资源懒加载。安装 `npm i @babel/plugin-syntax-dynamic-import -D` 然后将插件配置到 `.babelrc` 中。

新建 `src/dynamic-data.js` 用于测试，内容是 `export default { message: 'this is dynamic' }` 。然后在 `src/index.js` 中加入

```js
setTimeout(() => {
    import('./dynamic-data.js').then(res => {
        console.log(res.default.message)  // 注意这里的 default
    })
}, 1500)
```

重新运行 `npm run dev` 刷新页面，可以看到 1.5s 之后打印出 `this is dynamic` 。而且，`dynamic-data.js` 也是 1.5s 之后被加载进浏览器的 —— 懒加载，虽然文件名变了。

重新运行 `npm run build` 也可以看到 `dynamic-data.js` 的内容被打包一个单独的文件中。

------

## 常见性能优化

### tree shaking

使用 `import` 引入，在 `production` 环境下，webpack 会自动触发 tree shaking ，去掉无用代码。**但是使用 require 引入时，则不会触发 tree shaking**。这是因为 require 是动态引入，无法在编译时判断哪些功能被使用。而 import 是静态引入，编译时即可判断依赖关系。

### noParse

不去解析某些 lib 其内部的依赖，即确定这些 lib 没有其他依赖，提高解析速度。可配置到 `build/wepback.common.js` 中

```js
    module: {
        noParse: /jquery|lodash/,  // 不解析 jquery 和 lodash 的内部依赖
```

### ignorePlugin

以常用的 `moment` 为例。安装 `npm i moment -d` 并且 `import moment from 'moment'` 之后，monent 默认将所有语言的 js 都加载进来，使得打包文件过大。可以通过 ignorePlugin 插件忽略 locale 下的语言文件，不打包进来。

```js
    plugins: [
        new webpack.IgnorePlugin(/\.\/locale/, /moment/),  // 忽略 moment 下的 /locale 目录
```

这样，使用时可以手动引入中文包，并设置语言

```js
import moment from 'moment'
import 'moment/locale/zh-cn' // 手动引入中文语言包
moment.locale('zh-cn')
const r = moment().endOf('day').fromNow()
console.log(r)
```

### happyPack

多进程打包，参考 https://www.npmjs.com/package/happypack 。注意，小项目使用反而会变慢。只有项目较大，打包出现明显瓶颈时，才考虑使用 happypack 。

------

## 常用插件和配置

### ProvidePlugin

如要给所有的 js 模块直接使用 `$` ，不用每次都 `import $ from 'jquery'` ，可做如下配置

```js
    plugins: [
        new webpack.ProvidePlugin({
            $: 'jquery'
        }),
```

### externals

如果 jquery 已经在 html 中通过 cdn 引用了，无需再打包，可做如下配置

```js
    externals: {
        jquery: 'jQuery'
    },
```

### alias

设置 alias 别名在实际开发中比较常用，尤其是项目较大，目录较多时。可做如下配置

```js
    resolve: {
        alias: {
            Utilities: path.join(srcPath, 'utilities')
        }
    },
```

在该配置之前，可能需要 `import Utility from '../../utilities/utility'` 使用。配置之后就可以 `import Utility from 'Utilities/utility'` 使用，一来书写简洁，二来不用再考虑相对目录的层级关系。

### extensions

如果引用文件时没有写后缀名，可以通过 extensions 来匹配。

```js
    resolve: {
        extensions: [".js", ".json"]
    },
```

### clean-webpack-plugin

由于使用了 contentHash ，每次 build 时候都可能打包出不同的文件，因此要及时清理 dist 目录。安装 `npm i clean-webpack-plugin -D` ，然后在 `build/webpack.prod.js` 中配置

```js
// 引入插件
const CleanWebpackPlugin = require('clean-webpack-plugin')

// 增加配置
    plugins: [
        new CleanWebpackPlugin(),  // 默认清空 output.path 目录
```

### copy-webpack-plugin

build 时，将 src 目录下某个文件或者文件夹，无条件的拷贝到 dist 目录下，例如 `src/doc` 目录拷贝过去。安装 `npm i copy-webpack-plugin -D`，然后在 `build/webpack.prod.js` 中配置

```js
// 引入插件
const CopyWebpackPlugin = require('copy-webpack-plugin')

// 增加配置
    plugins: [
        new CopyWebpackPlugin([
            {
                from: path.join(srcPath, 'doc'),  // 将 src/doc 拷贝到 dist/doc
                to: path.join(distPath, 'doc')
            }
        ]),
```

### bannerPlugin

代码的版权声明，在 `build/webpack.prod.js` 中配置即可。

```js
    plugins: [
        new webpack.BannerPlugin('by github.com/wangfupeng1988 \r'),
```

------

## 总结

webpack 发展至今配置非常多，该视频中也没有全部讲解出来，只是一些实际开发中常用的。其他的配置可以去看官网文档。
