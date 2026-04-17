# 路径 path 学习笔记

## 1. 先分清两类“路径”

开发里经常把“路径”混在一起说，但实际上至少有两套东西：

- 文件系统路径：给操作系统和 Node.js 用，例如 `D:\project\src\index.js`
- URL 路径：给浏览器和服务器通信用，例如 `/api/user?id=1`

这两者看起来都像“路径”，但含义和规则并不一样。

一句话记忆：

- `path` 模块处理的是本地文件路径
- `URL` 处理的是网页地址

## 2. Node.js 中的路径

Node.js 里说“路径”，默认指文件系统路径，也就是文件或文件夹在磁盘上的位置。

常见形式有两种：

- 相对路径
- 绝对路径

## 3. 相对路径

相对路径是“相对于某个参照位置”的路径。

例如：

```js
./index.js
index.js
../images/logo.png
./public/css/main.css
```

这里的符号含义：

- `./`：当前目录
- `../`：上一级目录

### 3.1 相对路径的关键问题

相对路径最容易出错的地方是：它到底相对于谁。

在 Node.js 里，很多相对路径默认是相对于“当前工作目录” `process.cwd()`，不一定是当前文件所在目录。

例如：

```js
console.log(process.cwd());
```

如果你在 `D:\demo` 目录执行：

```bash
node src/app.js
```

那很多 `./xxx` 实际参考的是 `D:\demo`，不是 `src`。

### 3.2 相对路径的优点和问题

优点：

- 写起来短
- 适合模块引用和目录结构明确的场景

问题：

- 很依赖当前执行位置
- 项目一旦移动、脚本执行入口变化，就容易出错

## 4. 绝对路径

绝对路径是从根位置开始写的完整路径。

Windows 示例：

```txt
D:\mygithub\markdown\web\express.md
```

Linux / macOS 示例：

```txt
/usr/local/bin/node
```

特点：

- 不依赖当前工作目录
- 指向更明确
- 更适合拼接配置、读写文件、做部署脚本

## 5. `__dirname`、`__filename` 和 ESM 中的替代方案

### 5.1 CommonJS 里的 `__dirname`

在 CommonJS 模块中：

```js
console.log(__dirname);
console.log(__filename);
```

含义：

- `__dirname`：当前文件所在目录的绝对路径
- `__filename`：当前文件本身的绝对路径

它们最大的价值是：

- 让路径相对于“当前文件”而不是“命令执行目录”

例如：

```js
const filePath = path.join(__dirname, 'public', 'index.html');
```

这样不管你从哪里启动 Node，路径都更稳定。

### 5.2 ES Module 里没有 `__dirname`

如果项目使用：

```json
"type": "module"
```

那么默认是 ES Module，这时不能直接用 `__dirname`。

需要这样写：

```js
import path from 'path';
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
```

解释：

- `import.meta.url` 拿到当前模块的 `file://` URL
- `fileURLToPath()` 把 URL 转成系统路径
- `path.dirname()` 再取出目录名

这是 ESM 里最常见的 `__dirname` 替代写法。

## 6. `process.cwd()` 和 `__dirname` 的区别

这是路径题里最容易考混的点。

### `process.cwd()`

- 当前 Node 进程启动时所在的工作目录
- 更像“你现在站在哪”

### `__dirname`

- 当前代码文件所在目录
- 更像“这个文件住在哪”

示例理解：

假设文件在：

```txt
D:\demo\src\app.js
```

你在：

```txt
D:\demo
```

执行：

```bash
node src/app.js
```

那么通常：

- `process.cwd()` 是 `D:\demo`
- `__dirname` 是 `D:\demo\src`

所以：

- 想基于“运行命令的位置”找文件，用 `process.cwd()`
- 想基于“当前脚本文件的位置”找文件，用 `__dirname`

## 7. 网页中的 URL 路径

网页里的路径不是磁盘路径，而是 URL 的一部分。

例如：

```txt
https://example.com:8080/api/user?id=1#top
```

可以拆成：

- 协议：`https`
- 域名：`example.com`
- 端口：`8080`
- 路径：`/api/user`
- 查询参数：`?id=1`
- 哈希：`#top`

### 7.1 URL 中的路径

例如：

```txt
/api/user
/articles/1001
/images/logo.png
```

这个路径表示的是“向服务器请求哪个资源”，不是本地磁盘里的真实路径。

### 7.2 URL 路径和文件路径不要混用

例如：

- `C:\Users\test\a.txt` 是文件路径
- `/api/list?page=1` 是 URL

它们的用途不同：

- 文件路径给操作系统和 Node.js 用
- URL 给浏览器和服务器通信用

### 7.3 网页中的绝对路径

