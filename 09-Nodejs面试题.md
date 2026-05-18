[toc]



## 一、Node.js 基础认知

### 说说你对 Node.js 的理解？优缺点和应用场景是什么？

Node.js 是基于 V8 的 JavaScript 运行时，让 JS 跑在服务端。它把 V8（执行 JS）+ libuv（事件循环、I/O 调度）+ Node 自带模块（fs、http、crypto）粘在一起，做出一个不依赖浏览器的 JS 运行环境。

核心特性是**单线程事件循环 + 非阻塞 I/O**。"单线程"指的是跑 JS 代码的线程只有一条，I/O 操作（读文件、网络请求）真正干活的是 libuv 内部的线程池（默认 4 个），JS 主线程只负责扔任务、等回调。这套设计在 I/O 密集场景下吞吐很高——同一时间能扛上万个长连接，因为不是每个连接都开一条线程。

**适合**：

- Web API / BFF（后端聚合层）
- SSR（Next.js / Nuxt）
- 实时通信（WebSocket、聊天、推送）
- CLI 工具、构建工具链、脚本自动化
- 网关、代理、文件服务

**不适合**：

- CPU 密集计算（图像处理、压缩、加解密大文件）——主线程算到一半，整个服务就阻塞。要么扔进 `worker_threads`、要么用原生扩展、要么直接换语言
- 强事务性、强一致性的业务系统（这种 Java / Go 更稳）

工程上常见的痛点有：错误处理（一个未捕获的 Promise rejection 可能导致 Node 进程退出）、内存泄漏（事件监听不解绑、缓存无限增长）、单核利用率（要靠 cluster 或 PM2 起多进程）。

### 说说 Node.js 有哪些全局对象？

容易遇到问题的点是"看似全局，实际是模块局部"。

**实际的全局**（不管在哪个模块都直接能用）：

- `global`（Node 22+ 也支持 `globalThis`）
- `process`：进程对象
- `Buffer`：二进制数据
- `console`、`setTimeout` / `setInterval` / `clearXxx`
- `queueMicrotask`、`setImmediate`、`AbortController`
- ES Module 里还有 `import.meta`

**模块级局部变量**（CommonJS 模块包装时注入的）：

- `__dirname`、`__filename`：当前文件目录 / 路径
- `module`、`exports`、`require`

CommonJS 加载一个模块时，Node 实际是把你的代码包成一个函数：

```js
(function (exports, require, module, __filename, __dirname) {
  // 你的代码
})
```

所以那些"看似全局"的变量本质是这个函数的参数。**ESM 里 `__dirname` 和 `__filename` 不存在**，要用 `import.meta.url` 自己算。

### 说说 node 中 module.exports 与 exports 有什么区别。

模块初始化时 Node 实际做了 `exports = module.exports = {}`，两者一开始指向同一个对象。导出实际看的是 `module.exports`。

```js
// ✅ 给同一个对象加属性，两边都生效
exports.foo = 1
module.exports.bar = 2

// ✅ 替换整个导出对象
module.exports = { foo: 1 }

// ❌ exports 被指向新对象，但 module.exports 还是原来那个空对象
exports = { foo: 1 }   // 模块导出的是 {}，不是你想的 { foo: 1 }
```

记忆点：要整体替换导出，必须改 `module.exports`；只是加几个属性，`exports.xxx = ...` 没问题。

### 说说对 Node 中 process 的理解？有哪些常用属性和方法？

`process` 是 Node 提供的进程对象，用来获取当前进程信息并进行进程级控制。

常见用途：

1. 获取环境变量：`process.env`
2. 获取命令行参数：`process.argv`
3. 获取当前工作目录：`process.cwd()`
4. 退出进程：`process.exit()`
5. 注册进程事件：`beforeExit`、`exit`、`uncaughtException`、`unhandledRejection`
6. 调度微任务：`process.nextTick()`

`process` 之所以重要，是因为它把“代码运行时”和“当前 Node 进程自身状态”连接了起来。
环境变量、命令行参数、退出行为、进程事件、版本信息、微任务调度，这些都属于进程级能力，而 `process` 正是访问这些能力的统一入口。

### Node 中的标准输入、标准输出、标准错误是什么？

Node 进程启动后默认会有三条标准流：

1. `process.stdin`：标准输入
2. `process.stdout`：标准输出
3. `process.stderr`：标准错误输出

它们的作用分别是：

1. 从命令行或管道读取输入。
2. 向终端输出正常结果。
3. 向终端输出错误信息。

所以不少 Node 脚本、CLI 工具能够自然地和终端、管道、重定向配合工作。

### 说说 node 中 module.exports 与 exports 有什么区别。

两者都和 CommonJS 模块导出有关，但本质不同。

1. `module.exports` 才是真正导出的对象。
2. `exports` 只是 `module.exports` 的一个引用。

所以：

1. 给 `exports.xxx = ...` 赋值，等同于在给 `module.exports` 添加属性。
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

项目里更推荐优先用异步 API，避免阻塞事件循环。

### 如何监控文件的变动？

常用方式有：

1. `fs.watch`
2. `fs.watchFile`
3. 使用更稳定的第三方库，比如 `chokidar`

如果是工程场景，如构建工具、热更新、日志监听，通常更推荐成熟库，因为不同平台下原生监听行为可能有差异。

### Node 中如何判断一个路径是文件还是文件夹？

一般会配合 `fs.stat` 或 `fs.lstat` 使用。

常见思路：

1. 获取 `stats` 对象。
2. 通过 `stats.isFile()` 判断是否为文件。
3. 通过 `stats.isDirectory()` 判断是否为目录。

### 说说对 Node 中 Buffer 的理解？应用场景是什么？

`Buffer` 是 Node 用来处理二进制数据的核心对象。

因为网络传输、文件读写、流处理不少场景都不是字符串，而是二进制数据，所以需要 Buffer 来承载原始字节。

适合用在：

1. 文件读写
2. 网络通信
3. 图片、音视频处理
4. 编码转换
5. 流式数据处理

这两个概念经常一起出现，是因为它们都在处理二进制数据，但定位不同：

1. 浏览器世界更底层、标准化的对象通常是 `ArrayBuffer`
2. Node 为了更方便处理 I/O、网络、文件流，又在它之上提供了更贴近服务端开发的 `Buffer`

### 说说对 Node 中 Stream 的理解？应用场景是什么？

Stream 即流，适合处理大数据量、连续到达的数据，而不是一次性全部加载到内存。

Node 中常见流类型：

1. 可读流 `Readable`
2. 可写流 `Writable`
3. 双工流 `Duplex`
4. 转换流 `Transform`

用在哪：

1. 大文件读写
2. HTTP 请求和响应
3. 文件上传下载
4. 压缩、解压、转码
5. 日志处理

流之所以重要，不只是 API 形式不同，而是它改变了处理大数据的方式。
如果不用流，往往意味着先把完整数据全部读进内存再处理；而流允许你边接收、边处理、边输出，这会直接减少内存峰值，也更适合网络传输和大文件场景。

### Node 中流分为几类？有哪些应用场景？

Node 中流主要分为四类：

1. `Readable`
2. `Writable`
3. `Duplex`
4. `Transform`

常用场景：

1. 文件读取和写入
2. 请求体和响应体处理
3. 数据压缩解压
4. 数据格式转换
5. 实时日志传输

### 什么是背压（Backpressure）？为什么流处理里要关心它？

背压指的是“上游生产数据的速度”大于“下游消费数据的速度”。

如果不处理，就会出现：

