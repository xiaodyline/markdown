# 1. 组件的生命周期

- **组件生命周期**：指组件从创建、显示到页面、数据变化重新渲染，再到从页面中移除的整个过程。

- **核心阶段只有三个**：

  ```text
  挂载 → 更新 → 卸载
  ```

- **挂载阶段 Mount**：组件第一次被渲染到页面上。

  常见场景：

  ```text
  发送初始化请求
  读取本地缓存
  初始化定时器
  绑定事件监听
  ```

- **更新阶段 Update**：组件已经在页面上，但因为 `state` 或 `props` 变化，组件重新渲染。

  常见触发原因：

  ```text
  state 改变
  props 改变
  父组件重新渲染
  ```

- **卸载阶段 Unmount**：组件从页面中被移除。

  常见清理操作：

  ```text
  清除定时器
  解绑事件监听
  取消网络请求
  关闭 WebSocket
  清理订阅
  ```

- **类组件中的生命周期方法**：

  ```text
  componentDidMount      挂载后执行
  componentDidUpdate     更新后执行
  componentWillUnmount   卸载前执行
  ```

- **函数组件中没有传统生命周期方法**，主要通过 `useEffect` 处理生命周期相关逻辑。

- **useEffect 和生命周期的对应关系**：

  ```jsx
  useEffect(() => {
    // 挂载后执行
      
  }, [])
  ```

  类似：

  ```text
  componentDidMount
  ```

  ```jsx
  useEffect(() => {
    // count 变化后执行
  }, [count])
  ```

  类似：

  ```text
  componentDidUpdate
  ```

  ```jsx
  useEffect(() => {
    return () => {
      // 组件卸载时执行
    }
  }, [])
  ```

  类似：

  ```text
  componentWillUnmount
  ```

- **一句话理解**：

  ```text
  生命周期就是 React 组件从出现、变化到消失的过程；
  Hook，尤其是 useEffect，是函数组件处理生命周期逻辑的主要方式。
  ```

- ### 面试回答：

  - 组件生命周期指的是组件从创建、渲染到页面、数据变化重新渲染，再到最后从页面中移除的整个过程。React 中主要可以分为三个阶段：挂载、更新和卸载。

    挂载阶段是组件第一次显示到页面上，通常适合做初始化请求、事件绑定、定时器初始化等操作。更新阶段是组件已经存在，但 state 或 props 发生变化，React 重新执行渲染逻辑。卸载阶段是组件从页面中移除，通常需要清除定时器、解绑事件、取消订阅或关闭连接，避免内存泄漏。

    在类组件中，常用 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 来处理生命周期逻辑。而在函数组件中，主要通过 `useEffect` 来处理这些生命周期相关的副作用逻辑。可以简单理解为，Hook 是函数组件处理状态和生命周期问题的方式。



# 2. Hook

## 2.1 Hook 是什么

Hook 是 React 提供给函数组件使用的一组特殊函数，通常以 `use` 开头，例如：

```jsx
useState
useEffect
useRef
useMemo
useCallback
useContext
```

Hook 的作用是让函数组件也能拥有状态管理、副作用处理、生命周期控制、DOM 引用、性能优化和逻辑复用等能力。

在 Hook 出现之前，如果组件需要状态或生命周期，通常要使用类组件；Hook 出现之后，大多数场景都可以使用函数组件完成。

------

## 2.2 为什么会出现 Hook

早期 React 主要使用类组件处理复杂逻辑：

```jsx
class Counter extends React.Component {
  state = {
    count: 0
  }

  componentDidMount() {
    console.log('组件挂载')
  }

  render() {
    return <div>{this.state.count}</div>
  }
}
```

类组件虽然可以处理状态和生命周期，但存在一些问题：

```text
1. 写法较重，需要 class、this、生命周期方法。
2. 同一类业务逻辑可能分散在多个生命周期函数中。
3. 组件之间复用状态逻辑比较麻烦。
4. this 指向问题容易增加理解成本。
```

Hook 的出现，主要是为了解决这些问题：

```text
1. 让函数组件可以保存状态。
2. 让函数组件可以处理生命周期和副作用。
3. 让状态逻辑更容易复用。
4. 简化组件写法，减少 class 和 this 的复杂度。
```

------

## 2.3 useState

### 作用

`useState` 用来给函数组件添加状态。

