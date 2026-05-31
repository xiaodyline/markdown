# JWT 是什么？

JWT 全称是：

```txt
JSON Web Token
```

它是一种 **登录身份凭证**。

用户登录成功后，后端生成一个 token 返回给前端，前端后续请求接口时带上这个 token，后端通过校验 token 判断用户是谁、有没有权限。

------

# 1. JWT 解决什么问题？

HTTP 本身是无状态的。

也就是说，用户第一次登录成功后，第二次请求接口时，服务器默认不知道这个请求是谁发来的。

所以需要一种方式告诉后端：

```txt
这个请求来自已经登录的用户
```

JWT 就是常见方案之一。

------

# 2. JWT 登录流程

```txt
用户输入账号密码
  ↓
前端请求登录接口
  ↓
后端校验账号密码
  ↓
校验成功，生成 JWT
  ↓
后端把 JWT 返回给前端
  ↓
前端保存 JWT
  ↓
后续请求接口时带上 JWT
  ↓
后端校验 JWT
  ↓
校验通过，允许访问接口
```

------

# 3. 前端怎么带 JWT？

通常放在请求头里：

```http
Authorization: Bearer token字符串
```

例如 axios 中：

```js
axios.interceptors.request.use(config => {
  const token = localStorage.getItem('token')

  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }

  return config
})
```

后端收到请求后，会从请求头里取：

```txt
Authorization
```

然后解析里面的 token。

------

# 4. JWT 长什么样？

JWT 通常是三段字符串，用 `.` 分隔：

```txt
xxxxx.yyyyy.zzzzz
```

结构是：

```txt
Header.Payload.Signature
```

也就是：

```txt
头部.载荷.签名
```

------

# 5. JWT 三部分分别是什么？

## 5.1 Header：头部

Header 描述 token 类型和签名算法。

例如：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

意思是：

```txt
alg：签名算法
typ：token 类型
```

------

## 5.2 Payload：载荷

Payload 存储用户相关信息。

例如：

```json
{
  "userId": 1001,
  "username": "zhangsan",
  "role": "admin",
  "exp": 1710000000
}
```

常见字段：

| 字段       | 含义     |
| ---------- | -------- |
| `userId`   | 用户 ID  |
| `username` | 用户名   |
| `role`     | 用户角色 |
| `iat`      | 签发时间 |
| `exp`      | 过期时间 |

注意：Payload 只是 Base64Url 编码，不是加密。

所以不要把密码、身份证号、手机号等敏感信息放进 JWT。

------

## 5.3 Signature：签名

Signature 用来防止 token 被篡改。

生成逻辑可以理解为：

```txt
Signature = 加密算法(Header + Payload + 后端密钥)
```

如果攻击者修改了 Payload，比如把：

```json
{
  "role": "user"
}
```

改成：

```json
{
  "role": "admin"
}
```

那么签名就对不上，后端校验会失败。

------

# 6. JWT 是怎么校验的？

后端收到 token 后：

```txt
1. 拆分 Header、Payload、Signature
2. 用同样的算法和密钥重新生成签名
3. 比较新签名和 token 里的 Signature 是否一致
4. 判断 token 是否过期
5. 校验通过，认为用户身份可信
```

关键点：

```txt
JWT 的可信性来自签名，不是来自 Payload 本身。
```

------

# 7. JWT 和 Session 的区别

| 对比项           | Session                 | JWT                              |
| ---------------- | ----------------------- | -------------------------------- |
| 登录状态存在哪里 | 服务端                  | 客户端保存 token                 |
| 服务端是否存状态 | 需要                    | 通常不需要                       |
| 扩展性           | 集群下要共享 Session    | 更适合分布式                     |
| 退出登录         | 服务端删除 Session 即可 | JWT 本身较难主动失效             |
| 安全控制         | 服务端控制强            | 依赖过期时间、黑名单等           |
| 常见存储         | Cookie 中存 sessionId   | localStorage / Cookie 中存 token |

Session 的流程是：

```txt
用户登录
  ↓
服务端创建 Session
  ↓
返回 sessionId
  ↓
浏览器保存 sessionId
  ↓
后续请求带 sessionId
  ↓
服务端根据 sessionId 查 Session
```

JWT 的流程是：

```txt
用户登录
  ↓
服务端签发 JWT
  ↓
前端保存 JWT
  ↓
后续请求带 JWT
  ↓
服务端直接验证 JWT
```

------