1. 内存不断堆积
2. 写入缓冲区越来越大
3. 整个进程吞吐变差甚至抖动

Node 流之所以适合大数据处理，就是因为它内建了背压控制机制。

例如可写流的 `write()` 返回 `false` 时，意味着下游暂时吃不动了；这时上游应该暂停继续灌数据，等 `drain` 事件后再恢复。

### `stream.pipeline` 是做什么的？为什么比手动 pipe 更稳？

`pipeline` 的作用是把多个流安全地串起来，并统一处理错误和关闭逻辑。

它比简单手写 `readable.pipe(writable)` 更稳，原因在于：

1. 多段流出错时，`pipeline` 会统一清理
2. 可以避免部分流出错后其余流还悬挂不结束
3. 更适合生产环境的文件传输、压缩解压、代理转发链路

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

实现一个简化版 EventEmitter，主要包括：

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

这题重点不在背后缀名顺序，而在于说明 `require` 是一套完整的模块加载系统。
它既要做路径解析、目录入口判断、内置模块优先级处理，也要做模块执行和缓存复用，所以它远不只是“去磁盘把文件内容读出来”。

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

### CommonJS 和 ESM 在 Node 中有什么区别？

这种问题现在出现得越来越频繁。

**CommonJS：**

1. 使用 `require` / `module.exports`
2. 加载更偏运行时
3. 历史上是 Node 默认模块体系

**ESM：**

1. 使用 `import` / `export`
2. 导入导出关系更适合静态分析
3. 更贴近浏览器和现代构建生态

它重要的地方在于：

因为 Node 现在同时支持两套模块体系，而两者在下面这些方面都可能有差异：

1. 加载时机
2. 默认导出行为
3. 路径解析方式
4. `__dirname`、`__filename` 可用性
5. 与工具链的兼容方式

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

这里容易被误解的地方是：Node 异步不少，不代表所有任务都会无序并发一起跑。
真实情况是，I/O 完成、定时器到期、`setImmediate`、`nextTick`、Promise 回调等任务，都会被放到事件循环的不同阶段或不同优先级队列里，再由运行时按规则调度。

### node 中 nextTick 与 setTimeout 有什么区别？

并到下题一起讲。

### `process.nextTick`、`setImmediate`、`setTimeout` 的区别是什么？

三个都是延后执行，但落在事件循环的不同位置。

Node 的事件循环每一轮按阶段走：

```
┌──────────┐
│  timers  │ ← setTimeout、setInterval 到期回调
├──────────┤
│ pending  │
├──────────┤
│   idle   │
├──────────┤
│   poll   │ ← I/O 回调（绝大多数业务回调在这里）
├──────────┤
│  check   │ ← setImmediate 回调
├──────────┤
│  close   │ ← close 事件
└──────────┘
```

三者具体位置：

- **`process.nextTick(fn)`**：跳出阶段切换的"插队队列"——当前操作做完、事件循环准备进下个阶段前，把所有 nextTick 跑光。优先级最高，**滥用会把事件循环饿死**。
- **`setTimeout(fn, 0)`**：进 timers 阶段。注意 `delay=0` 实际不会真的 0ms，最小被钳到 1ms。
- **`setImmediate(fn)`**：进 check 阶段，在 poll 阶段（I/O 回调）之后跑。

谁先谁后？

- 在 I/O 回调里：**`setImmediate` 一定先于 `setTimeout`**（poll → check，timers 要等下一轮）
- 在主模块顶层：不一定，看启动时机，可能 setTimeout 先也可能 setImmediate 先
- nextTick 永远比 setImmediate、setTimeout 早（因为它根本不进入阶段队列）

微任务（Promise.then）的位置：Node 11+ 已对齐浏览器，**每个回调结束后清空微任务**，比 setImmediate / setTimeout 早。

```js
setImmediate(() => console.log('immediate'))
setTimeout(() => console.log('timeout'), 0)
Promise.resolve().then(() => console.log('promise'))
process.nextTick(() => console.log('nextTick'))

// 输出顺序：nextTick → promise → timeout → immediate
// （主模块顶层，setTimeout 可能先于 setImmediate）
```

### 请简述下 Node 与浏览器环境中的事件循环。

核心模型一样（同步代码 → 微任务 → 宏任务 → 渲染），但 Node 的"宏任务"被进一步拆成 timers / poll / check / close 等阶段，每个阶段处理一类回调。差异点：

| 维度 | 浏览器 | Node |
|------|--------|------|
| 阶段划分 | 一种宏任务队列 | 多阶段（timers/poll/check/close）|
| 微任务清空时机 | 每个宏任务后 | Node 11+ 同左 |
| 额外的高优先级队列 | 无 | `process.nextTick`（比微任务更早）|
| 渲染 | 每帧合适时机渲染 | 无渲染 |
| `setImmediate` | 无 | 有，独占 check 阶段 |

实际写代码时不用每天纠结这些差异——除非你在写库代码、写性能敏感的中间件，或者要保证某段逻辑在 I/O 回调之后立即执行（此时选 setImmediate）。日常业务用 `Promise.then` 和 `setTimeout` 足够。

### `worker_threads` 是什么？和 cluster 有什么区别？

这个问题适合用来区分“多线程”和“多进程”。

**worker_threads：**

1. 同一个进程内开启多个线程
2. 更适合 CPU 密集型任务拆分
3. 可通过消息传递或共享内存通信

**cluster：**

1. 实际上是多个独立进程
2. 更常用于服务多实例并发处理
3. 更适合利用多核承载 HTTP 服务

所以二者解决的问题并不完全一样：

1. `worker_threads` 更像“把重计算拆出去”
2. `cluster` 更像“把服务实例横向展开”

### Node 中服务端框架如何解析 HTTP 请求体 body？

HTTP 请求体本身就是流。

解析思路通常是：

1. 监听请求流的数据片段。
2. 逐块拼接 Buffer。
3. 在 `end` 时得到完整请求体。
4. 根据 `Content-Type` 做 JSON、表单、文件等不同解析。

框架里一般交给中间件处理，比如 Koa 的 `koa-bodyparser`、Express 的 `express.json()`。

### 在 Node 中如何发送请求？

常见方式有：

1. 原生 `http` / `https` 模块
2. `fetch`（现代 Node 版本）
3. 第三方库，如 `axios`

项目里多半不会手写底层 HTTP 请求细节太多，更常通过统一请求封装来管理超时、重试、日志和鉴权。

## 五、中间件、Koa 与服务端框架

### 说说对中间件概念的理解，如何封装 Node 中间件？

中间件就是对请求处理流程做分层包装。

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

Koa 中间件原理是把多个函数组合成一个按顺序调用的 Promise 链。

