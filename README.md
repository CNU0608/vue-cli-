# vue-cli脚手架项目结构分析
>接触Vue的同学应该都知道，用Vue-cli开发Vue项目是非常方便的，它可以帮阻你快速构建一个拥有强大功能能力的VueJS?项目。今天来说说用Vue-cli构建的项目结构是什么样的并且分析文件

这是我最近写的一个高仿饿了么开源项目为参考，项目使用Vue-cli脚手架工具构建的，项目github地址： https://github.com/CNU0608/elemeApp

##项目结构

```

|-- build   		// 构建webpack相关代码
|   |--build.js 		// 生产环境构建代码
|   |--check-versions.js		 // 检查npm、node等版本
|   |--dev-client.js		// 热重载相关
|   |--dev.server.js		// 构建本地服务器
|   |--utils.js			//构建工具相关
|   |--vue-loader.conf.js		// vue文件转换相关
|   |--webpack.base.conf.js			// webpack基础配置
|   |--webpack.dev.conf.js 			// webpack开发环境配置
|   |--webpack.prod.conf.js			 // webpack生产环境配置
|-- config      // 项目开发环境配置
|   |--dev.env.js       // 开发环境变量
|   |--index.js     // 项目一些配置变量
|   |--pro.env.js       //生产环境变量
|-- src         //源码目录
|   |-- components                     // vue公共组件
|   |-- store                          // vuex的状态管理
|   |-- App.vue                        // 页面入口文件
|   |-- main.js                         // 程序入口文件，加载各种公共组件
|-- tatic       // 静态文件，比如一些图片，json数据等
|-- .babelrc         // ES6语法编译配置
|-- .editorconfig       // 定义代码格式
|-- .eslintrc.js        //eslint语法规则
|-- .gitibnore      // git上传需要忽略的文件格式
|-- index.html      // 入口页面
|-- package.json        // 项目基本信息

```

## package.json
package.json文件是项目根目录下的一个文件，定义该项目开发所需要的各种依赖模块以及一些项目配置信息（如项目名称、版本、描述、作者等）。

### scripts字段
在package.json文件里有一个scripts字段。
```
 "scripts": {
    "dev": "node build/dev-server.js",
    "build": "node build/build.js",
    "lint": "eslint --ext .js,.vue src"
  }

```
在开发环境下，在命令行中运行`npm run dev`就相当于在执行`node build/dev-server.js`。所以`script`字段是用来指定`npm`相关命令的缩写的。

### dependencies字段和devDependencies字段
dependencies字段指定了项目运行时所依赖的模块，devDependencies字段指定了项目开发时所依赖的模块。在命令行中运行`npm install`命令，会自动安装dependencies和devDependencies字段中的模块。
package.json还有很多配置相关项，想进一步了解package.json的可参考: https://docs.npmjs.com/files/package.json

## webpack配置相关
上面说到在命令行中`npm run dev`就相当于在执行`node build/dev-server.js`，想必dev-server.js这个文件是十分重要的，它是在开发环境下构建时第一个要运行的文件。在这里我只做简要的补充，详细简介文件，在这特意推荐一篇对Vue-cli@2.0 webpack配置的分析文章（一定要自习阅读，收获会很大）： https://gold.xitu.io/post/584e48b2ac502e006c74a120

### dev-server.js

```javascript
...
...
// http-proxy可以实现转发所有请求代理到后端真实API地址，以实现前后端开发完全分离
// 在config/index.js文件中可以对proxyTable想进行配置
var proxyMiddleware = require('http-proxy-middleware')
...
...

```

### webpack.base.conf.js

```javascript

...
...
module.exports = {
  entry: { // 编译入口文件
    app: './src/main.js'
  },
  output: { // 编译输出文件
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: { // 一些解决方案配置
    extensions: ['.js', '.vue', '.json'],
    modules: [
      resolve('src'),
      resolve('node_modules')
    ],
    alias: {
      'vue$': 'vue/dist/vue.common.js',
      'src': resolve('src'),
      'components': resolve('src/components'),
      'common': resolve('src/common')
    }
  },
  module: {
    rules: [
      {
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      }
      // ...
    ]
  }
}
...
...

```

## .babelrc
Babel解释器的配置文件，存放在根目录下。Babel是一个转码器，项目里需要用它将ES6代码转为ES5代码。这里有一篇阮一峰老师写的相关文章供参考：[Babel入门教程](http://www.ruanyifeng.com/blog/2016/01/babel.html)

```

{
  // 设定转码规则
  "presets": [
    ["es2015", { "modules": false }],
    "stage-2"
  ],
  // 转码的一些插件
  "plugins": ["transform-runtime"],
  "comments": false,
  "env": {
    "test": {
      "plugins": [ "istanbul" ]
    }
  }
}

```

## .editorconfig
该文件定义项目的编码规范，编辑器的行为会与.editorconfig 文件中定义的一致，并且其优先级比编辑器自身的设置要高，这在多人合作开发项目时十分有用而且必要。

```
root = true

[*]	// 对所有文件应用下面的规则
charset = utf-8	// 编码规则用utf-8
indent_style = space // 缩进用空格
indent_size = 2	// 缩进数量为2个空格
end_of_line = lf // 换行符格式
insert_final_newline = true // 是否在文件的最后插入一个空行
trim_trailing_whitespace = true // 是否删除行尾的空格

```
了解更多请参考官方配置文档 http://editorconfig.org/

接触vue并不久，很多东西也不是特别清楚，文章里有什么问题欢迎指出。(通过这篇文章希望能帮助到更多刚入门Vue-cli脚手架的前端开发者。好啦，今天分享的内容就这些，happy，happy！！)
