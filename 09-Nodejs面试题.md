[toc]



## 一、Node.js 基础认知

### 说说你对 Node.js 的理解？优缺点和应用场景是什么？

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行时，让 JavaScript 可以脱离浏览器在服务端运行。

它的核心特点：

1. 单线程事件循环。
2. 非阻塞 I/O。
3. 事件驱动。
4. 适合高并发、I/O 密集型场景。

优点：

1. 前后端语言统一，协作成本低。
2. I/O 处理效率高，适合高并发连接。
3. 包生态丰富，开发效率高。
4. 很适合做中间层、BFF、工具链、实时服务。

缺点：

1. 不适合重 CPU 计算任务。
2. 单线程模型下，长时间同步任务会阻塞整个进程。
3. 回调、异步链路复杂时需要较好的工程治理。

常见应用场景：

1. Web 服务和 API 服务。
2. SSR 和中间层。
3. 实时通信，如聊天室、推送服务。
4. 脚手架、构建工具、自动化脚本。
5. 网关、代理层、文件服务。

### 说说 Node.js 有哪些全局对象？

Node 中的全局对象主要分两类：

1. 真正挂在全局作用域上的对象。
2. 模块级别可直接使用的对象。

常见全局对象有：

1. `global`
2. `process`
3. `Buffer`
4. `console`
5. `setTimeout`、`setInterval`
6. `clearTimeout`、`clearInterval`

常见模块级对象有：

1. `__dirname`
2. `__filename`
3. `module`
4. `exports`
5. `require`

面试时要注意：并不是所有“能直接用”的对象都是真正意义上的全局对象，有些是模块包装函数注入的。

### 说说对 Node 中 process 的理解？有哪些常用属性和方法？

`process` 是 Node 提供的进程对象，用来获取当前进程信息并进行进程级控制。

常见用途：

1. 获取环境变量：`process.env`
2. 获取命令行参数：`process.argv`
3. 获取当前工作目录：`process.cwd()`
4. 退出进程：`process.exit()`
5. 注册进程事件：`beforeExit`、`exit`、`uncaughtException`、`unhandledRejection`
6. 调度微任务：`process.nextTick()`

一句话总结：`process` 是 Node 运行时与当前进程交互的核心入口。

### Node 中的标准输入、标准输出、标准错误是什么？

Node 进程启动后默认会有三条标准流：

1. `process.stdin`：标准输入
2. `process.stdout`：标准输出
3. `process.stderr`：标准错误输出

它们的作用分别是：

1. 从命令行或管道读取输入。
2. 向终端输出正常结果。
3. 向终端输出错误信息。

这也是为什么很多 Node 脚本、CLI 工具能够很自然地和终端、管道、重定向配合工作。

### 说说 node 中 module.exports 与 exports 有什么区别。

两者都和 CommonJS 模块导出有关，但本质不同。

1. `module.exports` 才是真正导出的对象。
2. `exports` 只是 `module.exports` 的一个引用。

所以：

1. 给 `exports.xxx = ...` 赋值，本质上是在给 `module.exports` 添加属性。
2. 如果直接给 `exports = {}` 重新赋值，不会影响最终导出。
3. 如果要整体替换导出对象，应该直接操作 `module.exports`。

## 二、核心模块与常见能力

### 说说对 Node 中 fs 模块的理解？有哪些常用方法？

`fs` 是 Node 中负责文件系统操作的核心模块，可以读写文件、操作目录、监听文件变化等。

常见方法有：

1. `readFile`、`readFileSync`
2. `writeFile`、`writeFileSync`
3. `appendFile`
4. `mkdir`
5. `readdir`
6. `stat`
7. `unlink`
8. `rename`
9. `createReadStream`、`createWriteStream`
10. `watch`

实际项目里更推荐优先用异步 API，避免阻塞事件循环。

### 如何监控文件的变动？

常见方式有：

1. `fs.watch`
2. `fs.watchFile`
3. 使用更稳定的第三方库，比如 `chokidar`

如果是工程场景，如构建工具、热更新、日志监听，通常更推荐成熟库，因为不同平台下原生监听行为可能有差异。

### Node 中如何判断一个路径是文件还是文件夹？

通常会配合 `fs.stat` 或 `fs.lstat` 使用。

常见思路：

1. 获取 `stats` 对象。
2. 通过 `stats.isFile()` 判断是否为文件。
3. 通过 `stats.isDirectory()` 判断是否为目录。

### 说说对 Node 中 Buffer 的理解？应用场景是什么？

`Buffer` 是 Node 用来处理二进制数据的核心对象。

因为网络传输、文件读写、流处理很多场景都不是字符串，而是二进制数据，所以需要 Buffer 来承载原始字节。

常见场景：