手写 `koa-compose`` 的做法一般是：

1. 接收一个中间件数组。
2. 返回一个函数，执行时维护当前中间件索引。
3. 每次调用 `dispatch(i)` 执行第 `i` 个中间件。
4. 中间件内部调用 `next()` 时，实际触发 `dispatch(i + 1)`。
5. 通过 Promise 串联整个流程。

### Express 和 Koa 的中间件模型有什么区别？

这里关键不在于背框架 API，而在于理解两者对异步控制流的设计差异。

**Express：**

1. 更早期的中间件模型
2. 常见形态是 `req, res, next`
3. 历史上对回调风格更友好

**Koa：**

1. 更现代，基于 Promise / async-await
2. 常见形态是 `ctx, next`
3. 更强调洋葱模型和异步控制链

为什么不少人觉得 Koa 中间件“更优雅”？

因为在 `async/await` 语境下，Koa 的“先进去、再回来”的控制流更自然，更容易写出前后置逻辑，比如统一日志、耗时统计、异常处理。

## 六、鉴权、上传与常见业务设计

### 如何实现 JWT 鉴权机制？说说你的思路。

JWT 鉴权一般就是服务端签发一个带用户信息和过期时间的 token，客户端后续请求携带它，服务端再去校验签名和有效性。

一般流程：

1. 用户登录成功后签发 JWT。
2. 客户端保存 token。
3. 后续请求在请求头中携带 `Authorization`。
4. 服务端校验 token 是否合法、是否过期。
5. 合法则放行，不合法则拒绝。

JWT 之所以常被问，不是因为它“先进”，而是因为它把认证信息放进了自包含 token 里。
这样做的好处是服务端不一定要查 session 存储，但代价也明显：一旦 token 已经发出去，登出、强制失效、细粒度吊销就不能再只靠“删服务端 session”来解决，所以续签和失效策略必须额外设计。

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

排查流程：

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
2. 适当使用缓存。
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

`cluster` 的思路是主进程 fork 出多个工作进程，让多个进程共同监听同一端口，这样可以利用多核能力。

它的好处：

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

### `child_process` 和 `worker_threads` 的区别是什么？

`child_process` 是起一个新的进程，`worker_threads` 是在同一个进程里开工作线程。

差异主要在：

1. 进程隔离更强，但开销更大
2. 线程通信更轻，但共享资源和调度更复杂

所以命令执行、独立进程工具链常用 `child_process`，而 CPU 重计算拆分更适合 `worker_threads`。

### 为什么 Node 服务经常会提到“不要阻塞事件循环”？

因为 Node 单个进程里，主线程不仅跑你的业务代码，还负责调度大量 I/O 回调。

如果你在这条线程上长时间做同步计算，就会导致：

1. 请求无法及时响应
2. 定时器延迟
3. 事件循环卡顿

### 为什么不少 Node 服务会特别关注内存泄漏？

因为 Node 服务往往是常驻进程，一旦有泄漏，不是“页面关掉就结束”，而是会随着时间持续累积。

而且服务端对象生命周期往往比前端页面更长，缓存、闭包、全局变量、监听器残留更容易把问题放大。

如果是 Node 主子进程或 cluster 场景，常见的是内置消息通信机制。

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

实际的问题在于：

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

`inspector` 这类能力重点不在于是否手写过 API，而在于理解它的定位：
可以直接看成 Node 进程暴露给调试器和性能分析工具的一层接口，用来做断点、CPU Profile、Heap 快照和运行时诊断，所以它主要服务于调试和性能分析，不是日常业务逻辑。

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

一般这样处理：

1. 准备证书和私钥。
2. 使用 `https` 模块创建服务。
3. 传入证书配置并监听端口。

实际生产里，不少时候 HTTPS 会终止在 Nginx 或网关层，而不是直接由 Node 应用自己处理。

## 八、Node 进阶与实战

### Node 20+ 常用的新能力

不少新特性在 release notes 里很显眼，但项目中高频使用的通常只有少数几项。

**内置 test runner（`node:test`）**：20 起稳定。日常单测可以不再装 Jest/Vitest。

```js
// src/utils.test.js
import { test } from 'node:test'
import assert from 'node:assert/strict'
import { sum } from './utils.js'

test('sum 基础用例', () => {
  assert.equal(sum(1, 2), 3)
})

test('异步用例', async (t) => {
  await t.test('子测试', async () => {
    const r = await fetchData()
    assert.ok(r.ok)
  })
})

// 跑:node --test src/**/*.test.js
```

适合：脚本、内部工具、轻量库。重型项目（要 jsdom、要快照、要 mock 一切）还是用 Vitest。

**`--watch` 模式**：开发时不用再装 nodemon。`node --watch server.js` 改文件自动重启。

**`--env-file=.env`**：原生支持 .env 文件加载，不用再 require('dotenv')。

```bash
node --env-file=.env --env-file=.env.local server.js
```

**permission model**（实验性）：能限制 Node 进程只能读特定目录、不能 spawn 子进程。

```bash
node --experimental-permission --allow-fs-read=./data --allow-fs-write=./logs server.js
```

对运行不信任脚本（用户提交的代码、构建插件）有用，生产环境还在 RC。

**`fetch` 全局可用**：从 18 开始内置，20 稳定。不用再装 axios / node-fetch（除非你要 axios 的拦截器、超时控制这些进阶功能）。

```js
const res = await fetch('https://api.example.com')
const data = await res.json()
```

注意：内置 fetch 默认**没有超时**——`AbortController` 自己加：

```js
const ctrl = new AbortController()
const timer = setTimeout(() => ctrl.abort(), 5000)
try {
  const res = await fetch(url, { signal: ctrl.signal })
} finally {
  clearTimeout(timer)
}
```

**Web Streams 标准化**：跟浏览器的 Stream API 对齐，可以这样写：

```js
const res = await fetch(url)
for await (const chunk of res.body) {   // body 是 ReadableStream
  console.log(chunk)
}
```

但 Node Streams 跟 Web Streams 是两套 API，互转用 `Readable.fromWeb()` / `Readable.toWeb()`。

### Bun / Deno 跟 Node 的差异，能上生产了吗？

结论：Bun 在「快」和「内置工具链」上确实领先，Deno 在「安全 + Web 标准」上有理念优势，但**生产环境的主流仍是 Node**。

**Bun**：

- 启动速度：实测比 Node 快 2-5 倍（启动一个简单 server，Node 80ms vs Bun 20ms）
- 内置工具：bundler、test runner、package manager 全在一个二进制里
- `Bun.serve()` 性能比 Node http 模块高约 3 倍（基于 uWebSockets）
- 兼容性：99% Node API 已实现，npm 包大部分能跑
- 风险：单一公司（Oven）维护，社区比 Node 小得多，生产事故没那么多前车之鉴

**Deno**：

- 默认沙箱：脚本必须显式 `--allow-net --allow-read` 才能访问网络/文件
- TypeScript 一等公民，不用编译直接运行
- 用 Web 标准 API（fetch、Web Streams、URL）作为内置库
- 2.0 起增加了 npm 兼容（`npm:react` 这种 import），但仍有边界 case
- 风险：生态库少（Deno-native 库远少于 npm），切过去等于换语言

**生产现状**：

- **Node**：大公司后端、Next.js / Nuxt 服务端、所有主流 BFF 都是它
- **Bun**：开发态用得多（启动快、装包快），生产用得少（Cloudflare Workers 类 serverless 例外）
- **Deno**：Supabase、Vercel Edge Runtime 用了 Deno 内核，但应用代码层直接选择 Deno 的场景较少

**使用建议**：**新项目可以用 Bun 跑脚本和本地开发；生产 server 仍优先选择 Node**。等 Bun 继续稳定并积累更多生产案例后，再评估是否用于核心服务。

### Node 项目从 CJS 迁到 ESM 实战路径

CJS → ESM 是很多项目需要面对的迁移，不少老项目会在这里遇到阻力。完整路径：

**第一步：确定迁移范围**

新建项目直接上 ESM 没争议。老项目要看：

- 用了哪些只发 CJS 的依赖（如 `winston@2`、`mongoose@5`）
- 业务代码里有多少 `require` / `module.exports`
- 构建工具链（TS、babel、jest）的 ESM 支持

**第二步：改 `package.json`**

```jsonc
{
  "type": "module",          // 关键
  "main": "./dist/index.js",
  "exports": {
    ".": "./dist/index.js"
  },
  "engines": {
    "node": ">=18.0.0"       // ESM 需要的最低版本
  }
}
```

**第三步：改导入导出语法**

```js
// 之前
const fs = require('fs')
const { sum } = require('./utils')
module.exports = { foo: 1 }

