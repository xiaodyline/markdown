# 1. IndexedDB

`IndexedDB` 是浏览器提供的本地数据库，适合存储大量结构化数据。它和 `localStorage` 不同，`localStorage` 只能同步存字符串，容量较小；`IndexedDB` 是异步的，可以存对象、数组、Blob、文件等复杂数据，并且支持索引和事务，适合离线缓存、草稿保存、聊天记录、本地文件缓存等场景。



# React Router 是怎么实现的？

React Router 本质上做了三件事：

```txt
1. 监听 URL 变化
2. 根据 URL 匹配对应的组件
3. URL 变化后触发 React 重新渲染
```

------

# 1. React Router 的核心思想

比如你配置了这些路由：

```tsx
const routes = [
  {
    path: '/',
    element: <Home />
  },
  {
    path: '/login',
    element: <Login />
  },
  {
    path: '/user',
    element: <User />
  }
]
```

当浏览器地址是：

```txt
/login
```

React Router 会匹配到：

```tsx
{
  path: '/login',
  element: <Login />
}
```

然后渲染：

```tsx
<Login />
```

所以它的本质是：

```txt
当前 URL path  =>  匹配路由表  =>  渲染对应组件
```

------

# 2. 两种常见路由模式

React Router 常见有两种模式：

| 模式          | URL 示例                  | 底层 API                         |
| ------------- | ------------------------- | -------------------------------- |
| HashRouter    | `http://site.com/#/login` | `hashchange`                     |
| BrowserRouter | `http://site.com/login`   | `history.pushState` / `popstate` |

------

# 3. HashRouter 原理

HashRouter 使用的是 URL 中的 `#`。

例如：

```txt
http://localhost:3000/#/login
```

其中：

```txt
#/login
```

就是 hash 部分。

浏览器 hash 变化时，不会刷新页面。

比如：

```js
window.location.hash = '#/login'
```

页面不会重新请求服务器，只会触发：

```js
window.addEventListener('hashchange', () => {
  console.log(location.hash)
})
```

React Router 就可以监听 `hashchange`，然后更新组件。

------

## 3.1 简化实现 HashRouter

```tsx
import React, { useEffect, useState } from 'react'

function HashRouter({ routes }) {
  const [path, setPath] = useState(() => {
    return window.location.hash.slice(1) || '/'
  })

  useEffect(() => {
    const handleHashChange = () => {
      setPath(window.location.hash.slice(1) || '/')
    }

    window.addEventListener('hashchange', handleHashChange)

    return () => {
      window.removeEventListener('hashchange', handleHashChange)
    }
  }, [])

  const route = routes.find(item => item.path === path)

  return route ? route.element : <div>404</div>
}
```

使用：

```tsx
function App() {
  const routes = [
    { path: '/', element: <Home /> },
    { path: '/login', element: <Login /> },
    { path: '/user', element: <User /> }
  ]

  return <HashRouter routes={routes} />
}
```

跳转：

```js
window.location.hash = '#/login'
```

------

# 4. BrowserRouter 原理

BrowserRouter 使用的是 History API。

例如：

```txt
http://localhost:3000/login
```

它没有 `#`，URL 更正常。

跳转时使用：

```js
history.pushState({}, '', '/login')
```

这个方法会修改浏览器地址栏，但不会刷新页面。

但是注意：

```js
history.pushState({}, '', '/login')
```

本身不会自动触发 `popstate` 事件。

所以 React Router 内部在调用 `pushState` 之后，还会主动更新自己的状态，让 React 重新渲染。

------

## 4.1 BrowserRouter 监听什么？

主要监听：

```js
window.addEventListener('popstate', callback)
```

`popstate` 主要在用户点击浏览器前进、后退时触发。

例如：

```txt
用户点击后退按钮
  ↓
URL 变化
  ↓
触发 popstate
  ↓
React Router 更新 location
  ↓
重新匹配路由并渲染组件
```

------