1. 文件读写
2. 网络通信
3. 图片、音视频处理
4. 编码转换
5. 流式数据处理

面试时可以补一句：浏览器里常见的是 `ArrayBuffer`，Node 里常见的是 `Buffer`。

### 说说对 Node 中 Stream 的理解？应用场景是什么？

Stream 即流，适合处理大数据量、连续到达的数据，而不是一次性全部加载到内存。

Node 中常见流类型：

1. 可读流 `Readable`
2. 可写流 `Writable`
3. 双工流 `Duplex`
4. 转换流 `Transform`

应用场景：

1. 大文件读写
2. HTTP 请求和响应
3. 文件上传下载
4. 压缩、解压、转码
5. 日志处理

一句话总结：流的价值是边读边处理，减少内存占用，提高传输效率。

### Node 中流分为几类？有哪些应用场景？

Node 中流主要分为四类：

1. `Readable`
2. `Writable`
3. `Duplex`
4. `Transform`

常见应用场景：

1. 文件读取和写入
2. 请求体和响应体处理
3. 数据压缩解压
4. 数据格式转换
5. 实时日志传输

### Node 中如何判断一个对象是不是 stream？

常见思路是结合以下几个特征判断：

1. 是否继承自 Stream 类。
2. 是否具有 `pipe`、`on`、`read`、`write` 等特征方法。
3. 在工程里可使用 `stream` 模块或第三方工具做更稳妥判断。

### 说说 Node 中的 EventEmitter？如何实现一个 EventEmitter？

`EventEmitter` 是 Node 中事件驱动模型的核心实现，用于发布订阅。

常见能力有：

1. `on`
2. `once`
3. `emit`
4. `off` / `removeListener`

实现一个简化版 EventEmitter，本质上就是：

1. 用对象或 `Map` 存储事件名和回调列表。
2. `on` 时把回调放进去。
3. `emit` 时按顺序执行对应回调。
4. `off` 时把指定回调移除。
5. `once` 可以在第一次执行后自动移除。

## 三、模块机制与加载原理

### 说说 Node 文件查找的优先级以及 require 方法的文件查找策略。

`require` 的解析大致遵循以下思路：

1. 先判断是不是内置模块。
2. 如果是相对路径或绝对路径，则按文件路径查找。
3. 若没写后缀，会按规则尝试 `.js`、`.json`、`.node`。
4. 如果目标是目录，会查目录下的 `package.json` 的入口字段，或回退到 `index.js` 等。
5. 如果是第三方模块，会沿着当前目录向上逐级查找 `node_modules`。

面试时要强调：`require` 不只是简单读文件，它背后有一套完整的模块解析策略和缓存机制。

### Node 中 require JSON 文件时，如何在文件更新后重新 require？

因为 `require` 会缓存模块，后续再次 `require` 默认拿到的是缓存结果。

如果想重新加载：

1. 删除 `require.cache` 中对应模块缓存。
2. 再次执行 `require`。

但工程上更常见的是：

1. 直接使用 `fs.readFile` 动态读取。
2. 或者结合配置中心、热更新方案管理配置变更。

### 有没有接触过 fs-extra？它是解决什么问题的？

`fs-extra` 是对原生 `fs` 的增强封装。

它解决的问题主要有：

1. 提供更方便的文件操作方法。
2. 补齐一些原生 `fs` 不够顺手的高级能力。
3. 减少手写目录判断、递归创建、拷贝等样板代码。

常见方法如：

1. `ensureDir`
2. `copy`
3. `remove`
4. `outputFile`

## 四、异步机制与事件循环

### 说说对 Node.js 中的事件循环机制的理解。

Node 的事件循环负责调度同步任务、异步 I/O 回调、定时器和微任务。

和浏览器不同，Node 的事件循环是基于 libuv 实现的，分为多个阶段，比如：

1. timers
2. pending callbacks
3. idle/prepare
4. poll
5. check
6. close callbacks

在这些阶段之间，Node 还会处理微任务队列。

面试时要强调：Node 不是“所有异步都同时执行”，而是在事件循环不同阶段里有序调度。

### node 中 nextTick 与 setTimeout 有什么区别？

两者都能把任务延后执行，但优先级不同。

1. `process.nextTick` 会在当前阶段结束后、事件循环进入下一阶段前优先执行。
2. `setTimeout` 则属于 timers 阶段的宏任务调度。

所以通常 `nextTick` 的执行时机早于 `setTimeout`。

### `process.nextTick`、`setImmediate`、`setTimeout` 的区别是什么？

三者都能把任务延后执行，但调度时机不同。

1. `process.nextTick` 优先级最高，会在当前操作结束后、事件循环继续前先执行。
2. `setTimeout(fn, 0)` 进入 timers 阶段，不代表立刻执行。
3. `setImmediate` 通常进入 check 阶段，常用于 I/O 回调之后尽快执行。

