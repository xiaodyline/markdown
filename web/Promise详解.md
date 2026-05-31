# Promise 详细讲解

## 1. Promise 解决了什么问题

在没有 Promise 之前，异步代码通常这样写：

```js
getUser(function (user) {
  getMenu(user.id, function (menu) {
    getConfig(menu.id, function (config) {
      render(user, menu, config)
    })
  })
})
```

问题是：

```text
1. 回调嵌套太深，代码难维护
2. 错误处理分散
3. 多个异步任务并发控制麻烦
4. 无法方便地链式组织异步流程
```

Promise 的作用是：

> 用一个对象表示一个异步操作的最终结果，让异步代码可以用链式方式组织。

------

# 2. Promise 是什么

Promise 本质上是一个对象。

它代表一个异步任务的结果。

```js
const p = new Promise((resolve, reject) => {
  // 异步任务
})
```

Promise 有三种状态：

| 状态        | 含义   |
| ----------- | ------ |
| `pending`   | 等待中 |
| `fulfilled` | 已成功 |
| `rejected`  | 已失败 |

状态变化只能有两种：

```text
pending -> fulfilled
pending -> rejected
```

一旦状态确定，就不能再改变。

------

# 3. resolve 和 reject 是什么

创建 Promise 时，会传入一个函数：

```js
new Promise((resolve, reject) => {
  
})
```

这里的：

```js
resolve
reject
```

是 Promise 内部提供的两个函数。

## resolve

表示异步任务成功。

```js
const p = new Promise((resolve, reject) => {
  resolve('成功结果')
})
```

此时 Promise 状态变成：

```text
pending -> fulfilled
```

## reject

表示异步任务失败。

```js
const p = new Promise((resolve, reject) => {
  reject('失败原因')
})
```

此时 Promise 状态变成：

```text
pending -> rejected
```

------

# 4. Promise 状态一旦改变就不能再变

这是你之前问过的重点。

```js
const p = new Promise((resolve, reject) => {
  resolve('成功')
  reject('失败')
  resolve('再次成功')
})
```

最终结果是：

```text
fulfilled，值是 '成功'
```

因为第一次调用的是：

```js
resolve('成功')
```

Promise 状态已经从 `pending` 变成了 `fulfilled`。

后面的：

```js
reject('失败')
resolve('再次成功')
```

都会被忽略。

再看反过来：

```js
const p = new Promise((resolve, reject) => {
  reject('失败')
  resolve('成功')
})
```

最终结果是：

```text
rejected，原因是 '失败'
```

总结：

> Promise 只认第一次状态改变，后面的 resolve 或 reject 都无效。

------

# 5. then 是什么

`then` 用来接收 Promise 成功或失败的结果。

```js
promise.then(
  value => {
    console.log('成功', value)
  },
  reason => {
    console.log('失败', reason)
  }
)
```

第一个函数处理成功：

```js
value => {}
```

第二个函数处理失败：

```js
reason => {}
```

例如：

```js
const p = new Promise((resolve, reject) => {
  resolve('hello')
})

p.then(value => {
  console.log(value)
})
```

输出：

```text
hello
```

------

# 6. catch 是什么

`catch` 专门处理失败。

```js
promise.catch(error => {
  console.log(error)
})
```

它等价于：

```js
promise.then(null, error => {
  console.log(error)
})
```

例如：

```js
const p = new Promise((resolve, reject) => {
  reject('出错了')
})

p.catch(error => {
  console.log(error)
})
```

输出：

```text
出错了
```

------

# 7. finally 是什么

`finally` 表示不管成功还是失败，最后都会执行。

```js
promise
  .then(res => {
    console.log('成功', res)
  })
  .catch(err => {
    console.log('失败', err)
  })
  .finally(() => {
    console.log('结束')
  })
```

常用于：

```text
关闭 loading
清理资源
恢复按钮状态
```

例如：

```js
async function submit() {
  loading = true

  request()
    .then(res => {
      console.log(res)
    })
    .catch(err => {
      console.log(err)
    })
    .finally(() => {
      loading = false
    })
}
```

------

# 8. then 返回的是一个新的 Promise 吗？

是的。

这是非常重要的面试点：

> `then` 方法一定会返回一个新的 Promise 对象。

例如：

```js
const p1 = Promise.resolve(1)

const p2 = p1.then(value => {
  return value + 1
})

console.log(p1 === p2)
```

结果：

```text
false
```

说明 `p2` 是一个新的 Promise。

------

# 9. then 返回新 Promise 的规则

## 情况一：then 中返回普通值

