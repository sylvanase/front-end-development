# webpack
Webpack 是一个模块打包器。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。
## 安装
```bash
# 安装到全局环境
$ npm install webpack -g

# 进入项目目录
# 确定已经有 package.json，没有就通过 npm init 创建
# 安装 webpack 依赖
$ npm install webpack --save-dev

# 查看 webpack 版本信息
$ npm info webpack

# 安装指定版本的 webpack
$ npm install webpack@1.12.x --save-dev

# 安装webpack开发工具
$ npm install webpack-dev-server --save-dev
```

## 基本使用
html中引入编译后的文件，使用命令行打包js文件：
```bash
# webpack 待编译文件 输出文件
$ webpack entry.js bundle.js
```
模板文件 module.js ：
```javascript
module.exports = 'It works from module.js.';
```
入口文件 entry.js 引用模板：
```javascript
document.write(require('./module.js'));
```

## loader
模块和资源的转换器，接受源文件作为参数，返回转换的结果，可以处理CoffeeScript、 JSX、 LESS 或图片等。

一般以 xxx-loader 的方式命名，xxx 代表了这个 loader 要做的转换功能，比如 json-loader。

安装loader：
```bash
npm install css-loader style-loader --save-dev
```

在引用 loader 的时候可以使用全名 json-loader，或者使用短名 json。这个命名规则和搜索优先级顺序在 webpack 的 resolveLoader.moduleTemplates api （没找到在哪儿） 中定义。
```javascript
Default: ["*-webpack-loader", "*-web-loader", "*-loader", "*"]
```

使用loader的方式：
1. 在 require() 引用模块的时候添加。
2. webpack 全局配置中进行绑定，webpack.config.js。
3. 命令行方式调用。

require方式：
```javascript
require("!style-loader!css-loader!./style.css") // 载入 style.css
```
全局配置方式：
webpack默认的配置文件名是webpack.config.js。
先在package.json中添加依赖，安装loader时如果加了--save-dev，依赖会自动写入package.json
```json
{
  "name": "webpack-example",
  "version": "1.0.0",
  "description": "A simple webpack example.",
  "main": "bundle.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "webpack"
  ],
  "author": "zhaoda",
  "license": "MIT",
  "devDependencies": { //webpack依赖
    "css-loader": "^0.21.0",
    "style-loader": "^0.13.0",
    "webpack": "^1.12.2"
  }
}
```
依赖添加好后，创建webpack.config.js。
```javascript
var webpack = require('webpack')

module.exports = {
  entry: './entry.js',
  output: {
    path: __dirname,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {test: /\.css$/, loader: 'style-loader!css-loader'}
    ]
  }
}
```
此时入口文件entry.js中可以将laoder的前缀去掉，命令行直接运行 webpack 就可以了。
```javascript
require('./style.css')
```

命令行方式：
```javascript
require("./style.css") //原js中引用将loader前缀去掉
```
```bash
$ webpack entry.js bundle.js --module-bind 'css=style-loader!css-loader'
```
## 插件
在webpack.config.js中指定插件的配置信息。
以BannerPlugin为例，作用是在输出的文件头部添加注释信息。
```javascript
var webpack = require('webpack')

module.exports = {
  entry: './entry.js',
  output: {
    path: __dirname,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {test: /\.css$/, loader: 'style-loader!css-loader'}
    ]
  },
  plugins: [ //配置插件
    new webpack.BannerPlugin('This file is created by zhaoda')
  ]
}
```

## 开发环境
```bash
# 在编译输出的时候带有进度和颜色（测试只有进度，未见颜色）
$ webpack --progress --colors

#开启监听
$ webpack --progress --colors --watch

# 安装webpack-dev-server 开发服务
$ npm install webpack-dev-server -g

# 启动服务
$ webpack-dev-server --progress --colors
```

## 故障处理
打印错误详情：
```bash
$ webpack --display-error-details
```
Webpack 的配置提供了 resolve 和 resolveLoader 参数来设置模块解析的处理细节，resolve 用来配置应用层的模块（要被打包的模块）解析，resolveLoader 用来配置 loader 模块的解析。

当出现 Node.js 模块依赖查找失败的时候，可以尝试设置 resolve.fallback 和 resolveLoader.fallback 来解决问题。

> webpack中建议使用绝对路径， `path.resolve(__dirname, "app/folder")` 或者 `path.join(__dirname, "app", "folder")`。

```javascript
module.exports = {
  resolve: { fallback: path.join(__dirname, "node_modules") },
  resolveLoader: { fallback: path.join(__dirname, "node_modules") }
};
```



