# React 中常见路由跳转方式笔记

## 1. 先看怎么选

### 用户主动点链接

用 `Link` 或 `NavLink`。
它们适合导航栏、菜单、列表跳转这类“用户点击后进入别的路由”的场景。`Link` 是对 `<a>` 的客户端路由封装，`NavLink` 额外适合需要“当前激活态”的导航项。 ([React Router](https://reactrouter.com/api/components/Link?utm_source=chatgpt.com))

### 事件触发跳转

用 `useNavigate()`。
它适合登录成功后跳转、按钮点击跳转、定时跳转、返回上一页这类“在函数里控制跳转”的场景。官方也说明，它返回一个 `navigate` 函数，可用于 `navigate(to, options)` 或 `navigate(delta)`。 ([React Router](https://reactrouter.com/api/hooks/useNavigate?utm_source=chatgpt.com))

### 渲染时根据条件拦截

用 `<Navigate />`。
它适合权限守卫、未登录跳登录页、已登录访问登录页时跳首页这类“组件一渲染就判断是否该跳走”的场景。React Router 将它归入组件式导航能力。 ([React Router](https://reactrouter.com/?utm_source=chatgpt.com))

### loader / action 阶段跳转

用 `redirect()`。
它适合进入页面前做登录校验、提交表单成功后跳转、根据数据结果直接改去其他页面。官方明确说明：对于 action/loader 中的跳转，通常比在组件里用 `useNavigate()` 更合适。`redirect()` 本质上会返回一个带 `Location` 头的重定向响应，默认状态码是 `302`。 ([React Router](https://reactrouter.com/api/hooks/useNavigate?utm_source=chatgpt.com))

### 跳外部网站或强制整页刷新

用 `window.location.href` 或 `window.location.replace()`。
它们不属于 React Router，而是浏览器原生跳转方式。设置 `href` 会导航到新地址；`replace()` 也会导航，但不会把当前页保留在会话历史里，用户不能通过返回按钮回到当前页。 ([MDN веб-文档](https://developer.mozilla.org/ja/docs/Web/API/Location/href?utm_source=chatgpt.com))

------

## 2. `Link`

## 什么情况用

当用户通过点击进入另一个页面，而且你希望保持单页应用的客户端路由体验时，用 `Link`。最典型的就是导航菜单、卡片列表、文章标题、详情入口。 ([React Router](https://reactrouter.com/api/components/Link?utm_source=chatgpt.com))

## 作用是什么

它是一个“增强版的 `<a>` 标签”，用于客户端路由跳转，避免不必要的整页刷新。 ([React Router](https://reactrouter.com/api/components/Link?utm_source=chatgpt.com))

## 怎么用

```tsx
import { Link } from 'react-router-dom';

export default function Home() {
  return (
    <div>
      <Link to="/login">去登录页</Link>
    </div>
  );
}
```

也可以带查询参数和 hash：

```tsx
<Link
  to={{
    pathname: '/list',
    search: '?page=2',
    hash: '#top',
  }}
>
  去列表页
</Link>
```

`Link` 的 `to` 可以是字符串，也可以是一个位置对象。 ([React Router](https://reactrouter.com/api/components/Link?utm_source=chatgpt.com))

## 注意事项

- 它更适合“用户主动点击”的跳转，不适合登录成功后的业务逻辑跳转。
- 如果你只是想在代码里触发跳转，不要硬塞到 `Link`，应改用 `useNavigate()`。
- 站内路由优先用 `Link`，外部网址一般直接用普通 `<a>` 或 `window.location`。

------

## 3. `NavLink`

## 什么情况用

当你不仅要跳转，还要显示“当前页面高亮”时，用 `NavLink`。最常见的是侧边栏、顶部导航栏、标签页切换。 ([React Router](https://reactrouter.com/start/declarative/navigating?utm_source=chatgpt.com))

## 作用是什么

它和 `Link` 一样负责跳转，但多了“激活状态”的判断能力。官方文档明确指出，`NavLink` 适合需要 active state 的导航链接。 ([React Router](https://reactrouter.com/start/declarative/navigating?utm_source=chatgpt.com))

## 怎么用

```tsx
import { NavLink } from 'react-router-dom';

export default function Menu() {
  return (
    <nav>
      <NavLink to="/" end>首页</NavLink>
      <NavLink to="/article">文章</NavLink>
      <NavLink to="/user">个人中心</NavLink>
    </nav>
  );
}
```

常见配合样式使用：

```tsx
<NavLink
  to="/user"
  className={({ isActive }) => (isActive ? 'active' : '')}
>
  个人中心
</NavLink>
```

## 注意事项

- 它主要价值是“高亮当前路由”，如果你不需要激活态，直接用 `Link` 更简单。
- 根路径高亮时通常会配合 `end` 使用，避免 `/` 把所有子路径都匹配亮了。官方示例中也这样使用。 ([React Router](https://reactrouter.com/start/declarative/navigating?utm_source=chatgpt.com))

------

## 4. `useNavigate()`

## 什么情况用

当跳转发生在函数逻辑里时，用 `useNavigate()`。例如：

- 登录成功后跳首页
- 点击按钮后跳详情页
- 删除成功后跳列表页
- 点击返回按钮后 `navigate(-1)` 返回上一页 ([React Router](https://reactrouter.com/api/hooks/useNavigate?utm_source=chatgpt.com))

## 作用是什么

它返回一个 `navigate` 函数，让你在组件中以编程方式控制导航。官方文档给出的签名是 `navigate(to, options?)` 和 `navigate(delta)`。 ([React Router](https://reactrouter.com/api/hooks/useNavigate?utm_source=chatgpt.com))

## 怎么用

```tsx
import { useNavigate } from 'react-router-dom';

export default function Login() {
  const navigate = useNavigate();

  const handleLogin = () => {
    // 假设登录成功
    navigate('/home');
  };

  return <button onClick={handleLogin}>登录</button>;
}
```

返回上一页：

```tsx
const navigate = useNavigate();

<button onClick={() => navigate(-1)}>返回</button>
```

替换当前历史记录：

```tsx
navigate('/home', { replace: true });
```

## 注意事项

- 它是 Hook，只能在 React 函数组件或自定义 Hook 顶层调用，不能写在普通函数外部。
- 如果跳转逻辑本质上是 loader/action 的结果，官方更推荐用 `redirect()`，而不是等组件运行后再 `useNavigate()`。 ([React Router](https://reactrouter.com/api/hooks/useNavigate?utm_source=chatgpt.com))
- 登录页、支付回调页这类“不希望用户返回当前页”的场景，通常配合 `{ replace: true }`。

------

## 5. `<Navigate />`

## 什么情况用

当组件渲染时就能判断“该不该继续显示当前页面”时，用 `<Navigate />`。
典型场景：

- 未登录访问受保护页面，跳到登录页
- 已登录访问登录页，跳到首页
- 某些页面根据当前状态直接转走

## 作用是什么

它是声明式重定向组件。你不需要手动调用函数，而是在 JSX 里直接返回它，让 React Router 完成跳转。 ([React Router](https://reactrouter.com/?utm_source=chatgpt.com))

## 怎么用

```tsx
import { Navigate } from 'react-router-dom';
import type { ReactNode } from 'react';

type Props = {
  children: ReactNode;
};

export default function AuthRoute({ children }: Props) {
  const token = localStorage.getItem('token');

  if (!token) {
    return <Navigate to="/login" replace />;
  }

  return <>{children}</>;
}
```

## 注意事项

- 它更适合“渲染时就能决定”的跳转，不适合按钮点击后的跳转。
- 如果判断依赖的是路由加载数据，且你已使用 data router，通常把逻辑前移到 `loader` 里用 `redirect()` 会更干净。
- 登录拦截场景里通常加 `replace`，避免用户返回到受保护页再被重新拦截。

------

## 6. `redirect()`

## 什么情况用

当你已经在使用 React Router 的 data router，并且跳转是“数据处理的结果”时，用 `redirect()`。
典型场景：

- 进入页面前检查 token，没有就去 `/login`
- 表单提交成功后跳转到详情页
- 查询不到资源时跳 `/404`
- 根据 loader / action 的结果决定目标页 ([React Router](https://reactrouter.com/api/utils/redirect?utm_source=chatgpt.com))

## 作用是什么

它会返回一个重定向响应，设置 `Location` 头，默认状态码是 `302`。官方明确说：在 action/loader 函数里，这通常比 `useNavigate()` 更合适。 ([React Router](https://reactrouter.com/api/utils/redirect?utm_source=chatgpt.com))

## 怎么用

### 在 loader 中用

```tsx
import { createBrowserRouter, redirect } from 'react-router-dom';
import Home from './Home';
import Login from './Login';

const router = createBrowserRouter([
  {
    path: '/home',
    loader: async () => {
      const token = localStorage.getItem('token');
      if (!token) {
        return redirect('/login');
      }
      return null;
    },
    element: <Home />,
  },
  {
    path: '/login',
    element: <Login />,
  },
]);
```

### 在 action 中用

```tsx
import { redirect } from 'react-router-dom';

export async function action({ request }: { request: Request }) {
  const formData = await request.formData();
  const username = formData.get('username');
  const password = formData.get('password');

  if (username === 'admin' && password === '123456') {
    return redirect('/home');
  }

  return { error: '用户名或密码错误' };
}
```

## 注意事项

- 它依赖 data router 体系，通常要用 `createBrowserRouter` 这类路由配置，而不是只用传统的 `<BrowserRouter><Routes /></BrowserRouter>` 写法。`loader` 和 `action` 本身就是 data router 的能力。 ([React Router](https://reactrouter.com/start/framework/data-loading?utm_source=chatgpt.com))
- 它接受绝对 URL，也可以跳到外部域名；如果跳转地址来自用户输入，必须做校验，避免开放重定向风险。官方文档明确提醒了这一点。 ([React Router](https://reactrouter.com/api/utils/redirect?utm_source=chatgpt.com))
- 当你已经在 loader/action 中处理业务时，优先考虑 `redirect()`，不要把同样的逻辑拖到组件里二次跳转。 ([React Router](https://reactrouter.com/api/hooks/useNavigate?utm_source=chatgpt.com))

------

## 7. `window.location.href`

## 什么情况用

当你需要跳到外部网站，或者明确希望浏览器执行一次完整页面跳转时，用它。
例如：

- 跳到公司官网
- 跳第三方支付页
- 跳第三方 OAuth 登录页

## 作用是什么

给 `location.href` 赋值会导航到新的 URL。MDN 说明，设置 `href` 的值会触发导航。 ([MDN веб-文档](https://developer.mozilla.org/ja/docs/Web/API/Location/href?utm_source=chatgpt.com))

## 怎么用

```ts
window.location.href = 'https://example.com';
```

## 注意事项

- 这是浏览器原生导航，不是 React Router 内部导航。
- 一般会导致整页刷新。
- 如果是站内路由跳转，通常优先用 React Router 的方式，避免失去单页应用体验。
- 如果你真正想做“重定向替换”，不要和 `replace()` 混淆。MDN 明确指出，如果想要 redirection 的替换语义，应考虑 `location.replace()`。 ([MDN веб-文档](https://developer.mozilla.org/ja/docs/Web/API/Location/href?utm_source=chatgpt.com))

------

## 8. `window.location.replace()`

## 什么情况用

当你需要整页跳转，并且不希望用户通过返回按钮回到当前页时，用它。
典型场景：

- 从中间跳转页进入正式页
- 第三方登录回调后替换回业务页
- 某些一次性过渡页面不希望留在历史记录中

## 作用是什么

它会导航到新地址，但当前页面不会保存到会话历史中。MDN 明确说明，这和普通赋值 `href` 的区别在于：`replace()` 后用户不能通过返回按钮回到当前页。 ([MDN веб-文档](https://developer.mozilla.org/en-US/docs/Web/API/Location/replace?utm_source=chatgpt.com))

## 怎么用

```ts
window.location.replace('https://example.com');
```

## 注意事项

- 这同样不是 React Router 的客户端导航，而是浏览器原生导航。
- 一般会发生整页刷新。
- 因为会替换历史记录，所以要谨慎使用，避免影响用户的返回路径。
- 如果只是 React 应用内的普通站内跳转，大多数情况下不该优先用它。

------

## 9. 最后给一个实用选择表

## 导航菜单、文章链接、列表入口

优先用 `Link`。
如果要高亮当前项，用 `NavLink`。 ([React Router](https://reactrouter.com/api/components/Link?utm_source=chatgpt.com))

## 登录成功、按钮点击、返回上一页

优先用 `useNavigate()`。 ([React Router](https://reactrouter.com/api/hooks/useNavigate?utm_source=chatgpt.com))

## 权限守卫、未登录拦截

组件层面优先用 `<Navigate />`。
如果已经是 data router，路由进入前拦截更适合 `redirect()`。 ([React Router](https://reactrouter.com/?utm_source=chatgpt.com))

## loader / action 的业务跳转

优先用 `redirect()`。 ([React Router](https://reactrouter.com/api/hooks/useNavigate?utm_source=chatgpt.com))

## 跳外部地址、强制整页跳转

用 `window.location.href` 或 `window.location.replace()`。
如果不想保留当前页历史，选 `replace()`。 ([MDN веб-文档](https://developer.mozilla.org/ja/docs/Web/API/Location/href?utm_source=chatgpt.com))

------

## 10. 一句话总结

- **点链接**：`Link / NavLink`
- **函数里跳**：`useNavigate()`
- **渲染时拦截**：`<Navigate />`
- **loader / action 里跳**：`redirect()`
- **跳外部或整页刷新**：`window.location.href / replace()` ([React Router](https://reactrouter.com/start/declarative/navigating?utm_source=chatgpt.com))

如果你要，我下一条可以继续把这份笔记整理成更适合背诵的“面试版精简稿”。