面试时可以总结为：

1. `nextTick` 更像当前阶段尾部插队。
2. `setTimeout` 和 `setImmediate` 都属于事件循环调度，但阶段不同。
3. 在不同上下文里，`setImmediate` 和 `setTimeout` 的先后可能受执行环境影响。

### 请简述下 Node 与浏览器环境中的事件循环。

两者核心思想一致，都是任务队列加事件循环，但实现细节不同。

区别主要在：

1. 浏览器事件循环更关注 UI 渲染、DOM 事件和宏微任务调度。
2. Node 事件循环更关注 I/O 阶段、定时器阶段和 libuv 调度。
3. Node 里还要特别关注 `process.nextTick` 和 `setImmediate` 的执行顺序。

### Node 中服务端框架如何解析 HTTP 请求体 body？

HTTP 请求体本质上是流。

解析思路通常是：

1. 监听请求流的数据片段。
2. 逐块拼接 Buffer。
3. 在 `end` 时得到完整请求体。
4. 根据 `Content-Type` 做 JSON、表单、文件等不同解析。

框架里常见做法是通过中间件统一完成，比如 Koa 的 `koa-bodyparser`、Express 的 `express.json()`。

### 在 Node 中如何发送请求？

常见方式有：

1. 原生 `http` / `https` 模块
2. `fetch`（现代 Node 版本）
3. 第三方库，如 `axios`

实际项目里一般不会手写底层 HTTP 请求细节太多，更常通过统一请求封装来管理超时、重试、日志和鉴权。

## 五、中间件、Koa 与服务端框架

### 说说对中间件概念的理解，如何封装 Node 中间件？

中间件本质上是对请求处理流程的分层包装。

它的价值：

1. 把公共逻辑抽离出来。
2. 形成可插拔的处理链。
3. 便于做日志、鉴权、异常处理、参数校验等通用能力。

一个典型中间件通常会：

1. 接收上下文对象。
2. 决定是否继续调用下一个处理逻辑。
3. 在前后阶段插入统一处理。

### 什么是 Koa 的洋葱模型？

Koa 的洋葱模型指的是中间件执行顺序呈“先进后出”的嵌套结构。

执行过程是：

1. 请求进入时从外到内执行。
2. 遇到 `await next()` 后进入下一层。
3. 内层执行完成后，再从里到外返回。

这种模型很适合做：

1. 日志耗时统计
2. 错误统一处理
3. 响应包装

### 简述 Koa 的中间件原理，手写 koa-compose 的思路是什么？

Koa 中间件原理本质上是把多个函数组合成一个按顺序调用的 Promise 链。

手写 `koa-compose` 的核心思路：

1. 接收一个中间件数组。
2. 返回一个函数，执行时维护当前中间件索引。
3. 每次调用 `dispatch(i)` 执行第 `i` 个中间件。
4. 中间件内部调用 `next()` 时，实际触发 `dispatch(i + 1)`。
5. 通过 Promise 串联整个流程。

## 六、鉴权、上传与常见业务设计

### 如何实现 JWT 鉴权机制？说说你的思路。

JWT 鉴权的核心思路是：服务端签发一个包含用户信息和过期时间的 token，客户端后续请求携带 token，服务端验证签名和有效性。

一般流程：

1. 用户登录成功后签发 JWT。
2. 客户端保存 token。
3. 后续请求在请求头中携带 `Authorization`。
4. 服务端校验 token 是否合法、是否过期。
5. 合法则放行，不合法则拒绝。

面试时要强调：JWT 适合无状态认证，但登出控制、续签、失效管理要单独设计。

### 请简述重新登录 refresh token 的原理。

常见双 token 方案是：

1. 短期有效的 access token
2. 长期有效的 refresh token

流程通常是：

1. access token 过期后，客户端用 refresh token 请求新的 access token。
2. 服务端验证 refresh token 合法性后重新签发。
3. refresh token 本身也可能轮换更新。

它的价值是兼顾安全性和用户体验。

### 如何实现文件上传？说说你的思路。

文件上传的核心是前端通过 `multipart/form-data` 或分片方式把文件传给服务端，服务端再做解析和落盘处理。

常见流程：

1. 前端选择文件。
2. 构造 `FormData` 上传。
3. 服务端解析请求体。
4. 校验文件类型、大小、权限。
5. 保存到本地磁盘、对象存储或 CDN。

如果是大文件，还会进一步设计分片上传和断点续传。

### 如果让你设计一个分页功能，你会怎么设计？前后端如何交互？

分页设计一般有两类：

1. 页码分页：`page` + `pageSize`
2. 游标分页：`cursor` + `limit`

常见接口返回应包含：

