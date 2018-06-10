# Webpack4从0配置TS+SCSS
## 配置依赖
webpack4出来有一段时间，听说改动挺大的，但是也没有去真正的接触过，现在就来体验一下webpack4有哪些不同吧！  

[DEMO](https://github.com/itagn/webpack-demo)

`webpack4`把`webpack-cli`独立成一个包出来，所以还需要下载`webpack-cli`
按照之前的使用先安装webpack的依赖
```cmd
$ npm install webpack webpack-cli -D
```

`webpack`需要的插件依赖
```cmd
$ npm install html-webpack-plugin webpack-dev-server mini-css-extract-plugin -D
```
然后把一些通用的依赖也安装了  

`babel`的依赖
```cmd
$ npm install babel-core babel-loader babel-preset-env babel-preset-stage-2 -D
```

`typescript`的依赖
```cmd
$ npm install typescript ts-loader -D
```

`scss`的依赖
```cmd
$ npm install node-sass sass-loader -D
```

初始化配置文件`tsconfig.json`和`package.json`  

```cmd
$ npm i typescript -g
$ tsc --init
$ npm init -y
```
手动创建基础文件 `index.html` 和 `index.ts` 和 `index.scss` 

`index.html`
```html
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Webpack4Demo</title>
</head>
<body>
  <div id="demo"></div>
</body>
</html>
```

`index.ts`
```javascript
import './index.scss'

interface Author {
  name: string;
  age: number;
  phone?: string,
  github: string;
}
const me: Author = {
  name: 'itagn',
  age: 23,
  github: 'https://github.com/itagn'
}
console.log(me)
```

`index.scss`
```scss
@mixin pos-mid {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}

$px: 400px;

#demo {
  border: 2px solid #666;
  text-align: center;
  height: $px;
  width: $px;
  cursor: pointer;
  @include pos-mid;
}
```

## 配置package.json和.babelrc文件
打开`package.json`，在`"script"`里面添加`"dev"`和`"build"`  
这样我们就可以通过启动npm run dev启动开发环境，通过启动npm run build打包文件，体验了一把脚手架的感觉  
```json
"scripts": {
    "dev": "webpack-dev-server --mode development --watch --open",
    "build": "webpack --mode production"
  },
```
这里的的配置解释

    --mode development 是让webpack4进入到开发环境
    --mode production 是让webpack4进入到生产环境
    --watch 是监听文件变化，如果开发环境文件有修改，会重新进行开发环境的编译
    --open 是启动开发环境后自动打开浏览器


创建 `·babelrc` 文件
```javascript
{
    "presets": [
        "env",
        "stage-2"
    ],
    "plugins": []
}
```
## 配置webpack.config.js
新建`webpack.config.js`文件  
由于webpack4用有些插件有问题，这里引入了 `mini-css-extract-plugin` 去代替 `extract-text-webpack-plugin` 完成单独打包css文件  
```cmd
$ npm intall mini-css-extract-plugin -D
```
```javascript
const MiniCssExtractPlugin = require("mini-css-extract-plugin")
const path = require('path')

const webpackConfig = {
  context: path.resolve(__dirname, '../'),
  devServer: {
    host: 'localhost',
    port: 4000
  },
  entry: {
    bundle: './index.ts'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[hash].js'
  },
  resolve: {
    extensions: [".js", ".ts", ".json"]
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: {
          loader: 'ts-loader'
        }
      },
      {
        test: /\.jsx?$/,
        use:{
          loader: 'babel-loader'
        }
      },
      {
        test: /\.(css|sass|scss)$/,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          "sass-loader"
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: 'index.html',
      filename: 'index.html'
    }),
    new MiniCssExtractPlugin({
      filename: "[name].[hash].css",
      chunkFilename: "[id].css"
    }),
  ]
}

module.exports = webpackConfig
```
## 运行开发环境和打包
上面简单的配置了一个简单的脚手架，开发环境可以直接运行typescript和scss的语法

- 进入开发环境时启动 `npm run dev`  
- 进入打包时启动 `npm run build`  

## End
笔者研究了vue-cli的配置，结合 webpack4 完善了本文配置  [源码地址](https://github.com/itagn/webpack-demo)  
由于对开发环境和线上环境的要求不一致，所以 webpack 的配置区分成两个文件是很有必要的  
基于vue-cli脚手架的目录结构，使用 webpack4 去实现了一遍简单的打包配置  

作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn











