# Express 学习笔记

## 1. 项目概览

这个项目是一个小型的 Express 入门示例，覆盖了下面这些常见能力：

- 搭建 Express 服务器
- 使用 ES Module 语法
- 处理 JSON 请求体
- 处理表单请求体
- 处理查询参数和路径参数
- 提供静态资源
- 处理跨域请求
- 接收文件上传
- 使用前端页面和 `test.http` 测试接口

项目入口文件是 `index.js`，前端静态页面在 `public/index.html`，接口测试示例在 `test.http`。

## 2. 依赖与基础配置

### `package.json`

项目使用了这些核心依赖：

- `express`：Web 服务框架
- `cors`：处理跨域
- `multer`：处理 `multipart/form-data`，也就是文件上传

还配置了：

- `"type": "module"`：表示当前项目使用 ES Module，因此代码里可以直接使用 `import`
- `"dev": "nodemon index.js"`：开发时自动重启服务

## 3. 服务器初始化

```js
import express from 'express';
import cors from 'cors';
import multer from 'multer';

const app = express();
const port = 3000;
const upload = multer({ dest: 'uploads/' });
```

这里涉及的知识点：

- `express()` 用来创建应用实例
- `app` 是整个服务的核心对象，后续中间件、路由都挂在它上面
- `port = 3000` 表示服务运行在本地 `3000` 端口
- `multer({ dest: 'uploads/' })` 表示上传的文件先保存到 `uploads` 目录

## 4. 中间件

Express 很多功能都依赖中间件。这个项目里主要用了 4 类。

### 4.1 `express.json()`

```js
app.use(express.json());
```

作用：

- 解析 `Content-Type: application/json` 的请求体
- 解析完成后可以通过 `req.body` 拿到数据

如果不加它，`POST` 的 JSON 数据通常拿不到。

### 4.2 `express.urlencoded({ extended: true })`

```js
app.use(express.urlencoded({ extended: true }));
```

作用：

- 解析表单提交的数据
- 主要处理 `application/x-www-form-urlencoded`

这个项目里的 `test.http` 用了：

```txt
user[name]=xx&user[age]=18
```

当 `extended: true` 时，可以解析这种带嵌套结构的数据，最后在 `req.body` 中得到更接近对象结构的结果。

### 4.3 `express.static('public')`

```js
app.use('/', express.static('public'));
```

作用：

- 把 `public` 目录作为静态资源目录暴露出去
- 浏览器访问 `http://127.0.0.1:3000/` 时，可以直接拿到 `public/index.html`

补充理解：

- 静态资源中间件常用于托管 HTML、CSS、JS、图片等文件
- 这里写成 `app.use('/', ...)`，表示从根路径开始匹配

### 4.4 `cors()`

```js
app.use(cors());
```

作用：

- 允许浏览器跨域访问接口

为什么需要它：

- 页面里使用了 `fetch('http://127.0.0.1:3000/api')`
- 当前页面和接口只要“来源”不同，浏览器就可能触发跨域限制
- `cors()` 会自动设置响应头，允许跨域请求通过

## 5. 路由与请求数据获取

### 5.1 根路由

```js
app.get('/', (req, res) => {
  res.send('Hello World!');
});
```

作用：

- 定义一个 `GET /` 路由
- `res.send()` 用于返回普通文本或 HTML

注意：

- 当前项目前面已经注册了 `express.static('public')`
- 因为 `public/index.html` 存在，所以访问 `/` 时，大多数情况下会优先返回静态页面
- 这说明中间件和路由的注册顺序会影响最终结果

### 5.2 查询参数 `req.query`

```js
app.get('/api', (req, res) => {
  console.log(req.query);
  res.json('Hello World!');
});
```

示例请求：

```http
GET /api?name=xx&age=18
```

知识点：

- URL 中 `?` 后面的内容叫查询参数
- Express 会把它们解析到 `req.query`
- 适合传递筛选、分页、搜索等附加条件

### 5.3 路径参数 `req.params`

```js
app.get('/api/:id', (req, res) => {
  console.log(req.params);
  res.send('Hello World!');
});
```

示例请求：

```http
GET /api/123
```

知识点：

- `:id` 是动态路径参数
- 当访问 `/api/123` 时，`req.params.id` 的值就是 `123`
- 常用于查询某个具体资源，例如用户详情、文章详情

### 5.4 JSON 请求体 `req.body`

```js
app.post('/api', (req, res) => {
  console.log(req.body);
  res.send('Hello Post json!');
});
```

示例请求：

```http
POST /api
Content-Type: application/json
```

知识点：

- 客户端把 JSON 数据放到请求体中发送给服务端
- 服务端通过 `express.json()` 解析后，用 `req.body` 读取

### 5.5 表单请求体 `req.body`

```js
app.post('/api/form', (req, res) => {
  console.log(req.body);
  res.send('Hello Post form!');
});
```

知识点：