函数组件本质上是一个函数，每次组件重新渲染时，函数都会重新执行。如果只用普通变量保存数据，变量会在每次渲染时重新初始化，而且普通变量变化不会触发页面更新。

错误示例：

```jsx
function Counter() {
  let count = 0

  const add = () => {
    count++
    console.log(count)
  }

  return <button onClick={add}>{count}</button>
}
```

这里即使 `count++`，页面也不会自动更新。

正确写法：

```jsx
import { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  )
}
```

------

### 语法说明

```jsx
const [count, setCount] = useState(0)
```

含义：

```text
count：当前状态值
setCount：修改状态的方法
0：状态初始值
```

调用 `setCount` 后，React 会更新状态，并触发组件重新渲染。

------

### 新状态依赖旧状态时，推荐函数式写法

普通写法：

```jsx
setCount(count + 1)
```

函数式写法：

```jsx
setCount(prev => prev + 1)
```

如果连续更新状态，推荐使用函数式写法：

```jsx
setCount(prev => prev + 1)
setCount(prev => prev + 1)
setCount(prev => prev + 1)
```

这样最终会累加 3。

如果写成：

```jsx
setCount(count + 1)
setCount(count + 1)
setCount(count + 1)
```

在同一次事件中，可能只会基于同一个旧的 `count` 计算，最终只加 1。

------

### useState 面试回答

`useState` 是 React 中用于在函数组件中定义状态的 Hook。它返回一个状态值和一个修改状态的方法。状态变化后，React 会重新执行组件函数，并根据新的状态更新页面。

需要注意的是，不能直接修改 state，而应该通过 `setState` 方法更新。如果新状态依赖旧状态，推荐使用函数式更新，例如 `setCount(prev => prev + 1)`，这样可以避免连续更新时拿到旧值的问题。

------

## 2.4 useEffect

### 作用

`useEffect` 用来处理副作用。

组件渲染本身应该只负责根据数据生成 UI，但实际开发中经常需要做一些额外操作，例如：

```text
请求接口
操作 DOM
设置定时器
绑定事件
订阅数据
打印日志
关闭连接
```

这些操作不属于直接渲染 UI 的逻辑，所以称为副作用。

`useEffect` 就是专门用来处理副作用的 Hook。

------

### 基本写法

```jsx
import { useEffect } from 'react'

function Demo() {
  useEffect(() => {
    console.log('副作用执行')
  }, [])

  return <div>Demo</div>
}
```

------

### useEffect 的三种常见形式

#### 1. 不写依赖数组：每次渲染后都执行

```jsx
useEffect(() => {
  console.log('每次渲染后执行')
})
```

执行时机：

```text
组件挂载后执行
组件每次更新后也执行
```

这种写法不常用，因为容易导致重复执行。

------

#### 2. 空依赖数组：只在挂载后执行一次

```jsx
useEffect(() => {
  console.log('组件挂载后执行一次')
}, [])
```

类似类组件中的：

```text
componentDidMount
```

常见场景：

```text
初始化请求
初始化定时器
绑定事件监听
```

------

#### 3. 有依赖数组：依赖变化时执行

```jsx
useEffect(() => {
  console.log('keyword 变化了')
}, [keyword])
```

执行时机：

```text
组件挂载后执行一次
以后 keyword 每次变化时执行
```

常见场景：

```text
根据搜索关键字请求数据
根据 props 变化重新加载内容
监听某个 state 的变化
```

------

### 清理函数