```js
Promise.resolve(1)
  .then(value => {
    return value + 1
  })
  .then(value => {
    console.log(value)
  })
```

输出：

```text
2
```

解释：

```js
return value + 1
```

返回的是普通值 `2`，所以下一个 `then` 接收到 `2`。

等价于：

```js
return Promise.resolve(2)
```

------

## 情况二：then 中返回 Promise

```js
Promise.resolve(1)
  .then(value => {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve(value + 1)
      }, 1000)
    })
  })
  .then(value => {
    console.log(value)
  })
```

1 秒后输出：

```text
2
```

解释：

如果 `then` 里面返回的是 Promise，那么下一个 `then` 会等待这个 Promise 完成。

------

## 情况三：then 中抛出错误

```js
Promise.resolve(1)
  .then(value => {
    throw new Error('出错了')
  })
  .then(value => {
    console.log('成功', value)
  })
  .catch(error => {
    console.log('失败', error.message)
  })
```

输出：

```text
失败 出错了
```

解释：

`then` 中抛出异常，会让返回的新 Promise 变成 `rejected` 状态。

------

## 情况四：then 没有 return

```js
Promise.resolve(1)
  .then(value => {
    value + 1
  })
  .then(value => {
    console.log(value)
  })
```

输出：

```text
undefined
```

因为第一个 `then` 没有返回值，默认返回 `undefined`。

------

# 10. Promise 链式调用的本质

```js
doA()
  .then(resA => {
    return doB(resA)
  })
  .then(resB => {
    return doC(resB)
  })
  .then(resC => {
    console.log(resC)
  })
  .catch(error => {
    console.log(error)
  })
```

本质是：

```text
每个 then 都返回一个新的 Promise
后一个 then 等待前一个 then 返回的 Promise 完成
```

所以 Promise 可以链式调用。

------

# 11. Promise 和事件循环

Promise 的回调属于 **微任务**。

先看例子：

```js
console.log('1')

setTimeout(() => {
  console.log('2')
}, 0)

Promise.resolve().then(() => {
  console.log('3')
})

console.log('4')
```

输出顺序：

```text
1
4
3
2
```

原因：

```text
1. 同步代码先执行
2. Promise.then 回调进入微任务队列
3. setTimeout 回调进入宏任务队列
4. 当前同步代码执行完
5. 清空微任务队列
6. 执行下一个宏任务
```

所以执行顺序是：

```text
同步任务 -> 微任务 -> 宏任务
```

------

# 12. Promise 状态改变后，then 会马上执行吗？

不会。

即使 Promise 已经 resolve，`then` 回调也不会立即同步执行，而是进入微任务队列。

```js
const p = Promise.resolve('成功')

p.then(res => {
  console.log(res)
})

console.log('结束')
```

输出：

```text
结束
成功
```

原因：

```text
then 回调是微任务，要等同步代码执行完后再执行
```

------

# 13. 常见微任务和宏任务

## 微任务

常见微任务：

```text
Promise.then
Promise.catch
Promise.finally
queueMicrotask
MutationObserver
```

## 宏任务

常见宏任务：

```text
setTimeout
setInterval
setImmediate，Node.js 中
I/O
UI 渲染
script 整体代码
```

面试常说：

> 每执行完一个宏任务，都会清空当前所有微任务，然后再进入下一个宏任务。

------

# 14. async / await 和 Promise 的关系

`async / await` 是 Promise 的语法糖。

也就是说：

```js
async function getData() {
  const res = await request()
  return res
}
```

本质上还是 Promise。

## async 函数一定返回 Promise

```js
async function fn() {
  return 123
}

console.log(fn())
```

输出的是：

```text
Promise
```

等价于：

```js
function fn() {
  return Promise.resolve(123)
}
```

------

## await 后面通常接 Promise

```js
async function fn() {
  const res = await Promise.resolve(123)
  console.log(res)
}
```

输出：

```text
123
```

`await` 的作用是：

> 等待 Promise 完成，并拿到成功结果。

------

## await 遇到 reject

```js
async function fn() {
  try {
    const res = await Promise.reject('失败')
  } catch (error) {
    console.log(error)
  }
}
```

输出：

```text
失败
```

所以在 `async / await` 中，错误通常用 `try...catch` 捕获。

------

# 15. Promise.all

`Promise.all` 用于并发执行多个 Promise，并等待它们全部成功。

```js
const result = await Promise.all([
  fetchUser(),
  fetchMenu(),
  fetchConfig()
])
```

返回结果是数组：

```js
[user, menu, config]
```

特点：