## 4.2 简化实现 BrowserRouter

```tsx
import React, { useEffect, useState } from 'react'

function BrowserRouter({ routes }) {
  const [path, setPath] = useState(() => {
    return window.location.pathname
  })

  useEffect(() => {
    const handlePopState = () => {
      setPath(window.location.pathname)
    }

    window.addEventListener('popstate', handlePopState)

    return () => {
      window.removeEventListener('popstate', handlePopState)
    }
  }, [])

  function navigate(to) {
    window.history.pushState({}, '', to)
    setPath(window.location.pathname)
  }

  const route = routes.find(item => item.path === path)

  return (
    <RouterContext.Provider value={{ path, navigate }}>
      {route ? route.element : <div>404</div>}
    </RouterContext.Provider>
  )
}
```

核心就是：

```txt
pushState 改 URL
setPath 改 React 状态
状态变化触发重新渲染
重新匹配组件
```

------

# 5. Link 是怎么实现的？

平时写：

```tsx
<Link to="/login">登录</Link>
```

它不是普通的：

```html
<a href="/login">登录</a>
```

如果直接用普通 `a` 标签，浏览器会刷新页面。

React Router 的 `Link` 大概是这样实现的：

```tsx
function Link({ to, children }) {
  const { navigate } = useContext(RouterContext)

  function handleClick(e) {
    e.preventDefault()
    navigate(to)
  }

  return (
    <a href={to} onClick={handleClick}>
      {children}
    </a>
  )
}
```

关键点：

```js
e.preventDefault()
```

阻止浏览器默认跳转。

然后调用：

```js
navigate(to)
```

由 React Router 接管跳转。

------

# 6. useNavigate 是怎么实现的？

平时写：

```tsx
const navigate = useNavigate()

navigate('/login')
```

它底层其实是从 Router 的上下文里取出 `navigate` 方法。

简化版：

```tsx
const RouterContext = React.createContext(null)

function useNavigate() {
  const context = useContext(RouterContext)

  if (!context) {
    throw new Error('useNavigate must be used inside Router')
  }

  return context.navigate
}
```

所以为什么 `useNavigate` 只能在组件里用？

因为它内部用了：

```tsx
useContext()
```

而 Hook 只能在 React 函数组件或自定义 Hook 中使用。

------

# 7. Routes 和 Route 是怎么实现的？

平时写法：

```tsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/login" element={<Login />} />
</Routes>
```

React Router 会把这些 `Route` 配置解析成路由表。

大概等价于：

```tsx
[
  { path: '/', element: <Home /> },
  { path: '/login', element: <Login /> }
]
```

然后根据当前路径匹配。

简化逻辑：

```tsx
function Routes({ routes }) {
  const { path } = useContext(RouterContext)

  const route = routes.find(item => item.path === path)

  return route ? route.element : <div>404</div>
}
```

------

# 8. 动态路由怎么实现？

比如：

```tsx
<Route path="/user/:id" element={<User />} />
```

访问：

```txt
/user/100
```

React Router 会把：

```txt
/user/:id
```

和：

```txt
/user/100
```

进行匹配。

匹配成功后得到：

```js
{
  id: '100'
}
```

简化实现：

```js
function matchPath(routePath, currentPath) {
  const routeParts = routePath.split('/')
  const currentParts = currentPath.split('/')

  if (routeParts.length !== currentParts.length) {
    return null
  }

  const params = {}

  for (let i = 0; i < routeParts.length; i++) {
    const routePart = routeParts[i]
    const currentPart = currentParts[i]

    if (routePart.startsWith(':')) {
      const key = routePart.slice(1)
      params[key] = currentPart
    } else if (routePart !== currentPart) {
      return null
    }
  }

  return params
}

console.log(matchPath('/user/:id', '/user/100'))
```

输出：

```js
{
  id: '100'
}
```

`useParams()` 底层就是从当前匹配结果里取出这些参数。

------