`useEffect` 可以返回一个函数，这个函数叫清理函数。

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log('定时器执行')
  }, 1000)

  return () => {
    clearInterval(timer)
  }
}, [])
```

清理函数常用于：

```text
清除定时器
解绑事件监听
取消订阅
关闭 WebSocket
取消未完成请求
```

------

### useEffect 和生命周期的关系

| 生命周期场景 | 类组件                 | 函数组件                                   |
| ------------ | ---------------------- | ------------------------------------------ |
| 挂载后执行   | `componentDidMount`    | `useEffect(() => {}, [])`                  |
| 更新后执行   | `componentDidUpdate`   | `useEffect(() => {}, [依赖])`              |
| 卸载前清理   | `componentWillUnmount` | `useEffect(() => { return () => {} }, [])` |

注意：`useEffect(() => {}, [依赖])` 在组件第一次挂载后也会执行一次，不是只在依赖更新时执行。

------

### useEffect 面试回答

`useEffect` 是 React 中用于处理副作用的 Hook。副作用指的是组件渲染之外的操作，比如请求接口、绑定事件、设置定时器、订阅数据等。

`useEffect` 可以通过依赖数组控制执行时机。如果不写依赖数组，每次渲染后都会执行；如果依赖数组为空，只会在组件挂载后执行一次；如果依赖数组中有变量，那么组件挂载后会执行一次，之后变量变化时再次执行。

另外，`useEffect` 可以返回一个清理函数，用于组件卸载时或下一次 effect 执行前清理副作用，例如清除定时器、解绑事件、关闭连接等。

------

## 2.5 useRef

### 作用

`useRef` 用来保存一个在组件多次渲染之间保持不变的数据。

它的特点是：

```text
1. ref.current 可以保存数据。
2. ref.current 修改后不会触发组件重新渲染。
3. 组件重新渲染时，ref 对象不会重新创建。
```

常见使用场景：

```text
保存 DOM 元素
保存定时器 ID
保存上一次的值
保存防抖、节流中的标记
保存不需要触发渲染的数据
```

------

### 基本写法

```jsx
import { useRef } from 'react'

function Demo() {
  const timerRef = useRef(null)

  const start = () => {
    timerRef.current = setTimeout(() => {
      console.log('执行')
    }, 1000)
  }

  return <button onClick={start}>开始</button>
}
```

`useRef` 返回一个对象：

```js
{
  current: null
}
```

真正保存数据的是：

```js
timerRef.current
```

------

### 获取 DOM

```jsx
import { useRef } from 'react'

function InputDemo() {
  const inputRef = useRef(null)

  const focusInput = () => {
    inputRef.current.focus()
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>聚焦</button>
    </>
  )
}
```

这里的：

```jsx
<input ref={inputRef} />
```

会把真实 DOM 节点保存到：

```jsx
inputRef.current
```

------

### useRef 和 useState 的区别

| 对比项               | useState               | useRef                     |
| -------------------- | ---------------------- | -------------------------- |
| 是否触发重新渲染     | 会                     | 不会                       |
| 是否能跨渲染保存数据 | 能                     | 能                         |
| 适合保存什么         | 页面需要展示的数据     | 不需要展示但需要保存的数据 |
| 常见场景             | 表单值、列表、弹窗状态 | DOM、timer、旧值、防抖标记 |

------

### useRef 面试回答

`useRef` 用来保存组件多次渲染之间需要保留的数据。它返回一个对象，对象上有一个 `current` 属性。修改 `ref.current` 不会触发组件重新渲染，所以它适合保存一些不直接影响页面展示的数据，比如定时器 ID、DOM 节点、防抖 timer、上一次的值等。

和 `useState` 相比，`useState` 适合保存需要驱动页面更新的数据，而 `useRef` 适合保存不需要触发页面更新的数据。

------

## 2.6 useMemo

### 作用

`useMemo` 用来缓存计算结果。

函数组件每次重新渲染时，组件函数都会重新执行。如果组件内部有比较耗时的计算，那么每次渲染都会重新计算，可能造成性能浪费。

`useMemo` 可以让某个计算结果在依赖没有变化时复用上一次的结果。

------

### 基本写法

```jsx
const result = useMemo(() => {
  return heavyCalculate(list)
}, [list])
```

含义：

```text
只有 list 变化时，才重新执行 heavyCalculate。
如果 list 不变，就复用上一次的 result。
```

------

### 示例

```jsx
import { useMemo } from 'react'