```text
1. 多个任务同时执行
2. 全部成功，整体才成功
3. 任意一个失败，整体立即失败
4. 返回结果顺序和传入顺序一致
```

例如：

```js
const [user, menu, config] = await Promise.all([
  fetchUser(),
  fetchMenu(),
  fetchConfig()
])
```

即使 `fetchConfig` 最先完成，结果顺序仍然是：

```text
user
menu
config
```

## 使用场景

适合多个请求之间没有依赖，并且必须全部成功的情况。

例如：

```text
首页初始化，需要用户信息、菜单、配置都加载成功
```

------

# 16. Promise.allSettled

`Promise.allSettled` 用于等待所有 Promise 都结束，不管成功还是失败。

```js
const results = await Promise.allSettled([
  fetchUser(),
  fetchMenu(),
  fetchConfig()
])
```

返回结果类似：

```js
[
  { status: 'fulfilled', value: user },
  { status: 'rejected', reason: error },
  { status: 'fulfilled', value: config }
]
```

特点：

```text
1. 多个任务同时执行
2. 不会因为某一个失败就中断
3. 会等所有任务都有结果
4. 可以分别知道每个任务成功还是失败
```

## 使用场景

适合允许部分失败的场景。

例如：

```text
页面有多个独立模块
用户信息失败，不影响配置加载
菜单失败，可以展示默认菜单
```

------

# 17. Promise.race

`Promise.race` 表示竞速。

```js
const result = await Promise.race([
  request(),
  timeout()
])
```

谁先完成，就采用谁的结果。

注意，这里的完成包括：

```text
fulfilled
rejected
```

例如：

```js
Promise.race([
  new Promise(resolve => setTimeout(() => resolve('A'), 1000)),
  new Promise(resolve => setTimeout(() => resolve('B'), 500))
]).then(res => {
  console.log(res)
})
```

输出：

```text
B
```

因为 B 先完成。

## 使用场景：请求超时

```js
function timeout(ms) {
  return new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Timeout')), ms)
  })
}

async function init() {
  try {
    const result = await Promise.race([
      fetchData(),
      timeout(8000)
    ])

    render(result)
  } catch (error) {
    showError(error)
  }
}
```

注意：

> `Promise.race` 只能让外层 Promise 超时失败，不会真正取消底层请求。

如果要取消 fetch 请求，需要配合 `AbortController`。

------

# 18. Promise.any

`Promise.any` 表示：

> 只要有一个 Promise 成功，整体就成功。

```js
const result = await Promise.any([
  requestA(),
  requestB(),
  requestC()
])
```

特点：

```text
1. 只要一个成功，就返回这个成功结果
2. 全部失败，整体才失败
```

## 和 Promise.race 的区别

| 方法           | 规则                                     |
| -------------- | ---------------------------------------- |
| `Promise.race` | 谁先完成就用谁，不管成功还是失败         |
| `Promise.any`  | 谁先成功就用谁，失败会继续等其他 Promise |

例如：

```js
Promise.race([
  Promise.reject('A 失败'),
  Promise.resolve('B 成功')
])
```

结果：

```text
rejected: A 失败
```

因为 `A` 先失败了。

而：

```js
Promise.any([
  Promise.reject('A 失败'),
  Promise.resolve('B 成功')
])
```

结果：

```text
fulfilled: B 成功
```

因为它会等待第一个成功的结果。

------

# 19. Promise.resolve

把一个值包装成成功状态的 Promise。

```js
Promise.resolve(123).then(res => {
  console.log(res)
})
```

输出：

```text
123
```

等价于：

```js
new Promise(resolve => {
  resolve(123)
})
```

常见用途：

```js
function getData() {
  if (cache) {
    return Promise.resolve(cache)
  }

  return fetchData()
}
```

这样无论有没有缓存，返回值都是 Promise，调用方式统一。

------

# 20. Promise.reject

返回一个失败状态的 Promise。

```js
Promise.reject('失败').catch(error => {
  console.log(error)
})
```

输出：

```text
失败
```

等价于：

```js
new Promise((resolve, reject) => {
  reject('失败')
})
```

------

# 21. 你的四个方案回顾

## 方案 A：串行

```js
const user = await fetchUser()
const menu = await fetchMenu()
const config = await fetchConfig()
```

特点：

```text
一个完成后再执行下一个
总耗时是三个请求耗时相加
适合后一个请求依赖前一个请求的情况
```

------

## 方案 B：Promise.all 并发

```js
const [user, menu, config] = await Promise.all([
  fetchUser(),
  fetchMenu(),
  fetchConfig()
])
```

特点：

```text
三个请求同时发起
总耗时约等于最慢的那个请求
任意一个失败，整体失败
适合全部都必须成功的情况
```