// 之后
import fs from 'fs'
import { sum } from './utils.js'   // ESM 里 .js 后缀不能省
export const foo = 1
```

**第四步：处理 `__dirname` / `__filename`**

ESM 里没这俩，自己算：

```js
import { fileURLToPath } from 'url'
import { dirname } from 'path'

const __filename = fileURLToPath(import.meta.url)
const __dirname = dirname(__filename)

// Node 20.11+ 直接用 import.meta.dirname
console.log(import.meta.dirname)
```

**第五步：处理 CJS-only 依赖**

```js
// CJS 包 ESM 项目里用：default import
import lodash from 'lodash'   // ✅ 拿到 module.exports

// 命名导出可能拿不到（看包导出方式）
import { debounce } from 'lodash'   // 有的能用、有的报错

// 兜底用 createRequire
import { createRequire } from 'module'
const require = createRequire(import.meta.url)
const oldLib = require('old-cjs-lib')
```

**第六步：循环依赖在 ESM 下行为变了**

CJS 的循环依赖会得到「部分加载的对象」，能跑但容易出诡异 bug。ESM 的循环依赖是「绑定式 live binding」——拿到的是变量的引用，到使用时再求值，反而更可控。但如果你的代码本来依赖 CJS 的「先打半个对象」行为，迁过来会爆。

**第七步：测试 / 构建工具适配**

- Jest：要么用 `--experimental-vm-modules`，要么换 Vitest
- TS：`module: NodeNext`，`moduleResolution: NodeNext`
- ts-node：换 `tsx` 或 `--loader ts-node/esm`

迁移策略上要说的就这点：能整体一次性迁就一次性迁，不要 dual package。维护两套成本很高。

### Node 性能调优实战三件套：CPU profile、Heap snapshot、--prof

线上响应变慢、内存涨、CPU 飙高，这三个工具组合能定位 90% 的问题。

**CPU profile（`--inspect` + Chrome DevTools）**：

```bash
# 启动服务
node --inspect=0.0.0.0:9229 server.js

# Chrome 打开 chrome://inspect,连接到 9229
# Performance 面板里 Record CPU profile,跑一段业务流量,停下来分析火焰图
```

火焰图里横向越宽的函数占 CPU 越多。常见瓶颈：JSON.parse 大 body、同步加密、深层对象遍历、正则灾难（catastrophic backtracking）。

**`--prof` + `--prof-process`（不带 inspect）**：

```bash
# 跑一会儿,生成 isolate-xxx-v8.log
node --prof server.js

# 解析成可读形式
node --prof-process isolate-xxx-v8.log > profile.txt
```

输出按函数累计 tick 排序，看 LazyCompile 段的 top 几个即可。这个方式不需要连 DevTools，适合生产环境短时间采样。

**Heap snapshot（定位内存泄漏）**：

```bash
# 启动 + 连 inspect
node --inspect server.js

# DevTools Memory 面板,Take heap snapshot
# 跑业务,过一会儿再 take 一次
# Compare 两次快照,看 "delta" 列谁涨得多
```

定位泄漏的关键看「retained size」——一个对象被回收能释放多少内存。常见泄漏模式：

- Map / Set 无限增长（用 LRU 替代）
- 事件监听器没 off（addListener 必须配对 removeListener）
- 闭包捕获了大对象（function 内部引用了 outer 的大 array）
- timer 没 clear（setInterval 不停跑）
- 全局缓存只增不减

**线上抓 heap snapshot 的方式**（不能停服务）：

```js
import { writeHeapSnapshot } from 'v8'

// HTTP endpoint 触发,或者 SIGUSR2 信号触发
process.on('SIGUSR2', () => {
  const file = `./heap-${Date.now()}.heapsnapshot`
  writeHeapSnapshot(file)
  console.log(`heap snapshot saved to ${file}`)
})

// kill -SIGUSR2 <pid> 触发
```

文件下载下来用 DevTools 加载分析。

**关键 V8 flags**：

```bash
# 调整老生代大小（默认约 1.5GB，超了会 OOM）
node --max-old-space-size=4096 server.js   # 4GB

# 调试 GC 行为
node --trace-gc server.js                  # 输出每次 GC 时间和回收量

# 调小新生代大小（小对象多的场景 GC 太频繁时）
node --max-semi-space-size=64 server.js
```

### Worker Threads 实际场景与问题

常见误解是 `worker_threads` 是「Node 终于支持多线程了」，实际它是个**特定场景**才该用的东西。

**真正适合 worker_threads 的场景**：

- CPU 密集计算（图像处理、压缩、加解密大文件、AI 推理 ONNX 模型）
- 长时间同步任务，不想阻塞主线程
- 解析超大 JSON / CSV

**不适合的**：

- 普通 HTTP 服务（用 cluster 横向扩展更好）
- I/O 密集（数据库查询、文件读写）——Node 主线程 + libuv 线程池已经够用
- 简单业务逻辑——通信开销可能比单线程跑还大

**最小示例**：

```js
// main.js
import { Worker } from 'worker_threads'
import { fileURLToPath } from 'url'

const worker = new Worker(fileURLToPath(new URL('./worker.js', import.meta.url)), {
  workerData: { input: largeArray }
})

worker.on('message', (result) => {
  console.log('done:', result)
})

worker.on('error', (err) => console.error(err))

// worker.js
import { parentPort, workerData } from 'worker_threads'

const result = heavyCompute(workerData.input)
parentPort.postMessage(result)
```

**通信成本是大问题**：

- `postMessage` 默认是 structured clone——大对象会被深拷贝，几百 MB 数据传一次能卡半秒
- 用 `transferList` 把 ArrayBuffer 转移（零拷贝），但转移后原线程就拿不到了
- 用 `SharedArrayBuffer` + `Atomics` 共享内存，但要小心数据竞争

**Worker Pool 模式**：每次新建 worker 也要时间（约 50-100ms），高频任务要建池子复用：

```js
import workerpool from 'workerpool'

const pool = workerpool.pool('./worker.js', { maxWorkers: 4 })

const result = await pool.exec('heavyCompute', [input])
```

**`worker_threads` vs `cluster` vs `child_process`**：

| 维度 | worker_threads | cluster | child_process |
|------|----------------|---------|---------------|
| 隔离级别 | 共进程，独立 V8 实例 | 独立进程 | 独立进程 |
| 内存共享 | 可（SharedArrayBuffer） | 不行 | 不行 |
| 通信成本 | 中（同进程消息） | 高（IPC） | 高（IPC） |
| 启动成本 | 中 | 高 | 高 |
| 适用 | CPU 计算 | HTTP 横向扩展 | 跑外部命令 / 隔离任务 |

### Node 服务的稳定性建设：优雅停机、错误兜底

线上事故里不少是「停机时丢了正在处理的请求」「未捕获错误导致进程退出」这种。稳定性建设就是补齐这些风险点。

**优雅停机（graceful shutdown）**：

```js
import http from 'http'

const server = http.createServer(handler)
server.listen(3000)

let isShuttingDown = false