# 8. JWT 存在哪里？

常见有两种方式。

## 8.1 localStorage

```js
localStorage.setItem('token', token)
```

优点：

```txt
使用简单
前端控制方便
刷新页面不会丢失
```

缺点：

```txt
容易受到 XSS 影响
恶意脚本可以读取 localStorage 中的 token
```

------

## 8.2 Cookie

后端通过响应头设置：

```http
Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Lax
```

优点：

```txt
配合 HttpOnly 后，JS 不能读取 token
更能降低 XSS 窃取 token 的风险
```

缺点：

```txt
Cookie 会自动携带，要注意 CSRF
需要配置 SameSite、CSRF Token 等防护
```

常见安全配置：

```txt
HttpOnly：禁止 JS 读取 Cookie
Secure：只允许 HTTPS 传输
SameSite：限制跨站请求携带 Cookie
```

------

# 9. JWT 有什么优点？

```txt
1. 无状态，服务端不需要保存登录状态
2. 适合前后端分离
3. 适合分布式系统和微服务
4. 可以在 token 中携带少量用户信息
5. 后端只需要验证签名和过期时间
```

------

# 10. JWT 有什么缺点？

```txt
1. token 一旦签发，在过期前默认一直有效
2. 不适合存敏感信息
3. token 太大会增加请求体积
4. 如果被盗，攻击者可以冒用身份
5. 主动退出登录、强制下线比较麻烦
```

尤其要注意：

```txt
JWT 不是加密，它只是签名。
Payload 内容可以被解码查看。
```

------

# 11. token 过期怎么办？

常见做法是 **双 token 机制**：

```txt
accessToken：短期 token，用于访问接口
refreshToken：长期 token，用于刷新 accessToken
```

流程：

```txt
请求接口
  ↓
accessToken 有效
  ↓
正常返回数据
```

如果 accessToken 过期：

```txt
接口返回 401
  ↓
前端用 refreshToken 请求刷新接口
  ↓
后端校验 refreshToken
  ↓
返回新的 accessToken
  ↓
前端重试刚才失败的请求
```

如果 refreshToken 也过期：

```txt
清除登录信息
跳转登录页
```

------

# 12. axios 中 JWT 的常见封装

```js
import axios from 'axios'

const request = axios.create({
  baseURL: '/api',
  timeout: 5000
})

request.interceptors.request.use(config => {
  const token = localStorage.getItem('accessToken')

  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }

  return config
})

request.interceptors.response.use(
  response => response.data,
  error => {
    if (error.response?.status === 401) {
      localStorage.removeItem('accessToken')
      window.location.href = '/login'
    }

    return Promise.reject(error)
  }
)

export default request
```

React 项目中更推荐用 `navigate('/login')`，不要直接 `window.location.href`，因为后者会刷新整个页面。

------

# 13. 面试回答

JWT 是一种基于 JSON 的登录令牌，主要用于前后端分离项目中的身份认证。它由三部分组成：Header、Payload、Signature。Header 保存 token 类型和签名算法，Payload 保存用户 ID、角色、过期时间等信息，Signature 是后端用密钥对前两部分生成的签名，用来防止 token 被篡改。

用户登录成功后，后端生成 JWT 返回给前端。前端保存 token，后续请求接口时通过 `Authorization: Bearer xxx` 携带。后端收到请求后验证签名和过期时间，验证通过就认为用户已登录。

JWT 的优点是服务端无状态，适合分布式和前后端分离项目；缺点是 token 签发后在过期前默认有效，主动失效比较麻烦，而且 Payload 不是加密的，不能存敏感信息。实际项目中通常会使用 accessToken 和 refreshToken 机制，accessToken 负责访问接口，refreshToken 负责刷新登录态。





# 14.双Token

双 token 是为了解决 **安全性和用户体验之间的矛盾**。如果只用一个 token，设置得太长，被盗后风险很大；设置得太短，用户又需要频繁登录。双 token 把职责拆开：`accessToken` 有效期短，用来访问业务接口；`refreshToken` 有效期长，用来刷新 accessToken。

当 accessToken 过期时，前端用 refreshToken 请求刷新接口，拿到新的 accessToken 后重试原请求。这样既能让 accessToken 保持短有效期，降低被盗后的风险，又能让用户保持登录状态，不需要频繁重新登录。实际项目中 refreshToken 通常放在 HttpOnly Cookie 或服务端可控存储中，便于退出登录、强制下线和安全控制。 