网页里的绝对路径，本质上是浏览器拿到后，可以直接定位到目标资源，或者只需要补全极少一部分信息就能定位资源。

这类写法可靠性更强，也更容易理解，在项目中运用较多。

常见写法有 3 种：

#### 第一种：完整绝对 URL

```txt
http://atguigu.com/web
```

特点：

- 这是最完整的写法
- 浏览器可以直接向目标资源发送请求
- 网站的外链经常用这种形式

#### 第二种：协议相对路径

```txt
//atguigu.com/web
```

特点：

- 它没有写协议
- 浏览器会自动使用当前页面的协议去补全

例如当前页面是：

```txt
https://www.example.com/index.html
```

那么它最终会变成：

```txt
https://atguigu.com/web
```

这种写法常见于大型网站，用来同时兼容 `http` 和 `https`。

#### 第三种：根路径相对的绝对路径

```txt
/web
```

特点：

- 它没有写协议、主机名、端口
- 浏览器会自动把当前页面的协议、主机名、端口拼接上去

例如当前页面是：

```txt
http://www.atguigu.com:8080/course/h5.html
```

那么：

```txt
/web
```

最终会变成：

```txt
http://www.atguigu.com:8080/web
```

这种写法在站内资源引用里非常常见。

### 7.4 网页中的相对路径

网页里的相对路径在发送请求前，需要先和当前页面 URL 的路径部分进行计算，得到完整 URL 后再发送请求。

学习阶段和页面资源引用中，这类写法很常见。

假设当前页面 URL 是：

```txt
http://www.atguigu.com/course/h5.html
```

下面这些相对路径会被解析成：

| 形式 | 最终 URL |
| --- | --- |
| `./css/app.css` | `http://www.atguigu.com/course/css/app.css` |
| `js/app.js` | `http://www.atguigu.com/course/js/app.js` |
| `../img/logo.png` | `http://www.atguigu.com/img/logo.png` |
| `../../mp4/show.mp4` | `http://www.atguigu.com/mp4/show.mp4` |

可以这样理解：

- `./` 表示当前页面所在目录
- 不写 `./` 时，默认也是从当前目录开始
- `../` 表示回到上一级目录
- `../../` 表示回到上两级目录

浏览器处理相对路径时，参考的是当前页面 URL 的路径，不是本地电脑里的文件夹结构。

### 7.5 网页路径的记忆规则

可以直接这样记：

- `http://...`：完整绝对路径，直接访问
- `//...`：协议跟当前页面走
- `/...`：协议、主机名、端口跟当前页面走
- `./...` 或不写 `./`：从当前目录开始找
- `../...`：回到上一级目录再找

所以网页里的绝对路径和相对路径，本质区别就在于：

- 绝对路径定位更直接，依赖当前页面信息更少
- 相对路径需要基于当前页面 URL 进行计算

## 8. Path 模块的作用

Node.js 提供了内置模块 `path` 来专门处理路径。

导入方式：

```js
import path from 'path';
```

或者 CommonJS：

```js
const path = require('path');
```

它的作用主要是：

- 拼接路径
- 解析路径
- 取目录名、文件名、扩展名
- 标准化路径格式
- 处理跨平台路径差异

## 9. 常用 API

## 9.1 `path.join()`

作用：

- 把多个路径片段安全拼接起来

```js
path.join('public', 'images', 'logo.png');
```

结果类似：

```txt
public\images\logo.png
```

特点：