function ProductList({ products, keyword }) {
  const filteredProducts = useMemo(() => {
    return products.filter(item => item.name.includes(keyword))
  }, [products, keyword])

  return (
    <ul>
      {filteredProducts.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  )
}
```

------

### 适合场景

```text
1. 大量数据的过滤、排序、计算。
2. 计算逻辑比较耗时。
3. 避免每次渲染都生成新的对象或数组。
4. 配合 React.memo 减少子组件重复渲染。
```

------

### useMemo 面试回答

`useMemo` 是用来缓存计算结果的 Hook。当组件重新渲染时，如果依赖项没有变化，React 会直接复用上一次的计算结果，而不是重新执行计算函数。它适合用于一些比较耗时的计算，比如大列表过滤、排序、复杂数据转换等。

不过 `useMemo` 不是所有地方都需要使用，只有当计算成本比较高，或者需要保持对象、数组引用稳定时，才有必要使用。

------

## 2.7 useCallback

### 作用

`useCallback` 用来缓存函数引用。

函数组件每次重新渲染时，组件内部定义的函数都会重新创建。

例如：

```jsx
function Parent() {
  const handleClick = () => {
    console.log('点击')
  }

  return <Child onClick={handleClick} />
}
```

每次 `Parent` 重新渲染，`handleClick` 都是一个新的函数。

如果这个函数作为 props 传给子组件，可能导致子组件不必要地重新渲染。

------

### 基本写法

```jsx
const handleClick = useCallback(() => {
  console.log('点击')
}, [])
```

含义：

```text
依赖项不变时，复用上一次的函数引用。
```

------

### 示例

```jsx
import { useCallback, useState } from 'react'

function Parent() {
  const [count, setCount] = useState(0)

  const handleClick = useCallback(() => {
    console.log('点击')
  }, [])

  return (
    <>
      <button onClick={() => setCount(count + 1)}>更新父组件</button>
      <Child onClick={handleClick} />
    </>
  )
}
```

------

### 适合场景

```text
1. 函数作为 props 传递给子组件。
2. 子组件使用了 React.memo。
3. 函数作为 useEffect 的依赖。
4. 自定义 Hook 需要返回稳定函数。
```

------

### useCallback 和 useMemo 的区别

| Hook          | 缓存内容     |
| ------------- | ------------ |
| `useMemo`     | 缓存计算结果 |
| `useCallback` | 缓存函数引用 |

可以这样理解：

```jsx
useCallback(fn, deps)
```

大致等价于：

```jsx
useMemo(() => fn, deps)
```

------

### useCallback 面试回答

`useCallback` 是用来缓存函数引用的 Hook。函数组件每次渲染时，内部函数都会重新创建。如果这个函数传给子组件，可能导致子组件不必要的更新。使用 `useCallback` 后，只要依赖项不变，就可以复用上一次的函数引用。

它通常和 `React.memo` 配合使用，用来优化子组件重复渲染的问题。但如果函数没有传给子组件，也没有作为其他 Hook 的依赖，就不一定需要使用 `useCallback`。

------

## 2.8 useContext

### 作用

`useContext` 用来跨组件共享数据，解决 props 层层传递的问题。

例如很多组件都需要这些信息：

```text
用户信息
登录状态
主题颜色
语言设置
权限信息
```

如果通过 props 一层一层传递，会很麻烦：

```text
App → Layout → Header → UserInfo
```

这种问题叫：

```text
props drilling，属性透传
```

`useContext` 可以让深层组件直接读取共享数据。

------

### 基本写法

```jsx
import { createContext, useContext } from 'react'

const UserContext = createContext(null)

function App() {
  const user = { name: '张三' }

  return (
    <UserContext.Provider value={user}>
      <Header />
    </UserContext.Provider>
  )
}

function Header() {
  return <UserInfo />
}

function UserInfo() {
  const user = useContext(UserContext)

  return <div>{user.name}</div>
}
```

------

### useContext 面试回答

`useContext` 是用来读取 React Context 数据的 Hook，主要解决多层组件之间共享数据的问题。它可以避免通过 props 一层一层传递数据。

常见使用场景包括用户信息、登录状态、主题配置、语言配置、权限信息等。需要注意的是，Context 更适合共享全局或局部全局的数据，如果只是父子组件之间简单传值，直接使用 props 更合适。

------

## 2.9 自定义 Hook

### 作用

自定义 Hook 用来复用状态逻辑。

如果多个组件中有相同的逻辑，例如：

```text
接口请求
防抖节流
窗口尺寸监听
表单处理
本地缓存读取
权限判断
```

就可以把这些逻辑封装成自定义 Hook。

------

### 命名规则

自定义 Hook 必须以 `use` 开头，例如：

```text
useRequest
useDebounce
useLocalStorage
useWindowSize
useAuth
```

这样 React 才能识别它内部使用的 Hook，并检查 Hook 使用规则。

------

### 示例：防抖 Hook

```jsx
import { useRef, useEffect, useCallback } from 'react'

function useDebounceFn(fn, delay) {
  const timerRef = useRef(null)
  const fnRef = useRef(fn)

  useEffect(() => {
    fnRef.current = fn
  }, [fn])

  const debouncedFn = useCallback((...args) => {
    clearTimeout(timerRef.current)

    timerRef.current = setTimeout(() => {
      fnRef.current(...args)
    }, delay)
  }, [delay])

  useEffect(() => {
    return () => {
      clearTimeout(timerRef.current)
    }
  }, [])

  return debouncedFn
}
```

使用：

```jsx
function SearchBox() {
  const handleSearch = useDebounceFn((value) => {
    console.log('搜索:', value)
  }, 500)

  return (
    <input onChange={(e) => handleSearch(e.target.value)} />
  )
}
```

------

### 节流 Hook 写法

```
import { useRef, useEffect, useCallback } from 'react'

function useThrottleFn(fn, delay) {
  const lastTimeRef = useRef(0)
  const fnRef = useRef(fn)

  useEffect(() => {
    fnRef.current = fn
  }, [fn])

  const throttledFn = useCallback((...args) => {
    const now = Date.now()

    if (now - lastTimeRef.current >= delay) {
      lastTimeRef.current = now
      fnRef.current(...args)
    }
  }, [delay])

  return throttledFn
}
```

使用

```
function ScrollDemo() {
  const handleScroll = useThrottleFn(() => {
    console.log('滚动事件触发')
  }, 1000)

  useEffect(() => {
    window.addEventListener('scroll', handleScroll)

    return () => {
      window.removeEventListener('scroll', handleScroll)
    }
  }, [handleScroll])

  return <div style={{ height: '2000px' }}>滚动页面</div>
}
```

----





### 自定义 Hook 面试回答

自定义 Hook 是把多个组件中可复用的状态逻辑提取出来，封装成一个以 `use` 开头的函数。它本质上还是函数，只是内部可以调用 React 的其他 Hook。

自定义 Hook 常用于封装接口请求、防抖节流、表单逻辑、权限判断、本地缓存、窗口监听等逻辑。它解决的是状态逻辑复用问题，相比普通工具函数，自定义 Hook 可以使用 `useState`、`useEffect`、`useRef` 等 React 能力。

------

## 2.10 Hook 的使用规则

Hook 有两条核心规则。

------

### 规则一：只能在函数组件或自定义 Hook 中调用

正确写法：

```jsx
function App() {
  const [count, setCount] = useState(0)
}
```

正确写法：

```jsx
function useCounter() {
  const [count, setCount] = useState(0)
}
```

错误写法：

```jsx
function normalFn() {
  const [count, setCount] = useState(0)
}
```

------

### 规则二：不能在条件、循环、嵌套函数中调用 Hook

错误写法：

```jsx
function App({ show }) {
  if (show) {
    const [count, setCount] = useState(0)
  }

  return <div>App</div>
}
```

正确写法：

```jsx
function App({ show }) {
  const [count, setCount] = useState(0)

  if (!show) {
    return null
  }

  return <div>{count}</div>
}
```

------

### 为什么 Hook 不能写在 if 里面

React 是根据 Hook 的调用顺序来保存状态的。

例如：

```jsx
function Demo() {
  const [name, setName] = useState('张三')
  const [age, setAge] = useState(18)
  const ref = useRef(null)
}
```

React 内部大致可以理解为：

```text
第 1 个 Hook：name
第 2 个 Hook：age
第 3 个 Hook：ref
```

如果把 Hook 写在条件判断里：

```jsx
function Demo({ flag }) {
  const [name, setName] = useState('张三')

  if (flag) {
    const [age, setAge] = useState(18)
  }

  const ref = useRef(null)
}
```

第一次 `flag = true`：

```text
第 1 个 Hook：name
第 2 个 Hook：age
第 3 个 Hook：ref
```

第二次 `flag = false`：

```text
第 1 个 Hook：name
第 2 个 Hook：ref
```

Hook 顺序乱了，React 就无法正确对应每个状态，所以 Hook 必须写在组件最外层。

------

## 2.11 常见 Hook 对比

| Hook          | 作用               | 是否触发渲染             | 常见场景                   |
| ------------- | ------------------ | ------------------------ | -------------------------- |
| `useState`    | 保存组件状态       | 修改后触发               | 表单、计数器、弹窗状态     |
| `useEffect`   | 处理副作用         | 不直接触发               | 请求接口、定时器、事件监听 |
| `useRef`      | 保存可变数据或 DOM | 修改后不触发             | DOM、timer、旧值           |
| `useMemo`     | 缓存计算结果       | 不直接触发               | 大数据计算、过滤、排序     |
| `useCallback` | 缓存函数引用       | 不直接触发               | 传递给子组件的函数         |
| `useContext`  | 跨组件共享数据     | value 变化会影响消费组件 | 用户信息、主题、语言       |

------

## 2.12 面试回答案例

### 问题：React Hook 是什么？解决了什么问题？

可以回答：

React Hook 是 React 提供给函数组件使用的一组特殊函数，它让函数组件也可以拥有状态管理、副作用处理、生命周期控制和逻辑复用等能力。

在 Hook 出现之前，如果组件需要状态或生命周期，通常要写类组件，比如通过 `state` 保存数据，通过 `componentDidMount`、`componentDidUpdate`、`componentWillUnmount` 处理生命周期。但类组件写法相对复杂，而且同一类业务逻辑可能分散在多个生命周期方法中，维护起来不方便。

Hook 出现之后，函数组件也可以完成这些能力。比如 `useState` 用来保存组件状态，`useEffect` 用来处理副作用和生命周期相关逻辑，`useRef` 用来保存不会触发重新渲染的数据，`useMemo` 和 `useCallback` 用于性能优化，`useContext` 用于跨组件共享数据。

在实际开发中，我比较常用 `useState`、`useEffect` 和 `useRef`。例如搜索框中，可以用 `useState` 保存输入值，用 `useEffect` 监听关键字变化并发送请求，也可以用 `useRef` 保存防抖定时器，避免组件重新渲染时 timer 丢失。

所以我理解 Hook 的核心价值是：让函数组件具备类组件原本才有的状态和生命周期能力，同时让逻辑复用更加方便。

------

### 问题：useState 和 useRef 有什么区别？

可以回答：

`useState` 和 `useRef` 都可以在组件多次渲染之间保存数据，但它们最大的区别是：`useState` 更新后会触发组件重新渲染，而 `useRef` 修改 `current` 后不会触发重新渲染。

所以，如果这个数据需要展示到页面上，或者它变化后需要驱动 UI 更新，就应该使用 `useState`。如果只是想保存一个值，比如定时器 ID、DOM 节点、防抖 timer、上一次的值，并不希望它变化时重新渲染页面，就适合使用 `useRef`。

------

### 问题：useEffect 的依赖数组有什么作用？

可以回答：

`useEffect` 的依赖数组用来控制副作用的执行时机。

如果不写依赖数组，effect 会在组件每次渲染后都执行。如果依赖数组为空，effect 只会在组件挂载后执行一次，通常用于初始化请求、绑定事件、设置定时器等。如果依赖数组中有变量，那么组件挂载后会执行一次，之后这个变量发生变化时，effect 会再次执行。

另外，`useEffect` 可以返回一个清理函数，用于组件卸载时或者下一次 effect 执行前清理副作用，比如清除定时器、解绑事件、取消订阅等。

------

### 问题：为什么 Hook 不能写在 if 里面？

可以回答：

Hook 不能写在 `if`、循环或者嵌套函数中，因为 React 是根据 Hook 的调用顺序来保存状态的。

如果 Hook 写在条件判断里，不同渲染过程中条件可能不同，导致 Hook 的调用顺序发生变化。这样 React 就无法判断哪个状态对应哪个 Hook，组件状态可能会错乱。

所以 Hook 必须写在函数组件的最外层，保证每次渲染时 Hook 的调用顺序一致。

------

### 问题：useMemo 和 useCallback 有什么区别？

可以回答：

`useMemo` 和 `useCallback` 都是用于性能优化的 Hook，但是它们缓存的内容不同。

`useMemo` 缓存的是计算结果，适合用于复杂计算、列表过滤、排序等场景。`useCallback` 缓存的是函数引用，适合用于函数作为 props 传给子组件，或者函数作为其他 Hook 依赖的场景。

简单来说，`useMemo` 缓存值，`useCallback` 缓存函数。需要注意的是，它们不是必须到处使用，只有在计算成本较高、对象或函数引用稳定性会影响性能时才有必要使用。