function shutdown(signal) {
  console.log(`收到 ${signal}，开始优雅停机`)
  isShuttingDown = true

  // 1. 停止接收新请求
  server.close(async () => {
    // 2. 等正在处理的请求完成
    console.log('HTTP 服务已关闭')

    // 3. 关闭数据库连接、消息队列
    await db.close()
    await mq.close()

    // 4. 退出
    process.exit(0)
  })

  // 兜底：30 秒还没关闭就强杀
  setTimeout(() => {
    console.error('优雅停机超时，强制退出')
    process.exit(1)
  }, 30000)
}

// 健康检查接口要响应 503，让 LB 把流量切走
app.use((req, res, next) => {
  if (isShuttingDown && req.path === '/health') {
    return res.status(503).end()
  }
  next()
})

process.on('SIGTERM', () => shutdown('SIGTERM'))
process.on('SIGINT', () => shutdown('SIGINT'))
```

**uncaughtException / unhandledRejection 的正确处理**：

```js
process.on('uncaughtException', (err) => {
  console.error('FATAL: uncaughtException', err)
  // 重要：不要忽略异常继续跑
  // 进程状态已不可知,记日志后退出,让 supervisor 重启
  shutdown('uncaughtException')
})

process.on('unhandledRejection', (reason) => {
  console.error('FATAL: unhandledRejection', reason)
  shutdown('unhandledRejection')
})
```

常见做法是「捕获异常后继续运行」，这种处理不可靠——异常之后进程内部状态可能已坏，继续运行可能带来更隐蔽的问题。记录日志后退出进程，再由 PM2 或 k8s 拉起新进程，稳定性更可控。

**限流 + 熔断**：

- 限流：`@fastify/rate-limit`、`express-rate-limit`，按 IP / 用户 / 接口分别限
- 熔断：`opossum`（基于 Hystrix 模式），下游服务异常时要避免故障向当前服务扩散

**健康检查接口**：

```js
app.get('/health', async (req, res) => {
  try {
    await db.ping()
    await redis.ping()
    res.json({ status: 'ok', uptime: process.uptime() })
  } catch (e) {
    res.status(503).json({ status: 'unhealthy', error: e.message })
  }
})
```

**进程管理**：

- 单机：PM2（带集群、重启、日志）
- k8s：直接用 Deployment + liveness/readiness probe
- 不推荐 nodemon 上生产（dev 用 nodemon，prod 用 PM2 或容器）

### Node 真的是单线程吗？

高频问题，但回答容易不完整。完整答案应该是「主 JS 线程是单线程，但 Node 进程整体是多线程的」。

**Node 进程里的线程**：

1. **主线程**：跑 JS 代码、跑事件循环
2. **libuv 线程池**：4 个（默认，可调 `UV_THREADPOOL_SIZE`）
   - 文件 I/O（fs 的异步 API）
   - DNS 查询（dns.lookup）
   - 一些加密操作（crypto.pbkdf2）
   - 用户的 `worker_threads` 也走这里
3. **V8 worker 线程**：GC、JIT 优化、Inspector 通信
4. **如果你用了 worker_threads**：每个 worker 是独立的 V8 isolate（但仍在同一进程内）

**「单线程」指的是 JS 执行的那条主线程**——所有 setTimeout、Promise.then、HTTP 回调、setState 都在这条线程上排队跑。但 IO 操作真正干活的是 libuv 的线程池或操作系统的 async IO。

为什么不少人有「Node 是真单线程」的错觉？因为 JS 代码视角看不到那些底层线程，所有回调都串行在主线程上跑。

**实战影响**：

- 主线程上跑同步重计算 → 整个 Node 阻塞（包括所有 HTTP 请求都不响应）
- 主线程上跑大量 I/O → 线程池可能被占满，调大 `UV_THREADPOOL_SIZE=16` 缓解
- 想真正利用多核 → 用 cluster 起多进程，或者用 worker_threads

```bash
# 调大 libuv 线程池
UV_THREADPOOL_SIZE=16 node server.js
```

这个参数对「大量同步 fs 操作」「dns.lookup 密集」的应用提升明显，普通业务用默认 4 个即可。

### AsyncLocalStorage 是干什么的？为什么所有大厂 Node 服务都在用？

后端代码里有个常见诉求：在请求处理链路上的任意位置，都能拿到「当前请求的 traceId / userId / locale」这种上下文，但又不想每个函数都加参数透传。

Java 里这类问题可用 `ThreadLocal` 解决——每个线程一个独立存储。Node 单线程模型下没法直接用线程局部，得用 AsyncLocalStorage。

怎么用：

```js
import { AsyncLocalStorage } from 'async_hooks'

const requestContext = new AsyncLocalStorage()

// Express 中间件
app.use((req, res, next) => {
  const ctx = {
    traceId: req.headers['x-trace-id'] || crypto.randomUUID(),
    userId: req.user?.id,
    startTime: Date.now(),
  }
  requestContext.run(ctx, () => next())
})

// 任意位置(同步、异步、Promise 都行)拿到 ctx
function logSomething(message) {
  const ctx = requestContext.getStore()
  console.log(`[${ctx?.traceId}] [${ctx?.userId}] ${message}`)
}

async function someBusinessLogic() {
  await db.query(...)        // 异步操作
  logSomething('done')        // 仍然能拿到 ctx
}
```

它的魔法在哪？AsyncLocalStorage 利用 Node 的 `async_hooks` 监听异步资源的创建（promise、setTimeout、fs callback），把当前的 store 跟着传递下去。所以 await 之后、setTimeout 之后、setImmediate 之后，`getStore()` 仍然能拿到外层 `run` 时设的 ctx。

**实际使用场景**：

- **链路追踪**：traceId 全局可读，日志、metrics、上游调用自动带上
- **多租户**：每个请求带上 tenantId，DB 查询自动加 where tenant_id = ?
- **i18n**：当前请求的 locale 自动作用到所有翻译调用
- **权限**：当前用户身份在业务代码里随用随取

**性能成本**：

24 年前 AsyncLocalStorage 是公认的「能用但有点慢」——开起来每个 async 操作都有几个微秒的额外开销，benchmark 里能看出 5-10% 的吞吐下降。Node 21 起 V8 给它专门做了优化，现在开销可以忽略。生产 Node 服务全面用是没问题的。

**Express / Fastify 的 ALS 集成**：

Fastify 5 起内置了 `request.context`，背后就是 AsyncLocalStorage。Express 用 `express-http-context` 或自己包中间件。Koa 因为 ctx 本来就贯穿请求，老项目不一定要上 ALS，但跨多层 service 调用时 ALS 仍然更方便。

**问题**：

- **不要存大对象**——store 引用着，整个请求生命周期都不会 GC
- **`run` 是同步的**，但内部能起异步任务。回调里 await 一万年都拿得到 store
- 老版本（< 17）有内存泄漏 bug，要么升 Node，要么避免高频 run

### Cluster + sticky session 怎么协调？

Cluster 起多 worker 是 Node 利用多核的标准做法，但有个隐藏问题：**WebSocket / Server-Sent Events 需要 sticky session**。

普通 HTTP 请求每次都是独立的，cluster 内置的轮询调度（round-robin）哪个 worker 都行。但 WebSocket 建立连接后是「长连接」，后续帧必须发到同一个 worker——否则那个 worker 没握过手，连接状态对不上。

**问题表现**：

```js
// master.js
import cluster from 'cluster'
import os from 'os'

