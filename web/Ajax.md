

## 请求怎么取消以及后端如何知道

在传统 Ajax 中，可以通过 `XMLHttpRequest` 对象的 `abort()` 方法取消正在进行的请求。调用 `abort()` 后，浏览器会中止当前 HTTP 请求，前端会触发 `onabort` 事件。

如果使用 `fetch`，则需要配合 `AbortController`，把 `controller.signal` 传给 `fetch`，再通过 `controller.abort()` 取消请求。

后端感知请求取消的方式不是收到一个特殊参数，而是监听客户端连接是否关闭。比如在 Node.js 中可以监听 `req.on('close')` 或 `req.on('aborted')`。不过前端取消请求只代表连接断开，后端正在执行的耗时任务不一定会自动停止。如果想真正停止后端任务，需要在后端维护取消标记，并在耗时逻辑中主动检查这个标记。