------

## 方案 C：Promise.allSettled

```js
const results = await Promise.allSettled([
  fetchUser(),
  fetchMenu(),
  fetchConfig()
])
```

特点：

```text
三个请求同时发起
等待所有请求结束
可以分别知道每个请求成功还是失败
适合允许部分失败的情况
```

------

## 方案 D：Promise.race 超时

```js
const results = await Promise.race([
  timeout,
  Promise.all([fetchUser(), fetchMenu(), fetchConfig()])
])
```

特点：

```text
业务请求和超时 Promise 竞速
如果请求先完成，就正常渲染
如果超时先发生，就进入错误处理
适合给请求加最大等待时间
```

------

# 22. Promise 的底层原理

Promise 底层可以理解为三个核心点：

```text
1. 状态管理
2. 回调收集
3. 微任务调度
```

------

## 22.1 状态管理

Promise 内部维护一个状态：

```js
state = 'pending'
```

还有一个结果值：

```js
value = undefined
reason = undefined
```

当调用：

```js
resolve(data)
```

状态变成：

```js
state = 'fulfilled'
value = data
```

当调用：

```js
reject(error)
```

状态变成：

```js
state = 'rejected'
reason = error
```

并且状态只能改变一次。

------

## 22.2 回调收集

如果 Promise 还没完成，你就调用了 `then`：

```js
const p = new Promise(resolve => {
  setTimeout(() => {
    resolve('成功')
  }, 1000)
})

p.then(res => {
  console.log(res)
})
```

此时 Promise 还在 `pending` 状态，所以 `then` 里的回调不能马上执行。

Promise 会先把回调存起来：

```js
onFulfilledCallbacks = []
onRejectedCallbacks = []
```

等到 1 秒后调用 `resolve('成功')`，再依次执行之前保存的成功回调。

------

## 22.3 微任务调度

Promise 的 `then` 回调不是同步执行，而是放入微任务队列。

```js
Promise.resolve().then(() => {
  console.log('promise')
})

console.log('sync')
```

输出：

```text
sync
promise
```

说明：

```text
resolve 改变状态
then 回调进入微任务队列
同步代码执行完
再执行微任务
```

------

# 23. 简化版 Promise 实现

下面是一个非常简化版，帮助理解底层，不是完整规范实现。

```js
class MyPromise {
  constructor(executor) {
    this.state = 'pending'
    this.value = undefined
    this.reason = undefined

    this.onFulfilledCallbacks = []
    this.onRejectedCallbacks = []

    const resolve = value => {
      if (this.state !== 'pending') return

      // 如果 resolve 的是一个 MyPromise，就等待它的结果
      if (value instanceof MyPromise) {
        return value.then(resolve, reject)
      }

      queueMicrotask(() => {
        if (this.state !== 'pending') return

        this.state = 'fulfilled'
        this.value = value

        this.onFulfilledCallbacks.forEach(fn => fn(this.value))
      })
    }

    const reject = reason => {
      if (this.state !== 'pending') return

      queueMicrotask(() => {
        if (this.state !== 'pending') return

        this.state = 'rejected'
        this.reason = reason

        this.onRejectedCallbacks.forEach(fn => fn(this.reason))
      })
    }

    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }

  then(onFulfilled, onRejected) {
    onFulfilled =
      typeof onFulfilled === 'function'
        ? onFulfilled
        : value => value

    onRejected =
      typeof onRejected === 'function'
        ? onRejected
        : reason => {
            throw reason
          }

    return new MyPromise((resolve, reject) => {
      const handleFulfilled = value => {
        try {
          const result = onFulfilled(value)
          resolve(result)
        } catch (error) {
          reject(error)
        }
      }

      const handleRejected = reason => {
        try {
          const result = onRejected(reason)
          resolve(result)
        } catch (error) {
          reject(error)
        }
      }

      if (this.state === 'fulfilled') {
        queueMicrotask(() => {
          handleFulfilled(this.value)
        })
      } else if (this.state === 'rejected') {
        queueMicrotask(() => {
          handleRejected(this.reason)
        })
      } else {
        this.onFulfilledCallbacks.push(handleFulfilled)
        this.onRejectedCallbacks.push(handleRejected)
      }
    })
  }

  catch(onRejected) {
    return this.then(null, onRejected)
  }

  finally(callback) {
    return this.then(
      value => {
        return MyPromise.resolve(callback()).then(() => value)
      },
      reason => {
        return MyPromise.resolve(callback()).then(() => {
          throw reason
        })
      }
    )
  }

  static resolve(value) {
    return new MyPromise(resolve => {
      resolve(value)
    })
  }

  static reject(reason) {
    return new MyPromise((resolve, reject) => {
      reject(reason)
    })
  }
}
```

