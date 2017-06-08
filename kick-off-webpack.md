# Webpack 爱与恨

关于标题，为什么是“爱与恨”？

因为在 webpack 刚出来的时候，我并不是坚定的支持者，有很多地方用起来不方便，api 设计不合理。随着 webpack 和 react 生态的越发完善，加上 webpack2.0 的发布，它的功能也越来越强大，让我又重新认识它。

## 内容提要

### webpack 构建方案

- webpack 生态
- 需求是什么
- 对比其他方案

### webpack vs gulp

- webpack
- gulp
- 什么时候用

## webpack 构建方案

### webpack 生态
网上有好多介绍 webpack 的文章，本文只简单介绍两个基本概念。

- Loaders
- Plugins

#### Loaders
webpack 可以使用 loader 来处理文件，允许你打包除 JavaScript 之外的任何静态资源。

##### 文件
- raw-loader 加载文件原始内容
- file-loader 将文件发送到输出文件夹，并返回 URL
- url-loader 像 file loader 一样工作，但如果文件小于限制，可以返回 data URL

##### 编译
- babel-loader 加载 ES2015+ 代码，然后使用 Babel 转译为 ES5
- traceur-loader 加载 ES2015+ 代码，然后使用 Traceur 转译为 ES5
- ts-loader 像 JavaScript 一样加载 TypeScript 2.0+
- coffee-loader 像 JavaScript 一样加载 CoffeeScript

##### 模板
- html-loader 导出 HTML 为字符串，需要引用静态资源
- markdown-loader 将 Markdown 转译为 HTML
- handlebars-loader 将 Handlebars 转译为 HTML

##### 样式
- css-loader 解析 CSS 文件后，使用 import 加载，并且返回 CSS 代码
- less-loader 加载和转译 LESS 文件
- sass-loader 加载和转译 SASS/SCSS 文件
- postcss-loader 使用 PostCSS 加载和转译 CSS/SSS 文件

#### Plugins

- CommonsChunkPlugin
  - 将多个入口起点之间共享的公共模块，生成为一些 chunk，并且分离到单独的 bundle 中，例如，vendor.bundle.js 和 app.bundle.js
- DefinePlugin, EnvironmentPlugin
  - 允许在编译时(compile time)配置的全局常量，用于允许「开发/发布」构建之间的不同行为
- ExtractTextWebpackPlugin
  - 从 bundle 中提取 CSS 文本到独立的文件
- HtmlWebpackPlugin
  - 用于简化 HTML 文件（index.html）的创建，提供访问 bundle 的服务。
- I18nWebpackPlugin
  - 为 bundle 增加国际化支持
- NamedModulesPlugin
  - 保留编译结果的模块名，便于调试

### 需求是什么
技术问题还是要从需求出发，我们团队的实际需求是什么。
- 区分两套环境
- 多页面多入口
- mock 接口数据
- iconfont 字体打包

#### 1. 区分两套环境
- develop
- production

`/build` 文件夹编译结果如下：

```
/build

  - /develop
    - /pageA
      - index.js
      - index.css
      - index.html
    - /pageB
    - common.js
    - common.css

  - /production
    - /1.0.0
      - /pageA
        - index.js
        - index.css
        - index.html
      - /pageB
      - common.js
      - common.css
    - /1.0.1
      - /pageA
        - index.js
        - index.css
        - index.html
      - /pageB
      - common.js
      - common.css
```

其中开发环境的 html 编译结果为:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>首页</title>
        <link href="/common.css" rel="stylesheet">
        <link href="/home/index.css" rel="stylesheet">
    </head>
    <body>
        <div id="root"></div>
        <script src="/common.js" type="text/javascript"></script>
        <script src="/home/index.js" type="text/javascript"></script>
    </body>
</html>
```

其中生产环境的 html 编译结果为:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>首页</title>
        <link href="/0.1.0/common.css" rel="stylesheet">
        <link href="/0.1.0/home/index.css" rel="stylesheet">
    </head>
    <body>
        <div id="root"></div>
        <script src="/0.1.0/common.js" type="text/javascript"></script>
        <script src="/0.1.0/home/index.js" type="text/javascript"></script>
    </body>
</html>
```
生产环境的最终页面上静态资源路径是 

```html
<script src="/{version}/path/file.js"></script>
```
业界还有一种做法是使用 `/path/file.{hash}.js` 形式的路径，都是为了配合 http 缓存策略，做到前端资源的缓存和无缝发布。

- 缓存策略：在 http 响应头中，设置 `Cache-Control`、 `Expires` 和 `Last-Modified` 控制静态资源缓存。用户只有首次访问需要下载全部静态资源，以后的访问都直接使用缓存资源。

  ```
  cache-control:max-age=31536000
  expires:Fri, 06 Apr 2018 08:32:17 GMT
  last-modified:Thu, 06 Apr 2017 06:54:04 GMT
  ```
  对用户和客户端来说，每次发布更新代码，只需要下载新的资源，而无需清除缓存

