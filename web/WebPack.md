# Webpack 常用配置整理

## 1. 基本结构

```js
// webpack.config.js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  mode: 'development',

  entry: './src/main.js',

  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    clean: true
  },

  module: {
    rules: []
  },

  plugins: [],

  devServer: {},

  resolve: {},

  optimization: {},

  devtool: 'source-map'
}
```

------

# 2. mode：构建模式

用于指定当前构建环境。

```js
mode: 'development'
```

常用值：

| 值            | 说明                       |
| ------------- | -------------------------- |
| `development` | 开发环境，构建快，方便调试 |
| `production`  | 生产环境，会自动压缩和优化 |
| `none`        | 不使用默认优化             |

常用写法：

```js
mode: 'development'
```

或：

```js
mode: 'production'
```

------

# 3. entry：入口文件

指定 Webpack 从哪个文件开始打包。

```js
entry: './src/main.js'
```

常见写法：

```js
entry: './src/index.js'
```

多入口写法：

```js
entry: {
  main: './src/main.js',
  admin: './src/admin.js'
}
```

面试重点：

> `entry` 表示打包入口，Webpack 会从入口文件开始分析依赖关系。

------

# 4. output：输出配置

指定打包后的文件输出到哪里。

```js
output: {
  path: path.resolve(__dirname, 'dist'),
  filename: 'bundle.js',
  clean: true
}
```

常用配置：

| 配置项       | 作用                     |
| ------------ | ------------------------ |
| `path`       | 输出目录，必须是绝对路径 |
| `filename`   | 输出文件名               |
| `clean`      | 每次打包前清空输出目录   |
| `publicPath` | 配置静态资源访问路径     |

示例：

```js
output: {
  path: path.resolve(__dirname, 'dist'),
  filename: 'js/[name].[contenthash].js',
  clean: true
}
```

------

# 5. module.rules：Loader 配置

用于处理不同类型的文件。

Webpack 默认只能直接处理：

```text
JavaScript
JSON
```

其他文件需要 loader。

## 5.1 处理 CSS

```js
module: {
  rules: [
    {
      test: /\.css$/,
      use: ['style-loader', 'css-loader']
    }
  ]
}
```

说明：

| Loader         | 作用                                 |
| -------------- | ------------------------------------ |
| `css-loader`   | 解析 CSS 文件                        |
| `style-loader` | 把 CSS 插入到页面的 `<style>` 标签中 |

执行顺序：

```text
从右到左执行
先 css-loader
再 style-loader
```

------

## 5.2 处理 Less

```js
{
  test: /\.less$/,
  use: ['style-loader', 'css-loader', 'less-loader']
}
```

执行顺序：

```text
less-loader -> css-loader -> style-loader
```

------

## 5.3 处理 Sass

```js
{
  test: /\.scss$/,
  use: ['style-loader', 'css-loader', 'sass-loader']
}
```

------

## 5.4 处理 JavaScript

```js
{
  test: /\.js$/,
  exclude: /node_modules/,
  use: 'babel-loader'
}
```

常见作用：

```text
把高版本 JS 转成兼容性更好的 JS
```

------

## 5.5 处理 TypeScript

```js
{
  test: /\.ts$/,
  exclude: /node_modules/,
  use: 'ts-loader'
}
```

------

## 5.6 处理图片资源

Webpack 5 常用内置资源模块，不一定需要 `file-loader`、`url-loader`。

```js
{
  test: /\.(png|jpg|jpeg|gif|svg)$/i,
  type: 'asset'
}
```

常见类型：

| 类型             | 作用                          |
| ---------------- | ----------------------------- |
| `asset/resource` | 输出为单独文件                |
| `asset/inline`   | 转成 base64                   |
| `asset`          | 自动判断是单独文件还是 base64 |

示例：

```js
{
  test: /\.(png|jpg|jpeg|gif|svg)$/i,
  type: 'asset',
  parser: {
    dataUrlCondition: {
      maxSize: 8 * 1024
    }
  }
}
```

表示小于 `8KB` 的图片转成 base64。

------

# 6. plugins：插件配置

插件用于增强 Webpack 打包流程。

## 6.1 html-webpack-plugin

自动生成 HTML，并自动引入打包后的 JS/CSS。

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')

plugins: [
  new HtmlWebpackPlugin({
    template: './public/index.html',
    filename: 'index.html',
    title: 'Webpack App'
  })
]
```

常用配置：

| 配置项     | 作用             |
| ---------- | ---------------- |
| `template` | 指定 HTML 模板   |
| `filename` | 输出 HTML 文件名 |
| `title`    | 页面标题         |
| `inject`   | 控制资源注入位置 |
| `minify`   | 压缩 HTML        |

------

## 6.2 mini-css-extract-plugin

生产环境中把 CSS 提取成单独文件。

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

plugins: [
  new MiniCssExtractPlugin({
    filename: 'css/[name].[contenthash].css'
  })
]
```

对应 loader：

```js
{
  test: /\.css$/,
  use: [MiniCssExtractPlugin.loader, 'css-loader']
}
```

开发环境常用：

```js
style-loader
```

生产环境常用：

```js
MiniCssExtractPlugin.loader
```

------

## 6.3 DefinePlugin

定义环境变量。

```js
const webpack = require('webpack')

plugins: [
  new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify('production')
  })
]
```

------

# 7. devServer：开发服务器配置

用于配置本地开发服务器。

```js
devServer: {
  host: 'localhost',
  port: 8080,
  open: true,
  hot: true,
  historyApiFallback: true,
  proxy: {
    '/api': {
      target: 'http://localhost:3000',
      changeOrigin: true,
      pathRewrite: {
        '^/api': ''
      }
    }
  }
}
```

常用配置：