if (cluster.isPrimary) {
  for (let i = 0; i < os.cpus().length; i++) cluster.fork()
} else {
  // worker.js 起 WebSocket server
  // 客户端连接,握手帧到 worker 1
  // 后续 message 帧可能到 worker 2(因为 cluster round-robin)
  // worker 2 不认识这个连接,直接断开
}
```

**解决方案一：Sticky-session by IP hash**

不用 cluster 默认调度，自己写 master：

```js
import cluster from 'cluster'
import net from 'net'
import os from 'os'

if (cluster.isPrimary) {
  const workers = []
  for (let i = 0; i < os.cpus().length; i++) {
    workers.push(cluster.fork())
  }

  const server = net.createServer({ pauseOnConnect: true }, (conn) => {
    // 按客户端 IP hash 选 worker
    const ip = conn.remoteAddress
    const hash = ip.split('.').reduce((s, n) => s + parseInt(n), 0)
    const worker = workers[hash % workers.length]
    worker.send('sticky:connection', conn)
  })
  server.listen(3000)
} else {
  // worker 接收 master 转过来的 connection
  process.on('message', (msg, conn) => {
    if (msg === 'sticky:connection') {
      myHttpServer.emit('connection', conn)
    }
  })
}
```

社区库 `sticky-session` 把这套封装了。

**方案二：反向代理层做 sticky**

更常见。Nginx / HAProxy 配 IP hash 调度：

```nginx
upstream backend {
  ip_hash;
  server 127.0.0.1:3000;
  server 127.0.0.1:3001;
  server 127.0.0.1:3002;
  server 127.0.0.1:3003;
}
```

每个 worker 监听不同端口，Nginx 按 IP 路由到固定 worker。优点是不用改 Node 代码、能用 Nginx 的完整健康检查 / 重试机制。

**方案三：完全无状态化（推荐）**

最干净的方案：把 WebSocket 连接状态放到外部存储（Redis），任何 worker 都能处理任意连接。Socket.io 4+ 的 `@socket.io/redis-adapter` 就是这个思路。

```js
import { Server } from 'socket.io'
import { createAdapter } from '@socket.io/redis-adapter'
import { createClient } from 'redis'

const io = new Server()
const pubClient = createClient({ url: 'redis://localhost:6379' })
const subClient = pubClient.duplicate()
await Promise.all([pubClient.connect(), subClient.connect()])

io.adapter(createAdapter(pubClient, subClient))
```

任何 worker 都能给某个 socket 发消息，redis 负责跨 worker 路由。这是云原生时代的标准方案——k8s 起多 pod 时各 pod 之间没法 IP hash 路由，必须走 Redis 这种共享层。

### Node 进程间通信的几种方式怎么选？

业务里「让两个 Node 进程通信」的场景比想象多：主进程跟 worker、CLI 跟后台服务、不同 docker 容器之间。每种方式适合的场景不同。

**1. `child_process.fork()` 内置 IPC**

只能用于父子进程，是最方便的：

```js
// parent.js
import { fork } from 'child_process'
const child = fork('./worker.js')
child.send({ task: 'compute', input: 100 })
child.on('message', (result) => console.log('done:', result))

// worker.js
process.on('message', async (msg) => {
  const result = await compute(msg.input)
  process.send(result)
})
```

底层用的是 pipe + JSON 序列化。简单可靠，缺点是只能父子用、只能传可序列化的东西。

**2. `worker_threads` 的 MessagePort**

线程间通信，比进程通信轻不少：

```js
// main.js
import { Worker } from 'worker_threads'
const worker = new Worker('./worker.js')
worker.postMessage({ task: 'compute' })
worker.on('message', (r) => console.log(r))

// worker.js
import { parentPort } from 'worker_threads'
parentPort.on('message', (msg) => {
  parentPort.postMessage(compute(msg))
})
```

支持 transferList（传 ArrayBuffer 不拷贝）和 SharedArrayBuffer（共享内存）。同进程内通信首选。

**3. Unix domain socket / Named pipe**

跨进程但不跨机器，性能极高（不走 TCP/IP 栈）：

```js
import net from 'net'

// server
const server = net.createServer((conn) => {
  conn.on('data', (data) => conn.write(`echo: ${data}`))
})
server.listen('/tmp/myapp.sock')

// client
const client = net.createConnection('/tmp/myapp.sock')
client.on('data', (data) => console.log(data.toString()))
client.write('hello')
```

适合：本机的两个独立服务、CLI 跟后台 daemon 通信、docker compose 里几个容器之间。

**4. Redis Pub/Sub**

跨机器、跨语言、生产级广播。常见的「多实例 Node 服务协调」方案：

```js
import { createClient } from 'redis'

const pub = createClient(); await pub.connect()
const sub = createClient(); await sub.connect()

// instance A
await sub.subscribe('order:created', (msg) => {
  console.log('收到:', JSON.parse(msg))
})

// instance B
await pub.publish('order:created', JSON.stringify({ id: 123 }))
```

成本是要部署 Redis、有网络 IO 开销。但跨机器场景几乎只能选它（或者 NATS / RabbitMQ 这类专门的消息中间件）。

**5. HTTP / gRPC**

最重但最通用。微服务架构下服务间通信都是这个路子。Node 之间通信用 HTTP 成本较高，但跨语言时是通用选择。

**选型决策**：

- 父子进程 → `fork()` IPC
- 同进程多线程 → `worker_threads`
- 本机多服务、不需要跨机器 → Unix domain socket
- 多实例 Node 同业务、需要广播 → Redis Pub/Sub
- 跨语言、跨服务 → HTTP / gRPC / 消息队列

项目里经常混用：单机内主进程 + 多 worker 用 IPC、worker 之间用 Redis 同步状态、对外暴露 HTTP API。

### Fastify vs Express vs Hono:现代 Node 服务框架怎么选？

Express 几乎是 Node 服务的代名词,但十几年没大改了。Fastify 和 Hono 是这两年的新选择,各有特点。

**Express(仍在用,但是包袱)**:

- 13+ 年的老兵,生态最广
- API 简单:req / res / next 三件套
- 性能在现代框架里算慢的——v4 之前请求处理走的还是回调风格
- 24 年 v5 终于发布,原生支持 Promise / async,但社区已经在看别的

**Fastify(性能 + 现代化)**:

- TypeScript 一等公民,Schema 校验内置(基于 JSON Schema)
- 性能比 Express 快 2-3 倍(基于改良过的路由 + serializer 优化)
- 插件系统完善,生态丰富(@fastify/* 几十个官方插件)
- 默认带请求日志、请求 id、错误处理这些「生产必备」的东西

```ts
import Fastify from 'fastify'
const app = Fastify({ logger: true })

app.get('/users/:id', {
  schema: {
    params: { type: 'object', properties: { id: { type: 'number' } }},
    response: { 200: { type: 'object', properties: { name: { type: 'string' }}}}
  }
}, async (request, reply) => {
  return { name: 'Alice' }
})

await app.listen({ port: 3000 })
```

Schema 不只是校验,还能用来自动序列化 response(比 JSON.stringify 快 2 倍)和生成 OpenAPI 文档。

**Hono(边缘 + 跨运行时)**:

- 最新选手(22 年起),主打「跨运行时」
- 同一份代码能跑在 Node、Bun、Deno、Cloudflare Workers、Vercel Edge、Lambda
- API 受 Express 影响但更现代(基于 Web Standards:Request/Response 接口)
- 体积极小(核心 < 14KB),冷启动快,适合 serverless

```ts
import { Hono } from 'hono'
const app = new Hono()