# 9. 嵌套路由和 Outlet 是怎么实现的？

平时写：

```tsx
<Route path="/layout" element={<Layout />}>
  <Route path="home" element={<Home />} />
  <Route path="about" element={<About />} />
</Route>
```

`Layout` 组件中写：

```tsx
import { Outlet } from 'react-router-dom'

function Layout() {
  return (
    <div>
      <Header />
      <Outlet />
    </div>
  )
}
```

访问：

```txt
/layout/home
```

渲染结构是：

```tsx
<Layout>
  <Home />
</Layout>
```

但是 React 组件不能直接这样自动嵌套，所以 React Router 提供了：

```tsx
<Outlet />
```

`Outlet` 的作用是：

```txt
在父路由组件中，放置子路由组件的渲染位置
```

可以简单理解为：

```tsx
function Outlet() {
  return 当前匹配到的子路由组件
}
```

------

# 10. BrowserRouter 为什么需要后端配合？

HashRouter 不需要特殊配置，因为真正发给服务器的路径始终是：

```txt
/
```

例如：

```txt
http://localhost:3000/#/login
```

服务器看到的其实是：

```txt
/
```

但是 BrowserRouter 的路径是：

```txt
http://localhost:3000/login
```

如果你刷新页面，浏览器会真的向服务器请求：

```txt
/login
```

如果服务器没有这个路径，就会返回 404。

所以部署 React 单页应用时，需要让后端或 Nginx 把所有前端路由都返回 `index.html`。

Nginx 常见配置：

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

意思是：

```txt
先找真实文件
找不到就返回 index.html
让 React Router 接管路由
```

------

# 11. 总结

React Router 的实现可以概括为：

```txt
1. Router 监听 URL 变化
2. 把当前路径存到 React 状态中
3. Routes 根据当前路径匹配 Route
4. 匹配成功后渲染对应组件
5. Link 阻止 a 标签默认跳转，改用 history.pushState
6. useNavigate 从上下文中拿到 navigate 方法
7. URL 变化后更新状态，触发 React 重新渲染
```

面试回答可以这样说：

> React Router 是前端路由库，本质是监听浏览器 URL 的变化，然后根据当前路径匹配路由配置，渲染对应的 React 组件。HashRouter 基于 `hashchange` 事件实现，路径在 `#` 后面，变化时不会刷新页面；BrowserRouter 基于 History API 实现，主要使用 `history.pushState` 修改地址栏，并监听 `popstate` 处理浏览器前进后退。React Router 内部会把当前 location 存在状态中，URL 变化时更新状态，触发组件重新渲染。`Link` 组件会阻止 `a` 标签默认跳转，改用内部的 `navigate` 方法；`useNavigate` 则是通过 Context 获取导航方法。BrowserRouter 部署时需要服务器把未知路径都回退到 `index.html`，否则刷新页面会 404。

React Router 常见有两种模式：`HashRouter` 和 `BrowserRouter`。

`HashRouter` 基于 URL 的 hash 实现，比如 `/#/login`。hash 改变不会导致页面刷新，也不会向服务器重新请求页面，但会触发 `hashchange` 事件。React Router 监听这个事件，读取 `window.location.hash`，得到当前路径，再根据路由表匹配对应组件并触发 React 重新渲染。它的优点是部署简单，刷新不会 404；缺点是 URL 中带 `#`，不够美观。

`BrowserRouter` 基于 HTML5 History API 实现，比如 `/login`。它通过 `history.pushState` 修改地址栏路径，同时阻止 `a` 标签默认跳转，所以页面不会刷新。React Router 内部会更新当前 location 状态，然后重新匹配路由并渲染组件。对于浏览器前进后退，它通过监听 `popstate` 事件来感知路径变化。它的优点是 URL 更美观，更接近真实路径；缺点是部署时需要后端或 Nginx 配置回退到 `index.html`，否则刷新 `/login` 这类路径时服务器可能返回 404。