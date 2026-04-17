# Koa 教程

## 1. Koa 是什么

Koa 是一个面向 Node.js 的 Web 框架，由 Express 背后的团队设计。它的核心特点是更小、更简洁，强调基于 `async/await` 的中间件模型；同时，Koa 核心本身不内置路由、请求体解析等中间件，而是把这些能力交给生态中的独立中间件来组合。官方文档还说明，当前 Koa 需要 Node.js 18 或更高版本。 ([koajs.com](https://koajs.com/))

## 2. Koa 适合什么场景

Koa 很适合用来开发 API 服务、后台管理系统后端、轻量 Web 服务，以及希望自己掌控中间件组合方式的项目。它的设计偏“底层骨架”，因此灵活性高，但也意味着你通常需要自己选择路由、中间件和工程组织方式。Koa 应用本质上是一个由中间件数组组成的对象，请求到来时，这些中间件会按“栈式”方式组合执行。 ([koajs.com](https://koajs.com/))

------

## 3. 环境准备

### 3.1 安装 Node.js

Koa 官方要求 Node.js 18.0.0 或更高版本。 ([koajs.com](https://koajs.com/))

可以先检查本机版本：

```bash
node -v
npm -v
pnpm -v
```

### 3.2 创建项目

```bash
mkdir koa-demo
cd koa-demo
pnpm init
```

### 3.3 安装 Koa

```bash
pnpm add koa
```

如果你使用 TypeScript，通常还会补充类型与开发依赖。`@koa/router` 现在自带 TypeScript 类型；Koa 的类型一般通过 `@types/koa` 提供。 ([GitHub](https://github.com/koajs/router))

```bash
pnpm add -D typescript ts-node-dev @types/node @types/koa
```

------

## 4. 第一个 Koa 程序

新建 `app.js`：

```js
const Koa = require('koa');

const app = new Koa();

app.use(async (ctx) => {
  ctx.body = 'Hello Koa';
});

app.listen(3000, () => {
  console.log('server running at http://localhost:3000');
});
```

运行：

```bash
node app.js
```

浏览器访问：

```text
http://localhost:3000
```

这就是一个最基本的 Koa 服务。官方示例同样是通过 `app.use(async ctx => { ctx.body = 'Hello World' })` 的方式返回内容。 ([koajs.com](https://koajs.com/))

------

## 5. Koa 的核心概念

## 5.1 Application

`new Koa()` 创建的是应用实例。
这个实例负责维护中间件、监听端口、处理请求。

例如：

```js
const app = new Koa();
```

Koa 官方说明，`app.listen(...)` 只是 `http.createServer(app.callback()).listen(...)` 的语法糖。所以从本质上说，Koa 应用不是 HTTP 服务器本身，而是一个可交给 HTTP 服务器使用的请求处理器。 ([koajs.com](https://koajs.com/))

## 5.2 Context

Koa 每次收到请求，都会创建一个 `ctx` 对象。
`ctx` 是最重要的对象，它把 Node 原生的 `request` 和 `response` 封装到了一起，方便你统一处理请求与响应。官方文档明确说明，`ctx.request` 是 Koa Request，`ctx.response` 是 Koa Response，很多常见属性会直接代理到这两个对象上。 ([koajs.com](https://koajs.com/))

常见写法：

```js
app.use(async (ctx) => {
  console.log(ctx.method);
  console.log(ctx.path);
  console.log(ctx.query);
  ctx.body = 'ok';
});
```

## 5.3 Middleware

Koa 最核心的机制就是中间件。
中间件是一个函数，通常接收两个参数：

- `ctx`
- `next`

其中 `next` 表示“进入下一个中间件”。

Koa 的中间件不是简单地一路向下执行，而是典型的“洋葱模型”：先向下进入后续中间件，等后续中间件执行完成，再回到当前中间件继续往下写的逻辑。官方把这一过程描述为先向下游传递，再向上游回卷。 ([koajs.com](https://koajs.com/))

示例：

```js
const Koa = require('koa');
const app = new Koa();

app.use(async (ctx, next) => {
  console.log('中间件1 - before');
  await next();
  console.log('中间件1 - after');
});

app.use(async (ctx, next) => {
  console.log('中间件2 - before');
  await next();
  console.log('中间件2 - after');
});

app.use(async (ctx) => {
  console.log('中间件3');
  ctx.body = 'Hello';
});

app.listen(3000);
```

访问一次后，控制台顺序通常是：

```text
中间件1 - before
中间件2 - before
中间件3
中间件2 - after
中间件1 - after
```

------

## 6. 常用响应写法

### 6.1 返回字符串

```js
app.use(async (ctx) => {
  ctx.body = 'hello';
});
```

### 6.2 返回对象

```js
app.use(async (ctx) => {
  ctx.body = {
    code: 200,
    message: 'success',
    data: { name: 'Tom' }
  };
});
```

### 6.3 设置状态码

```js
app.use(async (ctx) => {
  ctx.status = 201;
  ctx.body = {
    message: 'created'
  };
});
```

### 6.4 设置响应头

```js
app.use(async (ctx) => {
  ctx.set('X-Custom-Header', 'koa-demo');
  ctx.body = 'ok';
});
```

------

## 7. 常用请求数据获取

### 7.1 获取请求方法和路径

```js
app.use(async (ctx) => {
  console.log(ctx.method);
  console.log(ctx.path);
  ctx.body = 'ok';
});
```

### 7.2 获取查询参数

请求地址：

```text
http://localhost:3000/user?id=1&name=tom
```

代码：

```js
app.use(async (ctx) => {
  console.log(ctx.query);
  console.log(ctx.query.id);
  ctx.body = ctx.query;
});
```

------

## 8. 路由

Koa 核心不内置路由，中大型项目通常配合 `@koa/router` 使用。`@koa/router` 官方仓库说明它支持 `app.get`、`app.post` 风格的路由方式、命名参数、嵌套路由、405/501 处理，并且自带 TypeScript 类型。该包当前要求 Node.js 20+、Koa 2+。 ([GitHub](https://github.com/koajs/router))

### 8.1 安装路由

```bash
pnpm add @koa/router
```

### 8.2 基本使用

```js
const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

router.get('/', async (ctx) => {
  ctx.body = '首页';
});

router.get('/user', async (ctx) => {
  ctx.body = {
    id: 1,
    name: 'Tom'
  };
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);
```

### 8.3 路由参数

```js
router.get('/user/:id', async (ctx) => {
  ctx.body = {
    id: ctx.params.id
  };
});
```

访问：

```text
http://localhost:3000/user/123
```

返回：

```json
{
  "id": "123"
}
```

### 8.4 路由前缀

```js
const router = new Router({
  prefix: '/api'
});

router.get('/users', async (ctx) => {
  ctx.body = ['Tom', 'Jerry'];
});
```

最终路径：

```text
GET /api/users
```

------

## 9. 解析请求体

Koa 核心不负责解析 POST 请求体。常见做法是使用 `@koa/bodyparser`。它会把解析后的内容放到 `ctx.request.body` 上，并支持 `json`、`form`、`text` 类型；官方 README 还特别说明，它不支持 `multipart/form-data`，文件上传应使用 `@koa/multer`。 ([GitHub](https://github.com/koajs/bodyparser))

### 9.1 安装

```bash
pnpm add @koa/bodyparser
```

### 9.2 使用

```js
const Koa = require('koa');
const { bodyParser } = require('@koa/bodyparser');

const app = new Koa();

app.use(bodyParser());

app.use(async (ctx) => {
  if (ctx.method === 'POST') {
    ctx.body = {
      received: ctx.request.body
    };
    return;
  }

  ctx.body = 'send a POST request';
});

app.listen(3000);
```

### 9.3 配合路由

```js
const Koa = require('koa');
const Router = require('@koa/router');
const { bodyParser } = require('@koa/bodyparser');

const app = new Koa();
const router = new Router();

app.use(bodyParser());

router.post('/login', async (ctx) => {
  const { username, password } = ctx.request.body;

  ctx.body = {
    username,
    password,
    message: '登录数据已接收'
  };
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);
```

------

## 10. 静态资源服务

如果要托管图片、HTML、CSS、前端构建产物等静态文件，常用 `koa-static`。官方仓库说明它是 `koa-send` 的封装，使用方式是 `app.use(serve(root, opts))`，支持 `maxage`、`index`、`gzip`、`brotli` 等选项。 ([GitHub](https://github.com/koajs/static))

### 10.1 安装

```bash
pnpm add koa-static
```

### 10.2 使用

```js
const Koa = require('koa');
const serve = require('koa-static');
const path = require('path');

const app = new Koa();

app.use(serve(path.join(__dirname, 'public')));

app.listen(3000);
```

假设 `public` 目录下有 `logo.png`，就可以通过：

```text
http://localhost:3000/logo.png
```

访问。

------

## 11. 错误处理

Koa 项目里，通常会在最外层写一个全局错误处理中间件。

```js
app.use(async (ctx, next) => {
  try {
    await next();
  } catch (error) {
    ctx.status = error.status || 500;
    ctx.body = {
      message: error.message || '服务器内部错误'
    };
  }
});
```

Koa 官方提供了 `ctx.throw()` 这个辅助方法，可以快速抛出带状态码的错误；另外，应用实例也支持通过 `app.on('error', ...)` 监听错误事件。 ([koajs.com](https://koajs.com/))

例如：

```js
router.get('/error', async (ctx) => {
  ctx.throw(400, '参数错误');
});
```

------

## 12. `ctx.state` 的作用

在多个中间件之间传递数据时，推荐使用 `ctx.state`。官方文档把它称为“推荐的命名空间”，适合在中间件链和视图层之间共享信息。 ([koajs.com](https://koajs.com/))

示例：

```js
app.use(async (ctx, next) => {
  ctx.state.user = {
    id: 1,
    name: 'Tom'
  };
  await next();
});

app.use(async (ctx) => {
  ctx.body = ctx.state.user;
});
```

------

## 13. 一个最小 CRUD 示例

下面写一个简化版用户接口。

```js
const Koa = require('koa');
const Router = require('@koa/router');
const { bodyParser } = require('@koa/bodyparser');

const app = new Koa();
const router = new Router({ prefix: '/users' });

app.use(bodyParser());

let users = [
  { id: 1, name: 'Tom' },
  { id: 2, name: 'Jerry' }
];

// 查询所有用户
router.get('/', async (ctx) => {
  ctx.body = {
    code: 200,
    data: users
  };
});

// 查询单个用户
router.get('/:id', async (ctx) => {
  const id = Number(ctx.params.id);
  const user = users.find(item => item.id === id);

  if (!user) {
    ctx.throw(404, '用户不存在');
  }

  ctx.body = {
    code: 200,
    data: user
  };
});

// 新增用户
router.post('/', async (ctx) => {
  const { name } = ctx.request.body;

  if (!name) {
    ctx.throw(400, 'name 不能为空');
  }

  const newUser = {
    id: Date.now(),
    name
  };

  users.push(newUser);

  ctx.status = 201;
  ctx.body = {
    code: 201,
    data: newUser
  };
});

// 修改用户
router.put('/:id', async (ctx) => {
  const id = Number(ctx.params.id);
  const { name } = ctx.request.body;

  const user = users.find(item => item.id === id);
  if (!user) {
    ctx.throw(404, '用户不存在');
  }

  user.name = name ?? user.name;

  ctx.body = {
    code: 200,
    data: user
  };
});

// 删除用户
router.delete('/:id', async (ctx) => {
  const id = Number(ctx.params.id);
  const index = users.findIndex(item => item.id === id);

  if (index === -1) {
    ctx.throw(404, '用户不存在');
  }

  const deleted = users.splice(index, 1);

  ctx.body = {
    code: 200,
    data: deleted[0]
  };
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000, () => {
  console.log('server running at http://localhost:3000');
});
```

------

## 14. 推荐的项目结构

```text
koa-demo/
├─ src/
│  ├─ app.js
│  ├─ routers/
│  │  └─ user.js
│  ├─ controllers/
│  │  └─ userController.js
│  ├─ services/
│  │  └─ userService.js
│  ├─ middlewares/
│  │  └─ errorHandler.js
│  └─ utils/
├─ package.json
└─ pnpm-lock.yaml
```

一种常见职责划分如下：

- `routers`：只负责定义接口路径
- `controllers`：处理请求和响应
- `services`：处理业务逻辑
- `middlewares`：错误处理、日志、鉴权等
- `utils`：工具函数

------

## 15. Koa 与 Express 的简单区别

### 15.1 Koa 更轻

Koa 核心不自带很多常见能力，中间件需要自己选。
这会让项目更灵活，但也需要你更了解整个中间件链路。 ([koajs.com](https://koajs.com/))

### 15.2 Koa 的中间件模型更突出“洋葱模型”

在 Koa 中，`await next()` 前后的逻辑非常重要。
这对日志、鉴权、统一异常处理、响应包装都很方便。 ([koajs.com](https://koajs.com/))

### 15.3 Koa 更偏底层组合

对于初学者，Koa 很适合学习中间件机制和 Node Web 服务的核心流程。
如果你想更清楚地理解请求是怎么一步步流转的，Koa 很适合拿来练手。

------

## 16. TypeScript 版本的最小模板

如果你想用 TS，可以这样写。

### 16.1 安装

```bash
pnpm add koa @koa/router @koa/bodyparser
pnpm add -D typescript ts-node-dev @types/node @types/koa
```

### 16.2 `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "moduleResolution": "Node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist"
  },
  "include": ["src"]
}
```

### 16.3 `src/app.ts`

```ts
import Koa from 'koa';
import Router from '@koa/router';
import { bodyParser } from '@koa/bodyparser';

const app = new Koa();
const router = new Router();

app.use(bodyParser());

router.get('/', async (ctx) => {
  ctx.body = 'Hello Koa + TS';
});

router.post('/login', async (ctx) => {
  const body = ctx.request.body;
  ctx.body = {
    message: 'ok',
    body
  };
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000, () => {
  console.log('server running at http://localhost:3000');
});
```

### 16.4 运行脚本

```
package.json
{
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/app.ts"
  }
}
```

运行：

```bash
pnpm dev
```

------

## 17. 学 Koa 时最容易踩的坑

### 17.1 忘了 `await next()`

如果你写了中间件，但没有正确调用 `next`，后续中间件可能不会执行。

错误示例：

```js
app.use(async (ctx, next) => {
  console.log('before');
  next();
  console.log('after');
});
```

更推荐：

```js
app.use(async (ctx, next) => {
  console.log('before');
  await next();
  console.log('after');
});
```

### 17.2 以为 Koa 自带路由和 body 解析

Koa 核心不内置这些能力，需要自己装中间件。 ([koajs.com](https://koajs.com/))

### 17.3 直接操作原生 `res.write()`、`res.end()`

Koa 官方文档明确建议不要绕过它自己的响应处理流程去直接操作这些底层方法，否则容易破坏 Koa 的响应机制。 ([koajs.com](https://koajs.com/))

### 17.4 上传文件还用 `@koa/bodyparser`

`@koa/bodyparser` 不支持 `multipart/form-data`，文件上传场景应该换成 `@koa/multer`。 ([GitHub](https://github.com/koajs/bodyparser))

------

## 18. 一个完整入门组合

对于初学者，最常见的一套组合是：

```bash
pnpm add koa @koa/router @koa/bodyparser koa-static
```

这套组合分别负责：

- `koa`：应用核心
- `@koa/router`：路由
- `@koa/bodyparser`：解析 JSON、表单、文本请求体
- `koa-static`：静态资源服务

这也是你写一个基础前后端分离后端时最常见的起点。 ([koajs.com](https://koajs.com/))

------

## 19. 学习顺序建议

建议按下面顺序学：

1. 先学 `new Koa()`、`app.use()`、`ctx.body`
2. 再学中间件执行顺序和 `await next()`
3. 再学 `@koa/router`
4. 再学 `@koa/bodyparser`
5. 再学错误处理、统一返回格式
6. 最后再拆分路由、控制器、业务层

这样会更顺。

------

## 20. 总结

Koa 的核心思想可以概括成三点：

1. 用尽量小的核心做 Web 服务骨架
2. 用 `async/await` 实现清晰的中间件链
3. 用独立中间件自由组合路由、请求体解析、静态资源等能力 ([koajs.com](https://koajs.com/))

如果你是初学者，可以先把这几个最关键的东西记住：

- `app.use()` 是注册中间件
- `ctx` 是每个请求的上下文
- `await next()` 体现洋葱模型
- `ctx.body` 是最常见的返回方式
- 路由通常配合 `@koa/router`
- POST 请求体通常配合 `@koa/bodyparser`

在真正做项目时，再逐步加入日志、鉴权、数据库、参数校验、文件上传和接口分层。

如果你需要，我可以继续给你整理下一份
《Koa + React 前后端分离项目实战.md》。