app.get('/', (c) => c.json({ ok: true }))
app.post('/users', async (c) => {
  const body = await c.req.json()
  return c.json({ created: true })
})

export default app   // 可以直接给 Cloudflare Workers 用
```

**对比表(重要维度)**:

| 维度 | Express | Fastify | Hono |
|------|---------|---------|------|
| 性能(req/sec) | 1x | 3x | 4x(Bun 上更高) |
| TS 支持 | 一般 | 优秀 | 优秀 |
| 生态成熟度 | 最高 | 高 | 中(追上来很快) |
| 边缘 / serverless | 一般 | 一般 | 极强 |
| 学习曲线 | 低 | 中 | 低 |

**选型决策**:

- 老项目维护、团队熟 Express → 不需要换
- 新项目、传统 Node 服务 → **Fastify**
- 部署到 Cloudflare Workers / Vercel Edge / 跨运行时 → **Hono**
- 内部脚本、小工具 → Express 开发效率高

社区里 Nest.js 这种「全家桶框架」底层也支持切到 Fastify(默认 Express),就是看中性能。

### HTTP/2、HTTP/3 在 Node 里支持得怎么样？

不少 Node 服务仍然:默认用的还是 HTTP/1.1。HTTP/2、HTTP/3 在协议层面提升不少,Node 都内置了。

**HTTP/2**:从 Node 8.4 起内置 `node:http2` 模块。主要收益:

- **多路复用**:一条 TCP 连接同时跑多个请求,没有 HTTP/1.1 的 head-of-line blocking
- **头部压缩**:HPACK 把重复 header 压一压,对 cookie / auth 大头特别有用
- **Server Push**:服务端主动推资源给客户端(实际上 Chrome 95 起已移除支持,这个特性凉了)

```js
import http2 from 'node:http2'
import fs from 'node:fs'

const server = http2.createSecureServer({
  key: fs.readFileSync('./key.pem'),
  cert: fs.readFileSync('./cert.pem'),
})

server.on('stream', (stream, headers) => {
  stream.respond({
    ':status': 200,
    'content-type': 'application/json',
  })
  stream.end(JSON.stringify({ ok: true }))
})

server.listen(8443)
```

注意 HTTP/2 几乎只跑在 HTTPS 上(h2c 即 cleartext HTTP/2 兼容性差)。生产里 Nginx 直接 `listen 443 ssl http2;`,HTTPS + HTTP/2 一起开。

**HTTP/3**:基于 QUIC(UDP 上的可靠传输),24 年还在 Node 实验性支持中(`--experimental-quic` flag)。生态库慢慢起来:

- `node-quic` / `@matrixai/quic` 等社区实现
- 想体验直接用 Cloudflare / Caddy 当 reverse proxy,业务 Node 仍跑 HTTP/2

HTTP/3 的最大卖点是「移动网络场景下连接快速恢复」——网络切换(WiFi → 4G)时不用重新握手,对实时通信类应用有意义。但传统 Web 应用提升不明显。

**生产实战**:

- 业务 Node 服务跑 HTTP/1.1 + Keep-Alive 通常已经足够
- HTTPS / HTTP/2 终止在 Nginx 或 Cloudflare 这种边缘
- 如果需要让 Node 直接运行 HTTP/2 的场景:内部微服务间高频通信、gRPC(基于 HTTP/2)

实际收益排序:连接池 (Keep-Alive) > HTTP/2 多路复用 > HTTP/3。先把 Keep-Alive 用对再考虑别的。

### Node 21+ 内置 fetch + undici 实战

Node 18 把 fetch 内置了,背后是 undici(npm 上独立维护的 HTTP/1.1 客户端,性能比 node-fetch 高 3-4 倍)。21+ 把 undici 接口也直接暴露出来,业务里能直接用更细粒度的能力。

**默认 fetch 跟以前用 axios 的差异**:

```js
// fetch (Node 18+ 内置)
const res = await fetch('https://api.example.com/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Alice' }),
})
const data = await res.json()

// 跟浏览器 fetch 一模一样,无需 polyfill
```

要注意几个跟浏览器不完全一样的细节:

- **默认没有超时**:必须自己加 AbortController
- **没有进度回调**:上传 / 下载进度要自己用 stream 算
- **没有内置的请求拦截 / 重试 / 缓存**:要么自己包一层,要么用 axios

**直接用 undici 拿更细的控制**:

```js
import { request, Agent } from 'undici'

// 自定义 Agent 控制连接池
const agent = new Agent({
  connect: { timeout: 5000 },
  keepAliveTimeout: 30000,
  keepAliveMaxTimeout: 60000,
  pipelining: 10,   // HTTP/1.1 pipelining
})

const { statusCode, body, headers } = await request('https://api.example.com', {
  dispatcher: agent,
})
const data = await body.json()
```

`undici.request` 比 fetch 更底层,少了一层 Response 包装,性能更好但 API 不如 fetch 友好。日常业务用 fetch,性能敏感场景用 undici。

**HTTP 代理支持**:

undici 21+ 起内置代理:

```js
import { ProxyAgent, setGlobalDispatcher } from 'undici'

const proxyAgent = new ProxyAgent('http://proxy.corp.example.com:8080')
setGlobalDispatcher(proxyAgent)

// 之后所有 fetch 都走代理
await fetch('https://api.example.com')
```

以前要装 `https-proxy-agent` 这种第三方包,现在 Node 自带。

**Mock 测试用 MockAgent**:

```js
import { MockAgent, setGlobalDispatcher } from 'undici'

const mockAgent = new MockAgent()
setGlobalDispatcher(mockAgent)

const pool = mockAgent.get('https://api.example.com')
pool.intercept({ path: '/users/1' }).reply(200, { name: 'Alice' })

const res = await fetch('https://api.example.com/users/1')
// 不真的发请求,返回 mock 数据
```

单元测试里再也不用 nock / msw 了,Node 自带 mock 能力。

**stream 流式处理**:

```js
const res = await fetch(url)
for await (const chunk of res.body) {
  // chunk 是 Uint8Array
  process(chunk)
}
```

下载大文件、SSE(Server-Sent Events)流处理都方便。

从 Node 21+ 起,**绝大多数场景 axios 已经可以不用了**——内置 fetch + undici 覆盖了 axios 的核心功能,且性能更好、零依赖。axios 还在的理由是「拦截器、统一错误处理、浏览器 + Node 一份代码」这类工程化封装,新项目可以考虑直接基于 fetch 包一层。

### Node 22+ 内置 SQLite 是怎么用的？

Node 22（24 年 4 月）实验性加了内置 SQLite，22.5 起 stable。也就是说小工具 / 单机服务 / 桌面应用不用再装 better-sqlite3 这类原生模块（编译麻烦、跨架构问题多）。

```js
import { DatabaseSync } from 'node:sqlite'

const db = new DatabaseSync('./app.db')

db.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    created_at INTEGER
  )
`)

// 写
const insert = db.prepare('INSERT INTO users (name, created_at) VALUES (?, ?)')
insert.run('Alice', Date.now())

// 读
const select = db.prepare('SELECT * FROM users WHERE name = ?')
const user = select.get('Alice')
console.log(user)
// { id: 1, name: 'Alice', created_at: 1715000000000 }

// 事务
const tx = db.prepare('INSERT INTO users (name, created_at) VALUES (?, ?)')
const insertMany = db.transaction((users) => {
  for (const u of users) tx.run(u.name, Date.now())
})
insertMany([{ name: 'Bob' }, { name: 'Carol' }])
```

跟 better-sqlite3 几乎一样的 API（实际 Node 团队就是参考它写的）。