| 配置项               | 作用                       |
| -------------------- | -------------------------- |
| `host`               | 主机地址                   |
| `port`               | 端口号                     |
| `open`               | 启动后自动打开浏览器       |
| `hot`                | 开启热更新                 |
| `historyApiFallback` | 解决前端路由刷新 404       |
| `proxy`              | 配置开发环境代理，解决跨域 |

------

## proxy 示例

前端请求：

```js
fetch('/api/users')
```

Webpack 配置：

```js
devServer: {
  proxy: {
    '/api': {
      target: 'http://localhost:3000',
      changeOrigin: true,
      pathRewrite: {
        '^/api': ''
      }
    }
  }
}
```

实际请求会被代理到：

```text
http://localhost:3000/users
```

------

# 8. resolve：模块解析配置

用于配置模块查找规则。

```js
resolve: {
  extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],
  alias: {
    '@': path.resolve(__dirname, 'src')
  }
}
```

常用配置：

| 配置项       | 作用                   |
| ------------ | ---------------------- |
| `extensions` | 导入文件时可以省略后缀 |
| `alias`      | 配置路径别名           |

示例：

```js
import App from './App'
```

Webpack 会尝试查找：

```text
./App.js
./App.jsx
./App.ts
./App.tsx
```

路径别名：

```js
alias: {
  '@': path.resolve(__dirname, 'src')
}
```

使用：

```js
import Button from '@/components/Button'
```

等价于：

```js
import Button from './src/components/Button'
```

------

# 9. optimization：打包优化配置

用于优化最终打包结果。

```js
optimization: {
  minimize: true,
  splitChunks: {
    chunks: 'all'
  },
  runtimeChunk: 'single'
}
```

常用配置：

| 配置项         | 作用                    |
| -------------- | ----------------------- |
| `minimize`     | 是否压缩代码            |
| `splitChunks`  | 代码分割，提取公共依赖  |
| `runtimeChunk` | 抽离 Webpack 运行时代码 |

------

## splitChunks 示例

```js
optimization: {
  splitChunks: {
    chunks: 'all'
  }
}
```

作用：

```text
把公共代码、第三方依赖拆分出来，减少重复打包，提高缓存利用率。
```

------

# 10. devtool：Source Map 配置

用于控制是否生成 Source Map。

```js
devtool: 'source-map'
```

常用值：

| 配置值                         | 说明                               |
| ------------------------------ | ---------------------------------- |
| `eval-cheap-module-source-map` | 开发环境常用，构建快，方便定位源码 |
| `source-map`                   | 生成完整 Source Map                |
| `hidden-source-map`            | 生成 map 文件，但不暴露给浏览器    |
| `false`                        | 不生成 Source Map                  |

开发环境常用：

```js
devtool: 'eval-cheap-module-source-map'
```

生产环境常用：

```js
devtool: false
```

或：

```js
devtool: 'hidden-source-map'
```

------

# 11. externals：外部依赖配置

用于声明某些依赖不参与打包，而是在运行时从外部获取。

```js
externals: {
  react: 'React',
  'react-dom': 'ReactDOM'
}
```

使用场景：

```text
项目通过 CDN 引入 React，不希望 Webpack 再把 React 打进 bundle。
```

HTML 中：

```html
<script src="https://cdn.jsdelivr.net/npm/react/umd/react.production.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/react-dom/umd/react-dom.production.min.js"></script>
```

Webpack 配置：

```js
externals: {
  react: 'React',
  'react-dom': 'ReactDOM'
}
```

作用：

```text
减小打包体积，避免重复打包第三方库。
```

------

# 12. 完整常用配置示例

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

const isProd = process.env.NODE_ENV === 'production'

module.exports = {
  mode: isProd ? 'production' : 'development',

  entry: './src/main.js',

  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: isProd ? 'js/[name].[contenthash].js' : 'js/[name].js',
    clean: true
  },

  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          isProd ? MiniCssExtractPlugin.loader : 'style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [
          isProd ? MiniCssExtractPlugin.loader : 'style-loader',
          'css-loader',
          'less-loader'
        ]
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      },
      {
        test: /\.(png|jpg|jpeg|gif|svg)$/i,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024
          }
        }
      }
    ]
  },

  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      filename: 'index.html'
    }),
    ...(isProd
      ? [
          new MiniCssExtractPlugin({
            filename: 'css/[name].[contenthash].css'
          })
        ]
      : [])
  ],

  devServer: {
    port: 8080,
    open: true,
    hot: true,
    historyApiFallback: true,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }
  },

  resolve: {
    extensions: ['.js', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },

  optimization: {
    splitChunks: {
      chunks: 'all'
    },
    runtimeChunk: 'single'
  },

  devtool: isProd ? false : 'eval-cheap-module-source-map'
}
```

------

# 13. 面试速记版

| 配置项         | 作用                                   |
| -------------- | -------------------------------------- |
| `mode`         | 指定开发环境或生产环境                 |
| `entry`        | 指定打包入口                           |
| `output`       | 指定打包输出位置和文件名               |
| `module.rules` | 配置 loader，处理 CSS、图片、JS 等文件 |
| `plugins`      | 配置插件，增强打包流程                 |
| `devServer`    | 配置本地开发服务器                     |
| `resolve`      | 配置模块解析规则                       |
| `optimization` | 配置打包优化                           |
| `devtool`      | 配置 Source Map                        |
| `externals`    | 配置外部依赖，不打进 bundle            |

一句话总结：

> Webpack 的核心配置包括入口 `entry`、出口 `output`、loader 配置 `module.rules`、插件配置 `plugins`、开发服务器 `devServer`、模块解析 `resolve`、打包优化 `optimization`、调试映射 `devtool` 和外部依赖 `externals`。loader 负责文件转换，plugin 负责增强构建流程。