---
title: React+Redux+webpack+Express应用环境搭建实战
date: 2017-02-16 10:19:22
tags:
- Webpack
- React
- Redux
- Express
- 前端
- 未完待续
---

> 文章来源:
> [es6环境搭建](http://es6.ruanyifeng.com/#docs/intro)
> [Webpack傻瓜指南（三）和React配合开发](https://zhuanlan.zhihu.com/p/20522487?columnSlug=FrontendMagazine)

<!-- more -->

## 本文目的

使用webpack对项目工程进行打包监听，实现文件模块化打包、项目文件变动自动更新页面，生成本地服务器进行页面显示和调试。

进阶部分补充如何将Webpack自动更新页面和后台数据处理Express结合进行全栈开发。

---

## 配置流程

### 生成项目目录

项目目录结构如下：

```
┍client(项目源文件目录)
┃ ┝actions（action文件目录）
┃ ┝components（UI组件文件目录）
┃ ┝containers（容器组件文件目录）
┃ ┝utils（公用资源模块目录）
┃ ┝reducers（reducer文件目录）
┃ ┕index.jsx（项目入口）
┝build（编译文件存放目录，可不创建）
┝node_modules（模块仓库）
┝.babelrc（babel配置文件）
┝package.json（项目描述文件）
┕webpack.config.js（webpack配置文件）
```

创建文件夹，命令行输入指令 `npm init` 生成项目描述文件，一路回车，然后便生成了 `package.json` 文件。随后建立 app 目录及 components 目录、utils 目录。

### 安装webpack及配置

输入指令 `npm install --save-dev webpack html-webpack-plugin` 安装 webpack 及插件，然后在根目录新建 `webpack.config.js` 文件，输入以下内容保存。

输入指令 `webpack-dev-server -v`，如无显示对应版本信息的话需要全局安装 webpack 的开发工具，指令为 `npm install -g webpack-dev-server`。

```javascript
// webpack.config.js
var path = require('path');
var webpack = require('webpack');
var HtmlwebpackPlugin = require('html-webpack-plugin');
var ROOT_PATH = path.resolve(__dirname);
var APP_PATH = path.resolve(ROOT_PATH, 'client');
var BUILD_PATH = path.resolve(ROOT_PATH, 'build');
module.exports= {
  entry: {
    client: path.resolve(APP_PATH, 'index.jsx')
  },
  output: {
    path: BUILD_PATH,
    filename: 'bundle.js'
  },
  //enable dev source map
  devtool: 'eval-source-map',
  //enable dev server
  devServer: {
    contentBase: APP_PATH,
    compress: true,
    port: 9000,
    hot: true,
    watchContentBase: true
  },
  //babel重要的loader在这里
  module: {},
  resolve: {
    extensions: ['.js', '.jsx', ".scss", ".less"],
    alias: {}
  },
  plugins: [
    new HtmlwebpackPlugin({
      title: 'My first react app'
    })
  ]
}
```

以上配置将 app 目录下的 `index.jsx` 设置为入口文件文件，设置根目录下的 build 目录设置为打包文件输出目录（使用 `build` 命令时才会生成文件，平时预览的时候编译的 `bundle.js` 文件是在内存中的）。然后还定义了 dev 工具运行的目录为 app 目录（默认入口文件为 `index.jsx`）当运行 `webpack-dev-server` 命令的时候会启动本地服务器的 9000 接口进行页面输出。

`resolve` 选项的 `extensions` 配置为常用文件后缀，进行设置之后引入这些类型的文件可以不输入后缀名。`alias` 配置项为对常用路径设置别名，这样有助于减少系统检索的时间。如：

```javascript
// 设置前
import bootstraps from '../node_modules/bootstrap/less/bootstrap.less';
```

```javascript
// 配置项为
alias: {
  "Bootstrap": path.resolve(__dirname, 'node_modules/bootstrap/less/bootstrap.less')
}
// 设置后可以这样子引入
import bootstrap from 'Bootstrap';
```

### 安装React+Redux并配置

输入命令 `npm install --save-dev react react-dom react-redux redux`，安装 react 和 redux。

由于 react 和 redux 会用到 es6 的语法，所以需要安装 babel 相关插件负责在打包的时候将其转换为 es5 的语法。

输入命令 `npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-stage-2 babel-preset-react` 安装相关插件。

然后在项目根目录建立 `.babelrc` 作为 babel 的配置文件，window 用户可在命令行中输入命令 `cd .>.babelrc` 创建这样一个不带文件名的文件。在 `.babelrc` 文件中输入以下内容。


```javascript{
  "presets": [
    "es2015",
    "react",
    "stage-2"
  ],
  "plugins": []
}
```

之后还需在 `webpack.config.js` 文件中修改 module 配置项，添加 `loaders` 属性，使其支持渲染 .jsx 为后缀的使用 es6 语法的文件。修改结果如下：

```javascript
...
//babel重要的loader在这里
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      loader: 'babel-loader',
      include: APP_PATH      
    }
  ]
},
...
```

其目的是使用 babel-loader 插件对后缀为 jsx 的文件进行语法转换和渲染。

### webpack对其他格式的文件的渲染配置

项目中常会用到的文件类型还有 .less|.sass 的样式脚本文件、.eot|.ttf 之类的 iconfont 文件和图片文件。下面对 webpack 进行配置，使之支持。

输入命令 `npm install --save-dev file-loader url-loader css-loader style-loader` 安装对应插件。

由于我使用 less 对我的 css 文件进行管理，所以我额外安装了 `less less-loader node-less` 选用 sass 的可以安装 `sass sass-loader node-sass` 插件。

然后修改webpack.config.js中的module配置项下面的loaders属性，修改结果如下：

```javascript
...
//babel重要的loader在这里
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      loader: 'babel-loader',
      include: APP_PATH      
    },
    {
      test: /\.scss?$/,
      loaders: ['style-loader', 'css-loader', 'sass-loader']
    },
    {
      test: /\.less?$/,
      loaders: ['style-loader', 'css-loader', 'less-loader']
    },
    {
      test: /\.(gif|jpg|png|woff|svg|eot|ttf)\??.*$/, 
      loader: 'url-loader?limit=50000&name=[path][name].[ext]'
    }
  ]
},
...
```

---

## 测试

在 app 目录下的 `index.jsx` 文件中添加如下代码：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import {createStore} from 'redux';
let normalVal = 0
let counter = (state = normalVal, action) => {
  switch (action.type){
    case 'ADD':
      return state + 1;
    case 'DOWN':
      return state - 1;
    default:
      return state;
  }
}
let store = createStore(counter);
class App extends React.Component{
    constructor() {
        super();
    }
    render() {
        //JSX here!
        return (
          <div className="container">
            <section className="jumbotron">
              <h3 className="jumbotron-heading">React+Redux+webpack</h3>
              <div>{store.getState()}</div>
              <button onClick={() => store.dispatch({type:'ADD'})}>+</button>
              <button onClick={() => store.dispatch({type:'DOWN'})}>-</button>
            </section>
          </div>
        )
    }
};
const app = document.createElement('div');
document.body.appendChild(app);
ReactDOM.render(<App />, app);
store.subscribe(() => ReactDOM.render(<App />, app))
```

可以选用为 react 定制的 bootstrap 模块，`npm install --save react-bootstrap` 使用方式参考这两篇文章 [React-BootStrap 框架快速体验上手](http://blog.csdn.net/github_26672553/article/details/52141457)，[官方文档](https://react-bootstrap.github.io/components.html)。

修改 `package.json` 中的 `script` 选项，将其修改为：

```javascript
...
"scripts": {
  "dev": "webpack-dev-server --progress --profile --colors --hot",
  "build": "webpack --progress --profile --colors",
  "test": "karma start"
},
...
```

然后子啊命令行中输入命令 `npm run dev`，编译成功无报错的话打开浏览器，输入地址 `http://127.0.0.1:9000` 查看效果。

---

## 安装并配置Express

Express 是基于 Node.js 的后台框架，我选择使用它来作为后端框架的原因是我不会用其他方式。

具体的介绍不多说，这里是中文官网链接。

运行命令 `npm install --save-dev express` 即可安装 Express。

现在我们来调整一下上面建立的文件目录结构，增加一个后台入口和后台文件存放目录。

```
┍client（项目源文件目录）
┃ ┝actions（action文件目录）
┃ ┝components（UI组件文件目录）
┃ ┝containers（容器组件文件目录）
┃ ┝utils（公用资源模块目录）
┃ ┝reducers（reducer文件目录）
┃ ┕view（对应后台的view视图文件的入口文件存放目录）
┃   ┝index1.jsx（view1入口文件）
┃   ┕index2.jsx（view2入口文件）
┝server（服务器文件目录）
┃ ┝views（view文件存放目录）
┃ ┕routes（route文件存放目录）
┝build（编译文件存放目录，可不创建）
┝node_modules（模块仓库）
┝app.js（后台入口文件）
┝.babelrc（babel配置文件）
┝package.json（项目描述文件）
┕webpack.config.js（webpack配置文件）
```

以上目录结构便是调整后的样子。目录文件依次建立好之后，在根目录的 `app.js` 文件中键入如下内容。

```javascript
var express = require('express');
var app = express();
app.get('/', function (req, res) {
  res.send('Hello World!');
});
var server = app.listen(3000, function () {
  var port = server.address().port;
  console.log('Example app listening at Port %s', port);
});
```

然后在命令行中输入指令 `node app.js` 当命令行输出 `Example app listening at Port 3000` 的时候，在浏览器中输入地址，如果能够在浏览器看到 `Hello World!`，那就代表 Express 环境搭好了。

---

## Express结合Webpack的全栈自动刷新

> 文章来源
> [Express结合Webpack的全栈自动刷新](http://acgtofe.com/posts/2016/02/full-live-reload-for-express-with-webpack)
> [入门 Webpack，看这篇就够了](https://segmentfault.com/a/1190000006178770#articleHeader6)

Redux+React 实现的 flux 前端框架（下文统称前端框架）实现了对状态的自洽管理和页面更新生成。但是作为传统的 C/S 结构，我们还需要安全封闭的后台环境来实现对核心逻辑的封装和数据库的保存。前端框架通过 post 接口向后台调用需要更新的数据，然后通过客户端的缓存文件实现页面更新和显示。

那么，在单人开发的时候，前端框架的本地服务器预览和后台的本地服务器模拟实现交接要如何做到呢，这里来整理一下。

前端框架的预览使用的是 `webpack-dev-server` 插件，该插件其实是使用了 express 生成了一个小型的本地页面来实行页面显示。我们只需要通过后台端的 express 启动文件中启动 `webpack-dev-middleware` 和 `webpack-hot-middleware` 这两个插件来替代 `webpack-dev-server` 的功能就可以了。

```bash
npm install --save-dev webpack-dev-middleware webpack-hot-middleware
```

安装完毕之后我们可以直接使用上面为 `webpack-dev-server` 所使用的设置文件生成一份副本 `webpack.config.dev.js` 来作为 `webpack-dev-middleware` 的配置文件。

```javascript
// webpack.config.dev.js
var path = require('path');
var webpack = require('webpack');
var HtmlwebpackPlugin = require('html-webpack-plugin');
var ROOT_PATH = path.resolve(__dirname);
var APP_PATH = path.resolve(ROOT_PATH, 'client');
var Mod_PATH = path.resolve(ROOT_PATH, 'node_modules');
var publicPath = 'http://localhost:3000/';
var hotMiddlewareScript = 'webpack-hot-middleware/client?reload=true';
var devConfig = {
    entry: {
        page1: ['./client/views/index1', hotMiddlewareScript],
        page2: ['./client/views/index2', hotMiddlewareScript]
    },
    output: {
        filename: './[name]/bundle.js',
        path: path.resolve('./public'),
        publicPath: publicPath,
    },
    devtool: 'source-map',
    module: {
        loaders: [
        {
          test: /\.(gif|jpg|png|woff|svg|eot|ttf)\??.*$/, 
          loader: 'url-loader?limit=50000&name=[path][name].[ext]'
        }, {
          test: /\.(js|jsx)$/,
          loader: 'babel-loader',
          include: APP_PATH      
        },{
          test: /\.css?$/,
          loaders: ['style-loader', 'css-loader']
        },{
          test: /\.less?$/,
          loaders: ['style-loader', 'css-loader', 'less-loader']
        },]
    },
    resolve: {
      extensions: ['.js', '.jsx', ".scss", ".less"],
      // 设置常用模块别名，优化webpack检索时间
      alias: {
        // 'jquery': 'client/utils/javascript/jquery-3.1.1.min.js',
        'react': path.join(Mod_PATH, 'react/dist/react.min.js'),
        'react-dom': path.join(Mod_PATH, 'react-dom/dist/react-dom.min.js'),
        'react-redux': path.join(Mod_PATH, 'react-redux/dist/react-redux.min.js'),
        'redux': path.join(Mod_PATH, 'redux/dist/redux.min.js'),
        'react-bootstrap': path.join(Mod_PATH, 'react-bootstrap/dist/react-bootstrap.min.js'),
      }
    },
    plugins: [
        // new webpack.OccurenceOrderPlugin(),
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoEmitOnErrorsPlugin(),
        new HtmlwebpackPlugin({
          title: 'My first react app',
        }),
        new webpack.ProvidePlugin({
          $:"jquery",
          jQuery:"jquery",
          "window.jQuery":"jquery"
        })
    ]
};
module.exports = devConfig;
```

要注意上面这份配置文件中的 devServer 配置项被去掉了，因为和预览服务器相关的属性设置被放到了中间件调用的参数当中去。

和热启动有关的还有一个插件 `react-transform-hmr` 下面介绍安装及配置方法。

```bash
npm install --save-dev babel-plugin-react-transform react-transform-hmr
```

安装完毕之后配置 `.babelrc` 文件：

```javascript
{
  "presets": ["react", "es2015"],
  "env": {
    "development": {
    "plugins": [["react-transform", {
       "transforms": [{
         "transform": "react-transform-hmr",
         
         "imports": ["react"],
         
         "locals": ["module"]
       }]
     }]]
    }
  }
}
```

现在使用 React 的时候，可以热加载模块了。

配置文件设置完成了之后就要到后台入口文件 `app.js` 中设置和中间件有关的代码了，不过要先安装模板引擎 `ejs npm install --save-dev ejs`


```javascript
var express = require('express');
var app = express();
var bs = require('browser-sync').create();
//nodeJs模板引擎，选用ejs(需要安装)，如下配置才可正常使用.html文件作为view入口
app.engine('.html', require('ejs').__express);
//change the template main catelog
app.set('views',__dirname+'server/views/');
app.set('view engine','html')
// 设置以开发模式启动项目
app.locals.env = process.env.NODE_ENV || 'dev';
app.locals.reload = false;
var isDev = process.env.NODE_ENV !== 'production';
// 检查当前是否以开发模式启动项目，若是的话，启用webpack中间件生成预览服务器
if(isDev){
    const webpack = require('webpack');
    const webpackDevMiddleware = require('webpack-dev-middleware');
    const webpackHotMiddleware = require('webpack-hot-middleware');
    const config = require('./webpack.config.dev.js');
    const compiler=webpack(config);
    app.use(webpackDevMiddleware(compiler,{
      publicPath:config.output.publicPath,
      noInfo: false,
      stats:{
        colors:true
      }
    }));
    app.use(webpackHotMiddleware(compiler));
    app.get('/', function (req, res) {
      res.render('./index.html');
    });
    app.listen(3000,function(){
        console.log("App (dev) is running on port 3000.");
    })
}
```

`env.NODE_ENV` 是当前项目的运行环境参数，可通过命令行命令 `set NODE_ENV = development` 来设置，也可以在代码中通过 `app.locals.env = development` 来设置。上面代码的意思便是在开发模式中启动 webpack-dev 本地服务器进行本地预览，关于开发模式和上线模式之间的参数设置技巧可以看下这篇文章[入门 Webpack，看这篇就够了](https://segmentfault.com/a/1190000006178770#articleHeader6)。

上面设置了在启动 Express 服务器的同时启动 webpack-dev 的本地预览服务器并实现在前端框架文件有改动的时候热启动重启插件或者刷新页面。

下面来说说如何监听后台文件更改和自动刷新。常用的思路是 `supervisor` 或者 `browser-sync` 这里使用 `browser-sync`。

```bash
npm install --save-dev browser-sync
```

安装好插件之后，修改我们刚才的 app.js 文件：

```javascript
var express = require('express');
var app = express();
var bs = require('browser-sync').create();
//nodeJs模板引擎，选用ejs(需要安装)，如下配置才可正常使用.html文件作为view入口
app.engine('.html', require('ejs').__express);
//change the template main catelog
app.set('views',__dirname+'server/views/');
app.set('view engine','html')
// 设置以开发模式启动项目
app.locals.env = process.env.NODE_ENV || 'dev';
app.locals.reload = false;
var isDev = process.env.NODE_ENV !== 'production';
// 检查当前是否以开发模式启动项目，若是的话，启用webpack中间件生成预览服务器
if(isDev){
    const webpack = require('webpack');
    const webpackDevMiddleware = require('webpack-dev-middleware');
    const webpackHotMiddleware = require('webpack-hot-middleware');
    const config = require('./webpack.config.dev.js');
    const compiler=webpack(config);
    app.use(webpackDevMiddleware(compiler,{
      publicPath:config.output.publicPath,
      noInfo: false,
      stats:{
        colors:true
      }
    }));
    app.use(webpackHotMiddleware(compiler));
    app.get('/', function (req, res) {
      res.render('./index.html');
    });
    app.listen(3000,function(){
        bs.init({
          open: false,
          ui: false,
          notify: false,
          proxy: 'loaclhost:3000',
          files: ['./client/**'],
          port: 8080
        })
        console.log("App (dev) is going to be running on port 8080 (by browsersync).");
    })
}
```

```json
...
"scripts": {
  "dev": "webpack-dev-server --progress --profile --colors --hot",
  "build": "webpack --progress --profile --colors",
  "test": "karma start",
  "kaifa": "set NODE_ENV = development && node app.js",
  "pd": "set NODE_ENV = production && node app.js"
},
...
"devDependencies": {
  "babel-core": "^6.23.1",
  "babel-loader": "^6.3.2",
  "babel-plugin-react-transform": "^2.0.2",
  "babel-preset-es2015": "^6.22.0",
  "babel-preset-react": "^6.23.0",
  "babel-preset-stage-2": "^6.22.0",
  "browser-sync": "^2.18.8",
  "css-loader": "^0.26.1",
  "ejs": "^2.5.6",
  "express": "^4.14.1",
  "file-loader": "^0.10.0",
  "html-webpack-plugin": "^2.28.0",
  "jquery": "^3.1.1",
  "less": "^2.7.2",
  "less-loader": "^2.2.3",
  "node-less": "^1.0.0",
  "react": "^15.4.2",
  "react-dom": "^15.4.2",
  "react-redux": "^5.0.2",
  "react-transform-hmr": "^1.0.4",
  "redux": "^3.6.0",
  "style-loader": "^0.13.1",
  "url-loader": "^0.5.7",
  "webpack": "^2.2.1",
  "webpack-dev-middleware": "^1.10.1",
  "webpack-hot-middleware": "^2.17.0"
}
...
```

最后贴出 `package.json` 里面的 `script` 配置和依赖列表，`dev` 命令为启动 webpack-dev-server 的本地预览服务器，`kaifa` 命令为启动 browser-sync 本地服务器。

大概的流程便是这样，不过关于前后台结合的具体原理的流程我还有不理解的地方，慢慢会继续更新的。教程文档各有不同，框架接口也在不断更新调整，要多逼自己去看官方文档和亲手测试。