- 自动处理分隔符
- 会顺便处理多余的 `/` 或 `\`

典型用法：

```js
const filePath = path.join(__dirname, 'public', 'index.html');
```

### `join` 的特点

- 更像“单纯拼接路径”
- 常用于构造文件路径

## 9.2 `path.resolve()`

作用：

- 把路径解析成绝对路径

```js
path.resolve('public', 'index.html');
```

它会基于当前工作目录，生成一个绝对路径。

例如当前工作目录是 `D:\demo`，结果可能是：

```txt
D:\demo\public\index.html
```

### `resolve` 的特点

- 从右往左处理参数
- 一旦遇到绝对路径，会把它前面的内容丢掉
- 最终结果通常是绝对路径

示例：

```js
path.resolve('a', 'b', 'c');       // 变成绝对路径
path.resolve('/x', 'y', 'z');      // 结果从 /x 开始
```

### `join` 和 `resolve` 的区别

- `join()`：更偏向路径拼接
- `resolve()`：更偏向得到最终绝对路径

## 9.3 `path.dirname()`

作用：

- 获取路径中的目录部分

```js
path.dirname('D:\\demo\\src\\app.js');
```

结果：

```txt
D:\demo\src
```

## 9.4 `path.basename()`

作用：

- 获取文件名部分

```js
path.basename('D:\\demo\\src\\app.js');
```

结果：

```txt
app.js
```

还可以去掉扩展名：

```js
path.basename('D:\\demo\\src\\app.js', '.js');
```

结果：

```txt
app
```

## 9.5 `path.extname()`

作用：

- 获取扩展名

```js
path.extname('index.html'); // .html
path.extname('app.js');     // .js
path.extname('README');     // ''
```

## 9.6 `path.parse()`

作用：

- 把一个完整路径拆成对象

```js
path.parse('D:\\demo\\src\\app.js');
```

返回结果大致是：

```js
{
  root: 'D:\\',
  dir: 'D:\\demo\\src',
  base: 'app.js',
  ext: '.js',
  name: 'app'
}
```

适合在你需要一次性拿多个部分时使用。

## 9.7 `path.format()`

作用：

- 把 `parse()` 得到的对象重新拼成路径

```js
path.format({
  dir: 'D:\\demo\\src',
  name: 'app',
  ext: '.js'
});
```

## 9.8 `path.isAbsolute()`

作用：

- 判断一个路径是不是绝对路径

```js
path.isAbsolute('D:\\demo'); // true
path.isAbsolute('./demo');   // false
```

## 9.9 `path.normalize()`

作用：

- 规范化路径
- 清理多余分隔符和 `..`

```js
path.normalize('a//b/../c');
```

结果会变成更标准的路径形式。

## 9.10 `path.relative()`

作用：

- 计算从一个路径到另一个路径的相对路径

```js
path.relative('D:\\demo', 'D:\\demo\\src\\app.js');
```

结果：

```txt
src\app.js
```

## 9.11 `path.sep` 和 `path.delimiter`

### `path.sep`

- 当前系统的路径分隔符
- Windows 通常是 `\`
- Linux / macOS 通常是 `/`

### `path.delimiter`

- 环境变量中多个路径的分隔符
- Windows 通常是 `;`
- Linux / macOS 通常是 `:`

例如 `PATH` 环境变量里就会用到它。

## 10. 跨平台问题

不同系统路径格式不一样：

- Windows 常见：`D:\project\src\app.js`
- Linux / macOS 常见：`/usr/local/project/src/app.js`

所以不建议自己手写拼接：

```js
const bad = __dirname + '/public/' + fileName;
```

更建议：

```js
const good = path.join(__dirname, 'public', fileName);
```

这样跨平台更稳。

## 11. 常见使用场景

### 11.1 读取静态文件

```js
const filePath = path.join(__dirname, 'public', 'index.html');
```

### 11.2 拼上传目录

```js
const uploadDir = path.resolve('uploads');
```

### 11.3 获取文件扩展名

```js
const ext = path.extname(file.originalname);
```

### 11.4 从完整路径提取文件名

```js
const filename = path.basename(filePath);
```

## 12. 容易混淆的点

### 12.1 `join()` 不等于 `resolve()`

- `join()` 负责拼接
- `resolve()` 负责解析成最终路径，常常是绝对路径

### 12.2 相对路径不一定相对于当前文件

很多初学者会以为：

```js
fs.readFileSync('./data.txt');
```

这里的 `./` 是相对于当前 JS 文件。

其实很多时候它相对于的是 `process.cwd()`，所以更稳妥的做法是：

```js
const filePath = path.join(__dirname, 'data.txt');
```

### 12.3 URL 路径不是 `path` 模块处理的重点

`path` 模块主要处理的是本地文件系统路径，不是网页 URL。

例如：

- `/api/user` 是 URL 路径
- `D:\demo\api\user.json` 是文件路径

### 12.4 ESM 中不能直接用 `__dirname`

只要项目是 ES Module，就要记得自己通过 `import.meta.url` 转出来。

## 13. 一套高频记忆法

可以直接这样记：

- `join`：拼起来
- `resolve`：算到底
- `dirname`：取目录
- `basename`：取文件名
- `extname`：取后缀
- `parse`：拆开
- `format`：装回去
- `relative`：算相对路径

## 14. 最终总结

### 路径的本质

路径就是“定位资源位置”的描述方式，但要先分清它是在描述：

- 本地磁盘资源
- 还是网络地址资源

### 一句话总结

Node.js 里的 `path` 模块主要用于处理文件系统路径，核心任务是安全拼接、解析和标准化路径；而网页里的 URL 路径属于网络地址的一部分，不能和磁盘路径混为一谈。

### 适合背诵的简版

在 Node.js 中，路径分为相对路径和绝对路径。相对路径依赖参照位置，绝对路径更明确。`path` 模块用于拼接、解析和处理文件路径，常用方法有 `join`、`resolve`、`dirname`、`basename`、`extname`、`parse` 等。开发时要特别区分 `process.cwd()`、`__dirname`、文件路径和 URL 路径。