- 无缝发布：在团队合作开发中，往往会遇到新老版本兼容和发布顺序的问题。例如原来的 `a.js` 添加了新功能变成了 `a'.js`，为了避免前端先发布上线的代码影响线上业务，保证发布上去的代码兼容旧版本，需要在代码中添加如下的兼容语句：

  ```javascript
  if(newVersion) {
    // new feature
  } else {
    // old feature
  }
  ```

  而使用增量发布的方式不需要这样的额外处理，由于每次都是生成新的 url，那么只要后端引用的路径没有变化，就始终引用旧资源，一旦后端发布完成就自动引用新资源。

#### 多页面多入口
虽然 React 天然是开发 SPA 的利器，它的生态中的配套工具也是以解决 SPA 问题为主，例如 React-Redux, React-Saga 等，但是我们的项目中页面非常简单，没有太多的用户交互，没有太多的组件间的消息传递，如果强行引入这些概念，反而把整个项目搞得非常复杂，因此选用多页面多入口的方案。使用前面提到的 `HtmlWebpackPlugin` 插件进行多页面的构建。

```javascript
// 编译 html，多页面多入口
Object.keys(webpackConfig.entry).forEach(name => {
  webpackConfig.plugins.push(new HtmlWebpackPlugin({
    template: `./src/template.ejs`,
    filename: `${name}.html`,
    chunks: ['common', name]
  }));
});
```

#### mock 接口数据
webpack-dev-server 提供了 `proxy` 代理解决方案，但是没有解决 mock 接口数据的问题。我们利用 Koa 开发了一个简版的 mock 服务器，可以加载本地文件中的模拟测试数据。

首先，在 package.json 中添加 mockEnable 字段，当 mockEnable 为 true 时，则开启 mock 服务。

```javascript
// package.json
{
  "mockEnable": true,
  "proxy": {
    "/api/**": {
      "target": "http://example.com"
    }
  },
}
```
其次，在 webpack.config.js 中，将 package.proxy 的 target 指向 mock 服务器

```javascript
// webpack.config.js
const pkg = require('./package.json');
const MockServer = require('./mock/server.js');

// 将 http://example.com/api/path 的接口，转发到 http://localhost:8088/api/path
if(pkg.mockEnable) {
  let port = 8088;
  MockServer.start(port);

  Object.keys(pkg.proxy).forEach(filter => {
    let proxy = pkg.proxy[filter];
    proxy.target = `http://localhost:${port}`; // mock server
  });
}
```
最后，在 mock 服务器中处理请求，mock server 的逻辑很简单，只有不到 50 行代码：

```javascript
// mock/server.js
const fs = require('fs');
const path = require('path');
const color = require('colorful');

const Koa = require('koa');
const router = require('koa-router')();

let app = new Koa();

/* 访问日志，logger */
app.use(async function (ctx, next) {
  console.log(color.green('Mock Server'), (new Date()).toLocaleString(), 'url:', ctx.url);
  await next();
});

/* 路由，router */
router.all('*', async (ctx) => {
  let filepath = path.resolve(__dirname, 'data', `./${ctx.url}.json`);
  let methodFilepath = path.resolve(__dirname, 'data', `./${ctx.url}.${ctx.method}.json`);
  if(fs.existsSync(filepath)) {
    ctx.body = require(filepath);
  } else if(fs.existsSync(methodFilepath)) {
    ctx.body = require(methodFilepath);
  } else {
    ctx.status = 404;
    ctx.body = 'Mock data not found';
  }
}); 
app.use(router.routes()).use(router.allowedMethods());

/* 启动 Mock Server */
exports.start = function(port) {
  app.listen(port, () => {
    console.log('Mock Server started on', color.green(`http://127.0.0.1:${port}/`));
  });
  return app;
}
```

最终的效果是，只要开启了 mockEnable，当用户在页面上发起请求时，自动返回本地文件中的模拟数据。例如请求的是`GET http://localhost/api/path/users`则返回`/mock/data/api/path/users.json`
中的数据。如果同一个 url 既有 GET 请求又有 POST 请求，则可以通过如下方式避免冲突 

```javascript
/mock/data/api/path/users.get.json
/mock/data/api/path/users.post.json
```

#### iconfont 字体打包
我们的项目是基于 Ant Design 做二次开发，由于 ant design 的 iconfont 会依赖 https://at.alicdn.com/ 的字体资源，而公司内网不能访问外部资源，所以需要将字体打包到自己的应用里。利用 less-loader 的 modifyVars 属性，修改 antd 里的 @icon-url 变量。

先设置 `@icon-url` 的变量值。

```javascript
// webpack.config.js
let cssVars = {
  '@icon-url': '"/assets/iconfont/iconfont"',
}
```
然后在 less-loader 的参数中设置 modifyVars。

```javascript
// webpack.config.js
{
  test: /\.less$/,
  use: ExtractTextPlugin.extract([
    'css-loader', 
    { loader:'less-loader', options: { modifyVars:cssVars } }
  ])
}
```

### 对比其他方案
- atool-build
  - 预设了适用于 antd 的 webpack 构建脚本
  - 配置非常灵活，几乎支持所有场景的定制
  - 对插件的修改略麻烦
  - 版本升级兼容性不太好
- roadhog
  - 使用非常简单，不用关心那么多概念
  - 自定义的灵活性受局限