这个简化版体现了几个核心：

```text
1. Promise 有状态
2. resolve / reject 只能改一次状态
3. then 会收集回调
4. then 返回新的 Promise
5. then 回调通过微任务执行
```

------

# 24. Promise.all 简化实现

```js
Promise.myAll = function (promises) {
  return new Promise((resolve, reject) => {
    const results = []
    let count = 0

    if (promises.length === 0) {
      resolve([])
      return
    }

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value
          count++

          if (count === promises.length) {
            resolve(results)
          }
        })
        .catch(error => {
          reject(error)
        })
    })
  })
}
```

关键点：

```text
1. 结果顺序按传入顺序保存
2. 全部成功才 resolve
3. 任意一个失败就 reject
```

------

# 25. Promise.allSettled 简化实现

```js
Promise.myAllSettled = function (promises) {
  return new Promise(resolve => {
    const results = []
    let count = 0

    if (promises.length === 0) {
      resolve([])
      return
    }

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = {
            status: 'fulfilled',
            value
          }
        })
        .catch(reason => {
          results[index] = {
            status: 'rejected',
            reason
          }
        })
        .finally(() => {
          count++

          if (count === promises.length) {
            resolve(results)
          }
        })
    })
  })
}
```

关键点：

```text
1. 不会因为某个失败就 reject
2. 每个结果都记录 status
3. 等全部完成后统一 resolve
```

------

# 26. Promise.race 简化实现

```js
Promise.myRace = function (promises) {
  return new Promise((resolve, reject) => {
    promises.forEach(promise => {
      Promise.resolve(promise)
        .then(resolve)
        .catch(reject)
    })
  })
}
```

关键点：

```text
谁先完成，就采用谁的状态和结果
```

------

# 27. 常见面试题

## 题 1：输出顺序

```js
console.log('1')

Promise.resolve().then(() => {
  console.log('2')
})

setTimeout(() => {
  console.log('3')
}, 0)

console.log('4')
```

输出：

```text
1
4
2
3
```

------

## 题 2：状态改变

```js
const p = new Promise((resolve, reject) => {
  resolve('A')
  reject('B')
})

p.then(res => {
  console.log(res)
}).catch(err => {
  console.log(err)
})
```

输出：

```text
A
```

原因：

```text
状态只能改变一次
```

------

## 题 3：then 返回值

```js
Promise.resolve(1)
  .then(res => {
    return res + 1
  })
  .then(res => {
    console.log(res)
  })
```

输出：

```text
2
```

------

## 题 4：then 中抛错

```js
Promise.resolve(1)
  .then(res => {
    throw new Error('error')
  })
  .then(res => {
    console.log('success', res)
  })
  .catch(err => {
    console.log('catch', err.message)
  })
```

输出：

```text
catch error
```

------

# 28. 最终总结

Promise 需要掌握这些点：

```text
1. Promise 是异步编程的一种解决方案
2. Promise 有 pending、fulfilled、rejected 三种状态
3. 状态只能从 pending 变成 fulfilled 或 rejected，且只能改变一次
4. resolve 表示成功，reject 表示失败
5. then 用于处理成功或失败结果
6. catch 用于捕获失败
7. finally 不管成功失败都会执行
8. then 会返回一个新的 Promise，所以可以链式调用
9. Promise.then 回调属于微任务
10. Promise.all 适合全部成功才继续
11. Promise.allSettled 适合收集所有成功和失败结果
12. Promise.race 适合竞速和超时控制
13. Promise.any 适合多个任务中只要有一个成功即可
14. async / await 是 Promise 的语法糖
```

面试回答可以压缩成：

> Promise 是 JavaScript 中处理异步操作的对象，它有 `pending`、`fulfilled`、`rejected` 三种状态，状态一旦从 `pending` 变为成功或失败就不能再改变。`resolve` 用于把 Promise 改为成功状态，`reject` 用于改为失败状态。`then` 会返回一个新的 Promise，因此可以链式调用；如果 `then` 中返回普通值，下一个 `then` 接收这个值；如果返回 Promise，下一个 `then` 会等待它完成；如果抛出异常，则进入 `catch`。Promise 的回调属于微任务，会在同步代码执行完后、宏任务之前执行。常用方法包括 `Promise.all`、`Promise.allSettled`、`Promise.race`、`Promise.any`、`Promise.resolve` 和 `Promise.reject`。



```


```