**实战场景**：

- **CLI 工具的本地状态**：git-like 工具记录历史、Linter 缓存检查结果
- **桌面应用本地数据库**：Electron / Tauri 应用存用户数据
- **测试 / mock 环境**：在内存 SQLite 里跑测试，比 mock DB 更真
- **小型 BFF / Edge 函数**：流量不大的服务用 SQLite 比 PostgreSQL 简单 10 倍

跟 PostgreSQL / MySQL 比，SQLite 的边界：

- **单进程**：多个 Node 进程同时写会锁竞争
- **不适合超大数据**：单文件 281TB 上限听着大，但实际百 GB 级就该考虑别的
- **没有网络**：只有本机文件访问，要远程访问得自己包一层

实战工具：drizzle-orm 25 年起也加了 SQLite 适配，能写出类型安全的 SQL。

### Node 性能基准测试：autocannon / clinic.js

「我的接口慢」不是结论，是问题。基准测试能告诉你 QPS、P95 / P99 延迟、并发瓶颈在哪。Node 生态里 autocannon + clinic.js 是组合拳。

**autocannon**：压测工具，类似 wrk / ab，但 Node 写的、跨平台、配置友好。

```bash
# 装
npm i -g autocannon

# 压测
autocannon -c 100 -d 30 http://localhost:3000/api/users
# -c 100: 100 个并发连接
# -d 30:  跑 30 秒
```

输出：

```
┌─────────┬──────┬──────┬───────┬───────┬─────────┐
│ Stat    │ 2.5% │ 50%  │ 97.5% │ 99%   │ Avg     │
├─────────┼──────┼──────┼───────┼───────┼─────────┤
│ Latency │ 12ms │ 18ms │ 45ms  │ 89ms  │ 21ms    │
└─────────┴──────┴──────┴───────┴───────┴─────────┘

Req/Sec: 4827
```

需要重点关注的指标是 P99 延迟——50% 用户感受不到的优化等于没优化，要看尾部。

JS 里用 autocannon 也行（适合写脚本压测多个接口）：

```js
import autocannon from 'autocannon'

const result = await autocannon({
  url: 'http://localhost:3000/api/users',
  connections: 100,
  duration: 30,
})
console.log(result.requests.average)
```

**clinic.js**：Node 性能诊断套件，几个工具：

- **clinic doctor**：自动诊断，告诉你瓶颈是 CPU / IO / event loop / GC
- **clinic flame**：CPU 火焰图
- **clinic bubbleprof**：异步操作可视化
- **clinic heap**：内存分配可视化

```bash
# 压测同时跑 clinic doctor
clinic doctor --on-port 'autocannon http://localhost:3000' -- node server.js

# 看 CPU 火焰图
clinic flame --on-port 'autocannon http://localhost:3000' -- node server.js
```

跑完自动打开浏览器显示分析报告。火焰图能直接看出哪个函数占 CPU 多。

**实战流程**：

1. autocannon 压测，看 QPS 和延迟，定下基线
2. clinic doctor 跑一遍，看是哪个维度瓶颈
3. CPU 瓶颈 → clinic flame 找慢函数
4. IO 瓶颈 → 检查 DB / 网络调用
5. event loop 瓶颈 → 看有没有同步阻塞
6. 改完代码再 autocannon 一次对比

**常见瓶颈模式**：

- **JSON.parse 大 body**：clinic flame 里 `parse` 占 30%+ → 用 streaming JSON parser（如 `@dpc/streaming-json`）
- **同步加密**：`crypto.pbkdf2Sync` 在主线程执行，高并发下容易阻塞 → 用异步版本或 worker thread
- **DB 连接池太小**：autocannon 一加并发就 ECONNRESET → 调大 pool size
- **没用 keep-alive**：每个请求都重连 → http.Agent 配 `keepAlive: true`

社区 benchmark 习惯：用 [benchmark.js](https://benchmarkjs.com/) 做函数级 micro-benchmark、用 autocannon 做 HTTP 级 macro-benchmark、用 clinic 定位瓶颈。

### WebSocket vs Server-Sent Events 怎么选？

实时通信场景里 WebSocket 和 SSE 是两个常见选择。不少人默认上 WebSocket，但 SSE 在「服务端推、客户端不发」的场景更简单。

**Server-Sent Events（SSE）**：

- **单向**：服务端 → 客户端
- **基于 HTTP**：长连接 HTTP 流，不用握手、不用单独协议
- **自动重连**：浏览器原生支持，断了自动重连
- **简单**：服务端就是一个 HTTP endpoint，写法跟普通 API 一样

```js
// 服务端 (Node)
import http from 'node:http'

http.createServer((req, res) => {
  if (req.url === '/events') {
    res.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    })

    setInterval(() => {
      res.write(`data: ${JSON.stringify({ time: Date.now() })}\n\n`)
    }, 1000)
  }
}).listen(3000)
```

```js
// 客户端
const events = new EventSource('/events')
events.onmessage = (e) => {
  console.log(JSON.parse(e.data))
}
events.onerror = (e) => console.log('断了,浏览器会自动重连')
```

**WebSocket**：

- **双向**：客户端 ↔ 服务端
- **协议升级**：从 HTTP upgrade 到 WS
- **手动管理重连 / 心跳**：浏览器原生 WebSocket 没有自动重连，要自己写
- **生态成熟**：socket.io、ws、@fastify/websocket 等

```js
// 服务端 (用 ws 库)
import { WebSocketServer } from 'ws'

const wss = new WebSocketServer({ port: 3000 })
wss.on('connection', (ws) => {
  ws.on('message', (msg) => console.log('收到:', msg.toString()))

  setInterval(() => {
    ws.send(JSON.stringify({ time: Date.now() }))
  }, 1000)
})
```

```js
// 客户端
const ws = new WebSocket('ws://localhost:3000')
ws.onmessage = (e) => console.log(e.data)
ws.send('hello')
```

**选型**：

| 场景 | 推荐 |
|------|------|
| 股票 / 新闻 / 监控数据推送（只下行） | SSE |
| ChatGPT 流式回复（只下行） | SSE |
| 聊天室、协作编辑（双向） | WebSocket |
| 实时游戏 | WebSocket |
| 通知中心 | SSE |

ChatGPT 用的就是 SSE——服务端往客户端推 token 流，客户端不发数据。如果你做类似的 AI 流式输出场景，SSE 写起来简单得多。

**SSE 的常见问题**：

- **HTTP/1.1 下浏览器限制每个域名 6 个并发连接**——SSE 占一个，影响普通 API 调用。HTTP/2 / HTTP/3 没这个问题
- **代理可能 buffer 长连接**：Nginx 默认会 buffer SSE 响应，要配 `proxy_buffering off`
- **不支持自定义 header**：EventSource API 不能传自定义 header（鉴权要用 cookie 或 url query）

**WebSocket 的常见问题**：

- **没心跳就会被中间网络设备掐断**：默认要写 ping/pong
- **重连逻辑要自己写**：socket.io 替你处理，裸 ws 要包一层
- **跟 HTTP 中间件不兼容**：CORS、Cookie、限流这些 HTTP 中间件对 WS 无效

实战工具：

- **socket.io**：成熟、自带重连 / 房间 / 命名空间，但协议是 socket.io 私有的，性能不如裸 WS
- **ws / @fastify/websocket**：裸 WebSocket，自己实现重连
- **Phoenix Channels / Centrifugo**：实时通信平台，跨语言、跨客户端

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