- 表单不只有浏览器页面能提交，`test.http`、Postman、Apifox 也都可以模拟
- `application/x-www-form-urlencoded` 是经典表单编码格式
- 这类数据同样会被放到 `req.body` 中

## 6. 文件上传

### 6.1 基本写法

```js
app.post('/api/upload', upload.single('file'), (req, res) => {
  console.log(req.body);
  console.log(req.file);
  res.json({
    message: '上传成功',
    file: req.file,
    body: req.body
  });
});
```

这里是本项目最值得掌握的一部分。

### 6.2 `upload.single('file')` 的含义

- `single` 表示只接收一个文件
- `'file'` 必须和前端表单中 `<input type="file" name="file">` 的 `name` 一致

如果名字对不上，Multer 就匹配不到上传文件。

### 6.3 上传后能拿到什么

- `req.file`：上传文件的信息
- `req.body`：同一个表单里其他普通字段，例如 `username`

因此，一个上传接口通常既能接收文件，也能接收文本字段。

### 6.4 `multipart/form-data`

文件上传表单必须使用：

```html
<form enctype="multipart/form-data">
```

原因：

- 普通表单编码无法正确传输文件二进制内容
- `multipart/form-data` 会把文件和其他字段分段传输

## 7. 前端页面中的知识点

`public/index.html` 主要展示了两类能力。

### 7.1 静态页面托管

浏览器直接访问根路径时，可以看到这个 HTML 页面，说明：

- Express 不只能写接口
- 也可以直接托管前端静态文件

### 7.2 使用 `fetch` 调接口

```js
fetch('http://127.0.0.1:3000/api')
  .then(res => res.json())
  .then(data => console.log(data));
```

知识点：

- `fetch` 用于在浏览器里发起 HTTP 请求
- `res.json()` 用于把响应解析成 JSON
- 这是最基础的前后端联动方式之一

### 7.3 使用表单上传文件

```html
<form action="http://127.0.0.1:3000/api/upload" method="post" enctype="multipart/form-data">
  <input type="text" name="username">
  <input type="file" name="file">
</form>
```

这一段体现了：

- `action` 指定提交到哪个接口
- `method="post"` 表示使用 POST 提交
- `name="username"` 对应后端 `req.body.username`
- `name="file"` 对应后端 `upload.single('file')`

## 8. `test.http` 里的测试知识

这个文件展示了如何快速测试接口：

- 测试带查询参数的 GET 请求
- 测试带路径参数的 GET 请求
- 测试 JSON 格式的 POST 请求
- 测试表单格式的 POST 请求

这类 `.http` 文件常用于 VS Code REST Client 等插件，优点是：

- 请求示例可直接保存在项目里
- 方便复现接口
- 比手写命令更直观

## 9. 这个项目覆盖的核心 Express 知识图谱

可以把它总结成下面这条主线：

1. 用 `express()` 创建服务器
2. 用 `app.use()` 注册中间件
3. 用 `app.get()`、`app.post()` 定义路由
4. 用 `req.query`、`req.params`、`req.body` 获取请求数据
5. 用 `res.send()`、`res.json()` 返回响应
6. 用 `express.static()` 提供静态页面
7. 用 `cors()` 解决跨域
8. 用 `multer` 处理文件上传

## 10. 容易混淆的点

### `req.query`、`req.params`、`req.body` 的区别

- `req.query`：取查询参数，例如 `/api?page=1`
- `req.params`：取路径参数，例如 `/api/123`
- `req.body`：取请求体数据，例如 POST 提交的 JSON 或表单

### `res.send()` 和 `res.json()` 的区别

- `res.send()`：可返回字符串、HTML、对象等，使用范围更广
- `res.json()`：明确返回 JSON，接口开发中更常见

### `express.json()` 和 `express.urlencoded()` 的区别

- `express.json()`：处理 JSON 请求体
- `express.urlencoded()`：处理传统表单编码

### `multer` 不是所有请求都需要

- 只有文件上传相关接口才需要使用 `multer`
- 普通 JSON 接口不需要它

## 11. 学完这个项目后应该掌握什么

如果这个项目已经看懂，至少应该能独立完成下面这些事情：

- 写一个最基础的 Express 服务
- 区分 GET 和 POST 的常见用途
- 从 URL、路径、请求体中取到参数
- 让 Express 返回 JSON 数据
- 托管一个静态页面
- 处理跨域请求
- 完成单文件上传
- 用 `.http` 文件或前端页面测试接口

## 12. 可继续扩展的方向

在这个项目基础上，还可以继续学习：

- 路由拆分，把接口拆到多个文件
- 错误处理中间件
- 连接数据库
- 用户登录与鉴权
- 多文件上传
- 文件类型和大小校验
- 接口返回统一格式
- 使用 `express.Router()`

## 13. 一句话总结

这个项目本质上是一个 Express 入门综合示例，串起了“中间件、路由、参数解析、静态资源、跨域、文件上传、接口测试”这些最常见的后端基础能力。