1. 当前数据列表
2. 总数或是否还有下一页
3. 当前页码或游标信息

前端要处理：

1. 翻页状态
2. 筛选条件
3. 空状态和加载状态
4. 翻页与筛选联动重置

## 七、性能、进程与稳定性

### Node 性能如何进行监控以及优化？

可以从监控和优化两个维度回答。

监控维度：

1. CPU 使用率
2. 内存使用情况
3. 事件循环延迟
4. GC 次数和耗时
5. 请求耗时、吞吐量、错误率
6. 日志与链路追踪

优化维度：

1. 避免阻塞事件循环。
2. 合理使用缓存。
3. 使用流处理大文件。
4. 优化数据库和外部请求。
5. 必要时用多进程或消息队列分担压力。
6. 用性能分析工具定位热点。

### 在 Node 应用中如何利用多核心 CPU 的优势？

因为 Node 单进程本身主要跑在单核上，所以要利用多核，通常需要多进程或线程方案。

常见方式：

1. `cluster`
2. 多实例部署配合负载均衡
3. `worker_threads` 处理 CPU 密集型任务

### Node 中 cluster 的原理是什么？

`cluster` 的核心思想是主进程 fork 出多个工作进程，让多个进程共同监听同一端口，从而利用多核能力。

它的优点：

1. 提高 CPU 利用率。
2. 提升服务并发处理能力。
3. 某个 worker 异常退出时可以拉起新进程。

不过现代场景里，也常用容器编排或进程管理工具替代直接依赖 cluster。

### Node 如何进行进程间通信？

常见方式有：

1. `process.send` / `message` 事件
2. 管道
3. Socket
4. 消息队列
5. Redis、数据库等共享介质

如果是 Node 主子进程或 cluster 场景，最常见的是内置消息通信机制。

### Node 应用中如何查看 GC 日志？

通常会在启动 Node 时增加 V8 相关参数，比如输出 GC 日志，再结合监控平台或分析工具观察。

这类题面试不一定要求背具体参数，更重要的是知道：

1. GC 日志可以辅助分析内存抖动和泄漏。
2. 要结合堆快照、性能分析和运行指标综合判断。

### 简述 Node/V8 中的垃圾回收机制。

V8 采用分代垃圾回收思想，通常把堆内存分为：

1. 新生代
2. 老生代

常见回收方式：

1. 新生代多用复制算法
2. 老生代多用标记清除和标记整理

核心目标是尽量减少回收停顿，并在性能和内存之间做平衡。

### Node 中循环引用会发生什么？

循环引用本身不一定等于内存泄漏。

在现代垃圾回收机制下，如果一组对象虽然相互引用，但整体已经不可达，仍然可以被回收。

真正的问题在于：

1. 对象仍然可达但不再需要。
2. 闭包、缓存、全局变量等路径把对象一直引用住。

所以排查重点不是“有没有循环引用”，而是“有没有不可释放的强引用链”。

### 有没有使用过 Node 的 inspect 这个核心模块？

`inspector` 模块可以让 Node 进程接入调试和性能分析能力。

常见用途：

1. 远程调试
2. 性能分析
3. Heap 快照
4. CPU Profile

面试时如果没有实际深用过，也可以说明它主要服务于调试和性能诊断场景。

### 如何得知当前 Node 版本对应的 V8 版本号？

可以直接通过 `process.versions` 查看，其中包含 `node` 和 `v8` 等版本信息。

例如常见方式是查看：

1. `process.version`
2. `process.versions.v8`

### node 中 dns.resolve 及 dns.lookup 有什么区别？

两者都和域名解析有关，但侧重点不同。

1. `dns.lookup` 更接近操作系统提供的解析能力，可能走系统 hosts、系统 DNS 配置。
2. `dns.resolve` 更偏直接发起 DNS 查询，返回更接近 DNS 协议层结果。

所以它们的结果来源和行为可能不同。

### 在 node 中如何开启 HTTPS？

常见做法是：

1. 准备证书和私钥。
2. 使用 `https` 模块创建服务。
3. 传入证书配置并监听端口。

实际生产里，很多时候 HTTPS 会终止在 Nginx 或网关层，而不是直接由 Node 应用自己处理。

## 附：常见代码实现

### 1. 读取文件

```javascript
const fs = require('fs')
fs.readFile('./data.txt', 'utf8', (err, data) => {
  if (err) return console.error(err)
  console.log(data)
})
```

### 2. 简易 HTTP 服务

```javascript
const http = require('http')
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json; charset=utf-8' })
  res.end(JSON.stringify({ ok: true }))
})
server.listen(3000)
```

### 3. EventEmitter 示例

```javascript
const EventEmitter = require('events')
const bus = new EventEmitter()
bus.on('done', data => console.log(data))
bus.emit('done', 'finish')
```
