---
title: Webpack构建原理
tags:
  - 前端工程化
url: 52.html
id: 52
categories:
  - 前端
date: 2017-08-19 18:09:08
---

![](http://7xqgks.com1.z0.glb.clouddn.com/head-0037.jpg) ![](https://sfault-image.b0.upaiyun.com/307/325/3073259919-576944e693dc3_articlex) 现代前端工程化：ES6 + Webpack + React + Babel，commonjs 的模块必须在使用前经过 webpack 的构建才可以被游览器直接使用，webpack是现代前端重要一环。

# webpack解决的问题

当项目很复杂的时候，没有模块系统会产生很多复杂问题，模块化相关如下：http://bugzhang.com/2017/08/01/javascriptmo-kuai-hua/ webpack可以打包所有静态资源，常见的例如：

*   根据模板生成html,并且自动处理上面的css/JS引用路径
*   自动处理img里图片路径，css样式中背景图的路径，字体引用
*   开启本地服务器，变写代码边自动更新页面内容watch
*   编译jsx、es6、sass、less、coffeescript等等并添加md5、sourcemap等辅助
*   异步加载内容，比如弹出框，不需要时不加载到dom
*   配合vue.js、react等框架，解析相关文件 ![](http://webpack.github.io/assets/what-is-webpack.png)

# webpack整体架构

以webpack.config主要部分划分，webpack主要有以下部分：

*   entry：一个可执行模块的入口文件
*   output：模块的出口目录
*   chunk：多个文件组成的一个代码块
*   loader：模块转换器，确定处理方式
*   plugin：插件，拓展webpack功能，与loader一同定义webpack的处理方式

```js
    module.exports = {
      devtool: isProd
        ? false
        : '#cheap-module-source-map',
      output: {
        path: path.resolve(__dirname, '../dist'),
        publicPath: '/dist/',
        filename: '[name].[chunkhash].js'
      },
      resolve: {
        alias: {
          'public': path.resolve(__dirname, '../public')
        }
      },
      module: {
        noParse: /es6-promise\.js$/, // avoid webpack shimming process
        rules: [
          {
            test: /\.vue$/,
            loader: 'vue-loader',
            options: vueConfig
          },
          {
            test: /\.js$/,
            loader: 'babel-loader',
            exclude: /node_modules/
          },
          {
            test: /\.(png|jpg|gif|svg)$/,
            loader: 'url-loader',
            options: {
              limit: 10000,
              name: '[name].[ext]?[hash]'
            }
          },
          {
            test: /\.css$/,
            use: isProd
              ? ExtractTextPlugin.extract({
                  use: 'css-loader?minimize',
                  fallback: 'vue-style-loader'
                })
              : ['vue-style-loader', 'css-loader']
          }
        ]
      },
      performance: {
        maxEntrypointSize: 300000,
        hints: isProd ? 'warning' : false
      },
      plugins: isProd
        ? [
            new webpack.optimize.UglifyJsPlugin({
              compress: { warnings: false }
            }),
            new ExtractTextPlugin({
              filename: 'common.[chunkhash].css'
            })
          ]
        : [
            new FriendlyErrorsPlugin()
          ]
    }
```

> 一个典型的webpack配置文件

# webpack构建过程

官网对webpack构建的图示如下： ![](https://segmentfault.com/img/remote/1460000004839887) webpack会将所有静态资源看做是模块，然后把这些模块组成到一个bundle，在页面上最终引入一个bundle.js实现对静态资源的加载。大概过程如下： ![](https://segmentfault.com/img/remote/1460000005770047) 构建过程中，webpack做了很多工作，主要的有：

*   读取并初始化option
*   编译
*   递归分析依赖，按照依赖build
*   构建，构建过程中会用相应的loader
*   构建完毕后编译，生成AST抽象语法树
*   遍历AST，在有依赖时，收集依赖
*   打包前合并、压缩等
*   输出文件