## Webpack vs Gulp

先看看 webpack 和 gulp 各自的官方说明：

- Webpack
  - webpack is a module bundler for modern JavaScript applications.
- Gulp
  - gulp is a toolkit for automating painful or time-consuming tasks in your development workflow, so you can stop messing around and build something.

从上述官方描述可以看出，webpack 的核心概念是 module bundler，而 gulp 的核心概念是 toolkit for tasks。一个侧重模块打包，一个侧重自动化任务处理。

### webpack - 一切皆是模块

![图片描述](https://raw.githubusercontent.com/meikidd/blog/master/images/3.png)


从官方网站上的图片可以看出，一切皆是模块，不管是 js 模块，还是 css、sass 样式，还是模板(hds)、图片(jpg/png)、字体等资源，都以被 webpack 当做互相依赖的模块（modules with dependencies）处理。经过 loader 的解析，最终处理成页面上可用的静态资源（static assets）。由于模块间需要有互相依赖关系，因此需要在 js 里 require 样式和图片等资源。

> 蛋疼的 api，webpack loader 的参数形式简直反人类，这种字符串拼接的方式不直观且不方便扩展
>
>  ```javascript
>  require("file-loader?name=js/[hash].script.[ext]!./javascript.js");
>
>  require("file-loader?name=html-[hash:6].html!./page.html");
>
>  require("file-loader?name=[hash]!./flash.txt");
>
>  require("file-loader?name=[sha512:hash:base64:7].[ext]!./image.png");
>
>  require("file-loader?name=img-[sha512:hash:base64:7].[ext]!./image.jpg");
>
>  require("file-loader?name=picture.png!./myself.png");
>
>  require("file-loader?name=[path][name].[ext]?[hash]!./dir/file.png")
>  ```

以下是一个简单的 webpack 的例子：

```javascript
// webpack.config.js
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

let webpackConfig = {
  entry: {
    index: './src/index.js'
  },
  output: {
    path: path.resolve(__dirname, './build'),
    filename: '[name].js',
    publicPath: './'
  },
  module: {
    rules: [{
      test: /\.jsx?$/,
      exclude: /node_modules/,
      use: {
        loader: 'babel-loader'
      }
    }, {
      test: /\.less$/,
      use: ExtractTextPlugin.extract([
        'css-loader', 
        'less-loader'
      ])
    }, {
      test: /\.(png|jpg|jpeg|gif)$/i,
      use: { 
        loader:'file-loader', 
        options: {
          name: 'static/images/[name].[ext]'
        }
      }
    }, {
      test: /\.(woff2?|ttf|eot|svg)$/,
      use: { 
        loader:'file-loader', 
        options: {
          name: 'static/fonts/[name].[ext]'
        }
      }
    }]
  },
  plugins: [
    new ExtractTextPlugin('[name].css'),
    new UglifyJSPlugin(),
    new CleanWebpackPlugin(['./build']),
    new HtmlWebpackPlugin({
      template: './src/index.html',
      filename: 'index.html',
      chunks: ['index'],
      title: '首页',
      message: 'webpack 测试页面'
    })
  ]
};

module.exports = webpackConfig;

```

### gulp - 一切基于任务

gulp 是作为一个 task runner 存在的，最核心的功能是自动化任务执行，复杂任务组织，基于文件 stream 的构建，加上完善的插件体系，处理各种类型的任务执行流程。用户可以预先定义好一系列的 task，定义好这些 task 分别做些什么，然后定义好执行顺序，最后由 gulp 来执行这些 task。所以 gulp 可以做到几乎所有 node 能做到的事情，不仅仅是用来打包 js。

下面是一个简单的 gulp 的例子：

```javascript
// gulpfile.js
const gulp = require('gulp');
const clean = require('del');
const ejs = require('gulp-ejs');
const less = require('gulp-less');
const jsmin  = require('gulp-jsmin');
const minifyCSS = require('gulp-csso');

gulp.task('html', function(){
  return gulp.src('src/*.html')
    .pipe(ejs({
      title: '首页',
      message: 'gulp 测试页面'
    }))
    .pipe(gulp.dest('build'))
});

gulp.task('css', function(){
  return gulp.src('src/*.less')
    .pipe(less())
    .pipe(minifyCSS())
    .pipe(gulp.dest('build'))
});

gulp.task('js', function () {
    gulp.src(['src/*.js'])
        .pipe(jsmin())
        .pipe(gulp.dest('build'))
});

gulp.task('clean', function () {
    clean(['build']);
});

gulp.task('clone', function () {
    gulp.src(['static/**'])
        .pipe(gulp.dest('build/static/'))
});

gulp.task('default', ['clean', 'clone', 'html', 'css', 'js' ]);

```

### webpack vs gulp 
下面对比 webpack 和 gulp 各自适合的场景

- webpack
  - 基于模块依赖的打包构建
  - 模块切割
  - 公共模块提取

- gulp
  - 与模块化无关的构建过程
    - js / css 批量压缩
    - 图片压缩
    - 批量文本替换
  - 复杂的任务处理
    - 项目发布任务
    - 启动服务器
  
  