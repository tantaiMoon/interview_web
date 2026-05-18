[toc]



## 一、HTML 基础

### src 和 href 的区别是什么？

`src` 和 `href` 都用于引入外部资源，但用途不同。

1. `src` 表示把资源嵌入到当前文档中，常见于 `img`、`script`、`iframe`。
2. `href` 表示建立当前文档与外部资源的关联，常见于 `a`、`link`。
3. 浏览器处理时，`src` 引入的资源会参与当前内容解析；`href` 更多是建立引用关系。

典型例子：

1. `script src="..."` 会下载并执行脚本。
2. `link href="..."` 会加载样式文件并应用到页面。
3. `a href="..."` 是跳转链接。

### 对 HTML 语义化的理解是什么？

HTML 语义化指的是根据内容结构选择合适的标签，而不是一味使用 `div` 和 `span`。

它的价值主要有：

1. 结构更清晰，代码可读性更好。
2. 更利于 SEO，搜索引擎更容易理解页面结构。
3. 对无障碍访问更友好，屏幕阅读器能更好识别内容。
4. 便于团队协作和后期维护。

常见语义化标签有：`header`、`nav`、`main`、`article`、`section`、`aside`、`footer`。

### DOCTYPE（文档类型）的作用是什么？

`DOCTYPE` 的作用是告诉浏览器当前文档使用哪种 HTML 规范，这样可以决定采用什么渲染模式。

以 `<!DOCTYPE html>` 为例，它表示当前文档使用 HTML5 标准。

它的核心意义：

1. 触发浏览器的标准模式。
2. 避免浏览器进入混杂模式。
3. 减少不同浏览器之间的兼容差异。

如果没有正确声明，浏览器可能会用旧规则渲染页面，导致盒模型、布局等行为异常。

### script 标签中 defer 和 async 的区别是什么？

它们都用于优化脚本加载，不阻塞 HTML 解析，但行为不同。

1. `defer`：脚本异步下载，等 HTML 解析完成后按顺序执行。
2. `async`：脚本异步下载，下载完成后立即执行，执行顺序无法保证。

使用建议：

1. 有依赖关系的脚本用 `defer`。
2. 独立的统计、埋点、广告脚本适合用 `async`。

简单区分：`defer` 保证执行顺序，`async` 按下载完成时间执行。

### 常用的 meta 标签有哪些？

按用途分两类。

**SEO / 文档元信息**：

```html
<meta charset="UTF-8">                         <!-- 编码 -->
<meta name="description" content="页面摘要">    <!-- 搜索结果摘要 -->
<meta name="keywords" content="...">           <!-- 现在权重很低，搜索引擎基本不看 -->
<meta name="author" content="...">
<meta name="robots" content="index, follow">   <!-- 控制爬虫 -->
<meta http-equiv="refresh" content="3;url=/">  <!-- 跳转 -->
```

**移动端 / 视口控制**：

```html
<!-- 几乎必写的一行 -->
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">

<!-- 浏览器地址栏配色（手机厂商定制） -->
<meta name="theme-color" content="#1677ff">

<!-- iOS Safari 全屏 PWA -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

<!-- 禁用电话号码自动识别（iOS 老问题） -->
<meta name="format-detection" content="telephone=no">
```

`viewport-fit=cover` 是 iPhone X 之后的刘海屏适配关键——配合 CSS 的 `env(safe-area-inset-bottom)` 给底部 Tab 留出安全区。

**社交分享（Open Graph / Twitter Card）**：

```html
<meta property="og:title" content="...">
<meta property="og:image" content="...">
<meta property="og:url" content="...">
<meta name="twitter:card" content="summary_large_image">
```

这套是分享到微信/微博/Twitter 时卡片预览的来源。

### head 标签有什么作用，其中什么标签必不可少？

`head` 用来存放页面元信息和外部资源引用，本身不会直接显示在页面中。

常见内容包括：

1. 页面标题 `title`
2. 编码声明 `meta charset`
3. 视口配置 `meta viewport`
4. 样式文件 `link`
5. 脚本 `script`
6. SEO 信息 `meta description`

必不可少的通常是：

1. `title`
2. `meta charset`
3. `<!DOCTYPE html>` 虽然不在 `head` 中，但也是文档必须具备的重要声明。

### 行内元素有哪些？块级元素有哪些？空元素有哪些？

常见块级元素有：`div`、`p`、`h1-h6`、`ul`、`ol`、`li`、`section`、`article`。

常见行内元素有：`span`、`a`、`strong`、`em`、`label`、`img`。

常见空元素也叫 void 元素，没有闭合内容，比如：`img`、`input`、`br`、`hr`、`meta`、`link`。

不过要注意，HTML5 中元素的显示特性可通过 CSS 修改，所以“块级/行内”更偏默认表现，而不是绝对属性。

### title 与 h1 的区别，b 与 strong 的区别，i 与 em 的区别？

这种题主要就看你能不能把语义和用途分开。

1. `title` 是浏览器标签页标题，也影响搜索结果标题；`h1` 是页面正文中的一级标题。
2. `b` 主要表示样式上的加粗；`strong` 表示语义上的强调，通常也会加粗显示。
3. `i` 主要表示样式上的斜体；`em` 表示语义上的强调，通常也会斜体显示。

`strong` 和 `em` 更有语义价值，也更利于无障碍和搜索引擎理解。

### label 的作用是什么？如何使用？

`label` 的作用是给表单控件定义说明文字，并增强点击区域和可访问性。

使用方式主要有两种：

1. `for` 属性关联控件的 `id`。
2. 直接把表单控件包裹在 `label` 内部。

示例：

```html
<label for="username">用户名</label>
<input id="username" type="text" />
```

点击“用户名”文字时，输入框会自动获得焦点。

### iframe 有哪些优点和缺点？

`iframe` 用于在当前页面中嵌入另一个独立页面。

好处：

1. 内容隔离，互不影响。
2. 适合嵌入第三方页面、地图、文档、支付页。
3. 可以复用现有系统或独立子应用。

问题：

1. 性能开销更大。
2. 不利于 SEO。
3. 调试复杂，通信麻烦。
4. 存在安全风险，需要配合 `sandbox`、`CSP` 等控制。

现代前端里，`iframe` 更适合做隔离嵌入，而不是常规业务布局手段。

### 浏览器乱码的原因是什么？如何解决？

乱码一般就是“文件编码”和“浏览器解析编码”没对上。

常见原因：

1. HTML 文件本身不是 UTF-8。
2. `meta charset` 配置与文件实际编码不一致。
3. 服务端响应头中的字符集设置错误。
4. 数据库存储和页面输出编码不统一。

解决方式：

1. 统一使用 UTF-8。
2. 页面声明 `meta charset="UTF-8"`。
3. 服务端响应头设置正确的 `Content-Type`。
4. 前后端、数据库统一编码规范。

### HTML 中 data-* 属性有什么作用？

`data-*` 是 HTML5 提供的自定义数据属性，用于在元素上存储额外信息。

它的特点大概是：

1. 语义上合法，不会和标准属性冲突。
2. 适合存放和页面交互相关的轻量数据。
3. JavaScript 可通过 `dataset` 方便读取。

示例：

```html
<button data-id="1001" data-role="delete">删除</button>
```

```javascript
const btn = document.querySelector('button')
console.log(btn.dataset.id)
console.log(btn.dataset.role)
```

使用建议：

1. 适合存轻量配置或标识。
2. 不适合存复杂业务数据。
3. 大量状态更适合交给 JavaScript 维护。

### disabled 和 readonly 的区别是什么？

这是一道典型表单题。

1. `disabled`
   - 表单控件不可操作
   - 不会参与表单提交
   - 通常不能获得焦点

2. `readonly`
   - 表单控件只读
   - 仍会参与表单提交
   - 通常可以获得焦点

如何区分：

1. 完全禁用：用 `disabled`
2. 只允许看、不允许改，但需要提交：用 `readonly`

### src、alt、title 在 img 上分别有什么作用？

1. `src`
   - 指定图片资源地址

2. `alt`
   - 图片加载失败时的替代文本
   - 也是屏幕阅读器读取图片含义的重要依据

3. `title`
   - 更多用于鼠标悬浮提示
   - 不能替代 `alt`

这里容易被忽略的是：

> `alt` 是语义和无障碍属性，`title` 更偏提示说明。

## 二、HTML5 与浏览器能力

### HTML5 有哪些更新？

按几个能力维度看会比单纯记忆标签更清晰：

- **语义结构**：`header`、`nav`、`main`、`article`、`section`、`aside`、`footer`，让文档结构能被机器和辅助技术理解。
- **多媒体**：原生 `<audio>` / `<video>`，不再依赖 Flash。
- **图形和绘制**：`<canvas>` 像素位图，`<svg>` 矢量图，外加 WebGL/WebGPU 接入。
- **表单增强**：`type="email|url|date|number|range|color"`、`required`、`pattern`、`placeholder`、`<datalist>`，浏览器自带校验。
- **存储**：localStorage / sessionStorage / IndexedDB 替代 cookie 当存储用的滥用。
- **设备能力**：地理定位、设备方向、电池、剪贴板、文件读取、拖拽、震动等原生 API。
- **离线和后台**：Service Worker / Web Worker / SharedWorker / Cache Storage。
- **通信**：WebSocket、Server-Sent Events、postMessage、Broadcast Channel。

一句概括：HTML5 把 Web 从"展示文档"扩成了"可以做应用"的平台。

### 资源加载相关：preload / prefetch / dns-prefetch / preconnect 区别？

都是 `<link rel>` 的取值，但作用阶段完全不同。

- **`dns-prefetch`**：只做 DNS 解析，开销极小，用来给后面要访问的第三方域名预先解析。`<link rel="dns-prefetch" href="//cdn.example.com">`
- **`preconnect`**：DNS + TCP + TLS 全部建好，比 dns-prefetch 重，但省下的时间更多。第三方关键资源用它。
- **`preload`**：**当前页面**马上会用到、但解析 HTML 时浏览器还发现不到的资源，提前下下来。典型用法是字体、关键 CSS、首屏图。必须写 `as` 指定类型，否则会重复请求。
- **`prefetch`**：**下一个页面**可能会用到的资源，浏览器空闲时再去拉，优先级低。SPA 路由预取常用。
- **`modulepreload`**：专门给 ES Module 用的 preload，会把模块图一起 warm 起来。

简单选法：当前页要用 → preload；下一页可能用 → prefetch；只想热一下连接 → preconnect。

### 响应式图片：`srcset` + `sizes` 和 `<picture>` 怎么选？

`srcset` + `sizes` 解决的是"同一张图，不同分辨率/尺寸"的问题。浏览器根据当前视口和 DPR 自己挑：

```html
<img
  src="small.jpg"
  srcset="small.jpg 480w, medium.jpg 1024w, large.jpg 1920w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="..."
>
```

`<picture>` 解决的是"不同条件，加载完全不同的图"——比如换格式、换裁剪、换方向：

```html
<picture>
  <source type="image/avif" srcset="hero.avif">
  <source type="image/webp" srcset="hero.webp">
  <source media="(max-width: 600px)" srcset="hero-mobile.jpg">
  <img src="hero.jpg" alt="...">
</picture>
```

记忆点：只换分辨率用 srcset，要换格式/换裁剪/横竖屏适配用 picture。

### IntersectionObserver、ResizeObserver、MutationObserver 各自干什么？

三个 Observer 都是浏览器原生的"观察"API，比 scroll/resize 监听轻得多。

**IntersectionObserver**：观察元素和视口（或某个父容器）的相交状态。图片懒加载、无限滚动、曝光埋点都靠它。

```js
const io = new IntersectionObserver(entries => {
  entries.forEach(e => {
    if (e.isIntersecting) e.target.src = e.target.dataset.src
  })
}, { rootMargin: '100px' })
document.querySelectorAll('img[data-src]').forEach(img => io.observe(img))
```

**ResizeObserver**：观察元素尺寸变化。比监听 window.resize 更准——元素自身被内容撑开、被父容器 flex 调整都会触发。常用于自适应组件、ECharts 自动 resize。

**MutationObserver**：观察 DOM 子树的结构或属性变化。第三方脚本注入检测、富文本编辑器、虚拟列表里都用得到。

为什么不直接 scroll/resize/setInterval？因为 Observer 是浏览器在合适时机批量回调，不阻塞主线程；监听事件需要自己防抖、自己读 offsetTop，性能通常更差。

### 对 Web Worker 的理解是什么？

主线程负责 UI、JS 和事件回调，遇到耗时计算（解析大 JSON、跑算法、压缩图片）就会阻塞页面。Web Worker 把这些计算放到独立线程执行，主线程继续刷新 UI。

用法简单：

```js
// main.js
const worker = new Worker('./heavy.js')
worker.postMessage({ data: bigArray })
worker.onmessage = e => console.log('结果', e.data)

// heavy.js
self.onmessage = e => {
  const result = doHeavyCalc(e.data.data)
  self.postMessage(result)
}
```

要点：

- Worker 跑在独立全局环境里，**拿不到 DOM、window、document**，只能算数据。
- 主线程和 Worker 之间只能 `postMessage` 通信，传过去的对象会被结构化克隆（大对象用 `Transferable` 转移所有权能避免拷贝）。
- 适合 CPU 密集型计算；如果只是"代码顺序执行慢"，扔进 Worker 反而因为通信开销更慢。

衍生类型：`SharedWorker` 多个标签共享一个 Worker；`Service Worker` 是离线缓存代理，不是干计算的。

### HTML5 的离线存储怎么使用，它的工作原理是什么？

老的方案是 AppCache，写一个 manifest 文件，把要缓存的资源列出来，浏览器首次访问就把这些资源放到本地，下次断网也能打开。但这套机制更新逻辑不直观（manifest 没变就一直用旧缓存，改一个字节才会重新拉），调试成本也高，标准里已经废弃了。

现在的标准答法是 Service Worker + Cache Storage：

```js
// sw.js
self.addEventListener('install', e => {
  e.waitUntil(caches.open('v1').then(c => c.addAll(['/index.html', '/app.js'])))
})

self.addEventListener('fetch', e => {
  e.respondWith(caches.match(e.request).then(r => r || fetch(e.request)))
})
```

页面里 `navigator.serviceWorker.register('/sw.js')` 注册一下即可。Service Worker 跑在独立线程上，拦截页面发出的所有请求，由 JS 代码决定走缓存还是走网络。

常见的几种缓存策略，放在一起看会更清楚：

- Cache First：先查缓存，没命中再走网络。适合静态资源（CSS、JS、字体、图标）。
- Network First：先请求网络，失败再用缓存。适合 API 数据这种新鲜度敏感的内容。
- Stale While Revalidate：立刻返回缓存，同时后台拉新版本更新缓存。体验和新鲜度的折中。
- Cache Only / Network Only：极端场景，前者完全离线包，后者跳过缓存。

### 浏览器是如何对 HTML5 的离线存储资源进行管理和加载的？

落到现代实现就是 Service Worker 一层拦截：浏览器发起请求时先经过 SW，SW 根据策略决定从 Cache Storage 取还是去网络拿，再把响应交回给页面。Cache Storage 本身是按 origin 隔离的键值存储，key 是 Request 对象，value 是 Response，开发者可以手动 `caches.open(name).then(c => c.put / c.match / c.delete)`。

更新机制相对简单：浏览器周期性比对 sw.js 文件本身（字节级别），文件变了就触发新版本的 install 阶段，老 SW 仍在控制页面，新 SW 进入 waiting，等所有旧标签关闭再 activate。强制立即激活可以在 install 里调 `self.skipWaiting()`，配合 `clients.claim()` 接管已有页面。

存储清理上，Cache Storage 跟 IndexedDB、localStorage 一起算 origin 的总配额，超出后浏览器可能整体回收。生产上一般会自己维护版本号，新版本激活时清掉老版本的 cache。

### Canvas 和 SVG 的区别是什么？

两者都能绘图，但原理不同：

1. Canvas 是基于像素绘图，适合频繁重绘。
2. SVG 是基于矢量和 DOM 节点，适合结构化图形。

更适合的场景：

1. Canvas：游戏、图表、粒子动画、图像处理。
2. SVG：图标、流程图、可缩放图形、交互式图表。

Canvas 更适合高频绘制，SVG 更适合可交互、可维护的矢量图形。

### 渐进增强和优雅降级的区别是什么？

这两个概念实际不是一回事，走的是两种兼容思路。

1. 渐进增强：先保证核心功能在低版本环境可用，再逐步为高版本浏览器增强体验。
2. 优雅降级：先针对现代浏览器实现完整效果，再向低版本浏览器做兼容退化。

通常更推荐渐进增强，因为它更关注基本功能可用性，也更符合现代 Web 的稳定交付思路。

### 说一下 HTML5 Drag API。

HTML5 Drag API 是浏览器原生拖拽能力，常用于实现元素拖动、排序、拖放上传等场景。

常见事件有：

1. `dragstart`
2. `drag`
3. `dragover`
4. `drop`
5. `dragend`

实现拖放时要注意：

1. 目标元素需要阻止默认行为，否则无法触发放置。
2. 可通过 `dataTransfer` 传递拖拽数据。

现代复杂拖拽场景里，不少项目也会直接使用成熟拖拽库来保证兼容性和体验。

### localStorage、sessionStorage、cookie 的区别是什么？

高频经典题。三者放一张表对比最清楚：

| 维度 | Cookie | localStorage | sessionStorage |
|------|--------|-------------|----------------|
| 容量 | 约 4KB | 5MB 左右 | 5MB 左右 |
| 生命周期 | 由 `Expires` / `Max-Age` 决定，不设是会话级 | 永久，手动删才没 | 标签页关闭就没 |
| 是否随请求发送 | 同源请求自动带 | 不会 | 不会 |
| 跨标签页共享 | 同源都共享 | 同源都共享 | **不共享**，每个标签独立 |
| API | `document.cookie` 字符串拼接 | `localStorage.setItem/getItem` | 同 localStorage |
| 安全属性 | `HttpOnly`、`Secure`、`SameSite` | 无，JS 完全可访问 | 无，JS 完全可访问 |

选用思路：

- **服务端认证、CSRF token**：cookie（带 HttpOnly + SameSite）
- **用户偏好、长期客户端配置**：localStorage
- **当前标签页临时状态**（比如多步表单、当前选中项）：sessionStorage
- **复杂结构化数据、大体积**（离线数据、缓存图）：IndexedDB

localStorage 是同步阻塞 API，频繁大数据读写会卡主线程，几 MB 的数据应该走 IndexedDB。

### 什么是浏览器的同源策略？

同源策略是浏览器的核心安全机制。"同源"指**协议、域名、端口**三者完全一致，差一个都算跨域。

它限制三类行为：

1. **跨源 DOM 访问**：A 页面拿不到 B 页面的 `document`、`window`（iframe 跨源也读不了）。
2. **跨源存储读取**：Cookie / localStorage / IndexedDB 按 origin 隔离。
3. **跨源 AJAX 响应读取**：请求可以发出去，但 JS 读不到响应（除非服务端开 CORS）。

目的就一个：防止 evil.com 在用户登录了 bank.com 的状态下，偷偷读取 bank 的数据。

注意几个不受同源策略限制的：

- `<img>`、`<script>`、`<link>`、`<video>` 等标签的资源请求本身不受限（这也是 JSONP 的原理，CSRF 攻击也跟这个有关）
- `<form>` 提交可以跨域，但拿不到响应
- WebSocket 不受同源策略限制（有自己的 Origin 检查机制）

### 跨域的常见解决方案有哪些？

主流就两条路：**CORS** 和 **代理**。

**CORS（推荐）**：服务端响应头里加 `Access-Control-Allow-Origin: https://example.com`，浏览器就放行。复杂请求（非简单方法、自定义头、非 simple content-type）会先发一个 OPTIONS 预检，服务端返回允许的方法和头，主请求才会真正发出去。带 Cookie 的还要：

- 前端 `fetch(url, { credentials: 'include' })` 或 `xhr.withCredentials = true`
- 后端响应 `Access-Control-Allow-Credentials: true`
- 而且 `Allow-Origin` 此时**不能是 `*`**，必须是具体域名

**代理（开发常用）**：开发环境 webpack/vite 的 proxy 把 `/api` 转发到后端，生产环境 nginx 同源转发。浏览器只看见自己的域，根本不触发跨域。

**JSONP**：利用 `<script>` 不受同源策略限制，让服务端返回一段调用回调函数的 JS。只能 GET，安全性差，现代项目基本淘汰。

**postMessage**：父子窗口、iframe、Web Worker 之间通信用，跟接口跨域是两回事。

**document.domain**：父子域之间共享 DOM 用，已经被现代浏览器逐步废弃（不安全）。

### CORS 预检请求（OPTIONS）什么时候发？

不是所有跨域请求都要预检。"简单请求"直接发，不简单的先发 OPTIONS。

**简单请求要同时满足**：

- 方法是 `GET` / `HEAD` / `POST`
- 自定义头只能是几个安全头（`Accept`、`Accept-Language`、`Content-Language`、`Content-Type`）
- `Content-Type` 只能是 `text/plain` / `multipart/form-data` / `application/x-www-form-urlencoded`

只要破其中一条（比如 `Content-Type: application/json`、带 `Authorization` 头、用 `PUT`/`DELETE`），就是非简单请求，先发 OPTIONS。

服务端可以用 `Access-Control-Max-Age: 86400` 缓存预检结果，一天内同样的请求不会再发 OPTIONS。

### Cookie 的 SameSite / Secure / HttpOnly 都是什么？

三个安全相关属性。

- **`HttpOnly`**：设置后 JS 拿不到这个 Cookie（`document.cookie` 看不见），防 XSS 偷 token。登录态 Cookie 必须加。
- **`Secure`**：只在 HTTPS 下发送。
- **`SameSite`**：控制跨站请求是否带 Cookie。
  - `Strict`：完全不带，即使从外站点链接跳过来也不带（用户体验最差但最安全）
  - `Lax`（Chrome 默认）：顶层导航的 GET 请求会带，其他不带，能防大部分 CSRF
  - `None`：跨站请求都带，但必须同时设 `Secure`，否则被拒
- **`Domain`** / **`Path`**：作用范围。`Domain=.example.com` 让子域共享。
- **`Max-Age`** / **`Expires`**：过期时间，都不设就是 Session Cookie，关闭浏览器就没了。

典型登录态 Cookie 写法：`Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=86400`。

### 如何理解无障碍访问（Accessibility / a11y）？

无障碍访问指的是让残障用户也能正常使用页面。

常见实践有：

1. 使用语义化标签
2. 给图片提供合理的 `alt`
3. 表单控件正确配合 `label`
4. 保证键盘可操作
5. 适当使用 ARIA 属性
6. 保证足够的颜色对比度

它的意义不只是“照顾特殊人群”，还包括：

1. 提升整体可用性
2. 提升 SEO
3. 提升产品专业度

## 三、CSS 基础

### 说说你对盒子模型的理解。

盒子模型描述的是元素在页面中占据空间的计算方式，主要由四部分组成：

1. `content`
2. `padding`
3. `border`
4. `margin`

盒模型分两种：

1. 标准盒模型：`width` 只包含内容区。
2. IE 盒模型：`width` 包含 `content + padding + border`。

可通过 `box-sizing` 控制：

1. `content-box`：标准盒模型。
2. `border-box`：更常用，布局计算更直观。

### CSS 选择器有哪些？优先级如何？哪些属性可以继承？

按类型分：标签、类、id、属性、后代/子代/兄弟组合、伪类（`:hover` / `:nth-child` 等）、伪元素（`::before` 等），以及通配符 `*`。

优先级从高到低：`!important` > 行内 `style` > id > 类 / 属性 / 伪类 > 标签 / 伪元素 > 通配符。

能继承的基本都是文字相关属性：`color`、`font-*`、`line-height`、`text-align`、`letter-spacing`、`word-spacing`、`visibility`、`cursor`，列表相关的 `list-style`，还有自定义属性（CSS Variables）。盒模型、定位、背景、布局这些（`width`、`margin`、`padding`、`border`、`background`、`display`、`position`）默认不继承。

想强制继承可以用 `inherit`，或者通用兜底的 `all: inherit`。

### CSS 选择器权重是怎么计算的？

CSS 规范里用一个四元组 `(a, b, c, d)` 表示优先级：

- `a`：行内 `style="..."`，有就 1，否则 0
- `b`：id 选择器的个数
- `c`：类、属性、伪类的个数
- `d`：标签、伪元素的个数

比较时从左往右逐位比，前一位大的整体就大，**不会进位**。也就是说 11 个类（0,0,11,0）也压不过一个 id（0,1,0,0）。

`!important` 凌驾于所有上面的规则，权重相同的 important 之间再按四元组比；都打了 important 又同权重，就看代码顺序后写覆盖先写。

几个问题：

- `:not(.x)` 不参与权重，但 `.x` 参与。`:is()` 也是按里面最高权重那个算。
- `:where()` 永远不贡献权重，常用于库样式打底。
- 内联样式可以被 `!important` 覆盖，所以 `!important` 真不是最高优先级（在两端都用时还要比 important + 四元组）。

### CSS 中伪类和伪元素有什么区别？常见的有哪些？

伪类用于描述元素的“状态”，伪元素用于创建或选中特定的“元素片段”。

常见伪类有：

1. `:hover`
2. `:active`
3. `:focus`
4. `:visited`
5. `:first-child`
6. `:last-child`
7. `:nth-child(n)`
8. `:not()`

常见伪元素有：

1. `::before`
2. `::after`
3. `::first-letter`
4. `::first-line`
5. `::selection`

伪类强调“元素状态”，伪元素强调“元素内容片段或额外生成内容”。

### 说说 em、px、rem、vh、vw 的区别。

这些都是 CSS 长度单位，但参照对象不同。

1. `px`：绝对单位，基于 CSS 像素。
2. `em`：相对单位，相对当前元素或父元素字体大小。
3. `rem`：相对单位，相对根元素字体大小。
4. `vh`：视口高度的百分比。
5. `vw`：视口宽度的百分比。

使用建议：

1. 字体和间距做整体缩放时常用 `rem`。
2. 需要基于视口做适配时常用 `vw/vh`。
3. `px` 适合固定尺寸场景。

### 说说设备像素、CSS 像素、设备独立像素、DPR、PPI 之间的区别。

1. 设备像素：屏幕物理像素点。
2. CSS 像素：浏览器中的逻辑像素。
3. 设备独立像素：一种抽象单位，便于跨设备统一显示。
4. DPR：设备像素比，通常表示 `设备像素 / CSS 像素`。
5. PPI：每英寸像素数，反映屏幕精细度。

常见结论：

1. DPR 越高，同样的 CSS 尺寸会映射更多物理像素。
2. 高清屏容易出现 1px 边框、图片模糊等适配问题。

解决方式：

1. 1px 边框问题
   在高 DPR 屏幕下，`1px` 的 CSS 边框可能会映射成多个物理像素，视觉上显得偏粗。
   常见解决方式有：
   1. 使用伪元素配合 `transform: scale()` 做 hairline 方案。
   2. 使用 `box-shadow` 模拟细线。
   3. 使用 SVG 绘制线条。
   4. 某些场景下直接使用 `0.5px`，但要注意兼容性。

2. 图片模糊问题
   图片模糊的本质还是图片资源分辨率不够，高 DPR 设备会把低分辨率图片放大显示。
   常见解决方式有：
   1. 提供 2x、3x 高清图。
   2. 使用 `srcset` 做响应式图片。
   3. 图标优先使用 SVG。
   4. 背景图按不同 DPR 提供不同资源。
   5. 避免把小图强行放大显示。

高清屏适配的关键，就是把“CSS 像素”和“物理像素”分开理解。边框问题重点在 hairline，图片问题重点在高分辨率资源。

### CSS 中有哪些方式可以隐藏页面元素？区别是什么？

常见隐藏方式有：

1. `display: none`：元素不占空间，不参与渲染。
2. `visibility: hidden`：元素占空间，但不可见。
3. `opacity: 0`：元素透明，但仍占空间，也可响应事件。
4. 绝对定位移出视口。
5. `clip-path` 或无障碍隐藏方案。

主要差别就在这几项：

1. 是否占据文档流空间。
2. 是否能响应事件。
3. 是否会被屏幕阅读器读取。
4. 是否触发回流和重绘。

如果展开说：

1. `display: none`
   - 不参与布局
   - 通常不会响应事件

2. `visibility: hidden`
   - 仍占位
   - 不可见

3. `opacity: 0`
   - 仍占位
   - 仍可能响应点击

4. 移出视口 / 裁剪隐藏
   - 常用于特殊动画或无障碍场景

### 谈谈你对 BFC 的理解。

BFC 全称 Block Formatting Context，块级格式化上下文。可以把它看成页面里的一个独立“小区域”，里面怎么排版，尽量不去影响外面。

它能解决三类经典问题：

1. **清除浮动**：父容器触发 BFC 后，计算高度时会把浮动子元素也算进去，父容器不会塌陷。
2. **阻止 margin 折叠**：相邻块元素的垂直 margin 在同一个 BFC 内会折叠，分到不同 BFC 就不会。
3. **阻止文字环绕浮动元素**：右侧内容设置 `overflow: hidden` 触发 BFC，左侧浮动元素就不会侵入它。

怎么触发：

- `overflow` 不为 `visible`
- `display: flow-root`（最干净的方式，副作用最小，专门为 BFC 设计的）
- `display: inline-block` / `table-cell` / `flex` / `grid`
- `float` 不为 `none`
- `position: absolute` / `fixed`

老代码爱用 `overflow: hidden` 触发 BFC，但它会把超出部分裁掉。现代写法直接 `display: flow-root` 更好。

### 什么是层叠上下文（Stacking Context）？

层叠上下文是 z 轴方向上的独立绘制环境。同一个层叠上下文里的元素按规则比较谁在上面，不同层叠上下文之间则是整体比较——里面再大的 `z-index` 也压不出去。

什么情况会创建新的层叠上下文：

- 根元素 `<html>`
- `position: relative/absolute` 且 `z-index` 不是 `auto`
- `position: fixed` / `sticky`（不管有没有 z-index）
- `opacity` 小于 1
- `transform`、`filter`、`perspective`、`clip-path`、`mask` 不为 `none`
- `will-change` 指定了会创建层叠上下文的属性
- `isolation: isolate`

常见问题：父级是 `transform` 元素，里面的 `position: fixed` 子元素不再相对视口定位，而是相对这个 transform 元素。`z-index: 9999` 盖不住别的元素，往往也是因为父级建立了独立的层叠上下文，无法脱离该上下文参与外部层级比较。

同一层叠上下文内的层叠顺序（从下往上）：背景和边框 → 负 z-index → 普通块 → 浮动 → 普通行内 → z-index 为 auto/0 的定位元素 → 正 z-index 的定位元素。

### 什么是 containing block（包含块）？

包含块就是元素布局时的参考矩形。绝对定位走哪、百分比宽高按谁算、`top/left` 偏移相对谁，都看包含块。

规则简化下来：

- `static` / `relative` 元素：包含块是最近的块级祖先的内容区。
- `absolute` 元素：包含块是最近的 `position` 不为 `static` 的祖先的 padding box。如果一路找不到，就是初始包含块（视口）。
- `fixed` 元素：包含块是视口。**但如果祖先有 `transform`、`filter`、`perspective`、`will-change` 等，会变成那个祖先**——这是 fixed 看起来"失效"的常见原因。
- `sticky` 元素：包含块是最近的滚动祖先。

不少"绝对定位跑偏""百分比宽度不对"的问题，根源都是包含块与预期不一致。

### `@import` 和 `link` 引入 CSS 的区别是什么？

两者都能引入样式，但行为和适用场景不同。

主要区别：

1. `link` 属于 HTML 标签，浏览器会并行加载资源。
2. `@import` 属于 CSS 语法，通常要等当前 CSS 解析到该语句时再继续加载。
3. `link` 兼容性和性能通常更好。
4. `@import` 除了引入样式，还可以配合媒体查询做条件导入。

项目里，更推荐使用 `link` 引入主样式文件，`@import` 更适合特定样式组织场景。

## 四、布局与适配

### CSS 中 position 有哪些取值？它们的区别是什么？

| 取值 | 是否脱离文档流 | 相对谁定位 | 常见场景 |
|------|--------------|----------|---------|
| `static` | 否 | 默认值，`top/left` 等失效 | 普通流元素 |
| `relative` | 否（占原位） | 自身原位置 | 给子绝对定位当参考、微调元素位置 |
| `absolute` | 是 | 最近的非 static 祖先（找不到就是初始包含块） | 弹层、角标、tooltip |
| `fixed` | 是 | 视口（祖先有 transform 等会变成那个祖先） | 吸顶、悬浮按钮、Modal 遮罩 |
| `sticky` | 半脱离 | 最近的滚动容器，到阈值前像 relative，到阈值后像 fixed | 表头吸顶、章节标题 |

几个高频注意点：

- `absolute` 找不到合适的 `relative` 祖先时，会一路查到 `<html>`，最终相对初始包含块定位。
- `fixed` 在父级有 `transform` / `filter` / `will-change` 时，包含块会变成那个父级，看起来不再相对视口。
- `sticky` 不生效通常是父级 `overflow: hidden` 改变了滚动容器，或者没有设置 `top` 阈值。

### 浮动为什么会导致父元素高度塌陷？如何清除浮动？

浮动元素脱离了普通文档流，父元素算高度时只看普通流子元素，没看到浮动的，自然就塌成 0。

清浮动一共三种主流写法：

```css
/* 1. 触发 BFC，最干净 */
.parent { display: flow-root; }

/* 2. 兜底 BFC（兼容老浏览器） */
.parent { overflow: hidden; }

/* 3. 经典 clearfix（伪元素） */
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

现代项目基本不用浮动做布局了，Flex / Grid 一上来就没有塌陷问题。这道题主要考你知不知道 BFC 原理。

### 元素水平垂直居中的方法有哪些？如果元素不定宽高呢？

常见方案有：

1. Flex：`display: flex; justify-content: center; align-items: center;`
2. Grid：`display: grid; place-items: center;`
3. 绝对定位 + `transform: translate(-50%, -50%)`
4. 行高配合 `text-align`，但只适合简单单行文本场景。

如果元素不定宽高，最推荐：

1. Flex
2. Grid
3. 绝对定位 + `transform`

因为这些方式都不依赖已知宽高。

常见用的写法就是：

1. Flex 居中

```html
<div class="parent">
  <div class="child">内容</div>
</div>
```

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 200px;
}
```

2. Grid 居中

```css
.parent {
  display: grid;
  place-items: center;
  height: 200px;
}
```

3. 绝对定位 + `transform`

```css
.parent {
  position: relative;
  height: 200px;
}

.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

如果需要说“最推荐哪个”，一般就这么答：

1. 现代项目优先 Flex。
2. 二维同时控制时可用 Grid。
3. 兼容或特殊场景再考虑绝对定位。

### 如何实现两栏布局，右侧自适应？三栏布局中间自适应呢？

两栏布局常见方案：

1. Flex：左侧定宽，右侧 `flex: 1`
2. 左侧浮动，右侧 `overflow: hidden`
3. 绝对定位配合 margin

三栏布局中间自适应常见方案：

1. Flex：左右定宽，中间 `flex: 1`
2. 圣杯布局
3. 双飞翼布局
4. Grid 布局

现代项目里，更推荐 Flex 或 Grid，语义清晰、兼容性也足够。

常见用的写法就是：

1. 两栏布局，右侧自适应，Flex 方案

```html
<div class="two-col">
  <div class="left">左侧</div>
  <div class="right">右侧自适应</div>
</div>
```

```css
.two-col {
  display: flex;
}

.two-col .left {
  width: 200px;
  background: #f66;
}

.two-col .right {
  flex: 1;
  background: #ddd;
}
```

2. 两栏布局，右侧自适应，Float + BFC 方案

```css
.left {
  float: left;
  width: 200px;
}

.right {
  overflow: hidden;
}
```

如果继续问圣杯布局和双飞翼布局：

两者目标完全一样——三栏布局、中间自适应、HTML 里中间内容先渲染（这点对 SEO 和首屏体验很关键，左右栏一般是辅助信息）。区别在于"左右栏空间从哪里预留"：

- **圣杯**：父容器写 `padding-left/right` 给左右栏预留位置，左右栏再用负 margin + `position: relative` 走到坑里。
- **双飞翼**：中间栏外面再套一层 wrapper，wrapper 占满一行；实际的内容容器靠 `margin-left/right` 给左右栏让位。

圣杯结构简单点但要靠定位调整；双飞翼多套一层 DOM 但不用动 position。本质都是 float 时代为了"中间内容先写、布局靠后调整"做的妥协。

现代项目基本不会再用这两个，Flex 或 Grid 几行就能写完。面试要求会写一遍代码：

圣杯：

```html
<div class="holy-grail">
  <div class="main">中间</div>
  <div class="left">左侧</div>
  <div class="right">右侧</div>
</div>
```

```css
.holy-grail { padding: 0 200px 0 150px; }
.holy-grail .main,
.holy-grail .left,
.holy-grail .right { float: left; min-height: 200px; }
.holy-grail .main  { width: 100%; }
.holy-grail .left  { width: 150px; margin-left: -100%; position: relative; left: -150px; }
.holy-grail .right { width: 200px; margin-left: -200px; position: relative; right: -200px; }
```

双飞翼：

```html
<div class="double-wing">
  <div class="main-wrap"><div class="main">中间</div></div>
  <div class="left">左侧</div>
  <div class="right">右侧</div>
</div>
```

```css
.double-wing .main-wrap,
.double-wing .left,
.double-wing .right { float: left; min-height: 200px; }
.double-wing .main-wrap { width: 100%; }
.double-wing .main      { margin: 0 200px 0 150px; }
.double-wing .left      { width: 150px; margin-left: -100%; }
.double-wing .right     { width: 200px; margin-left: -200px; }
```

两套代码的核心都是：主区域先 `width: 100%` 占满一行，左右栏靠负 margin 拉回到同一行；区别在于预留空间的方式——一个用父 padding，一个用子 margin。

### 说说 Flexbox（弹性盒布局模型）以及适用场景。

Flex 是一维布局，意思是它一次只管一个方向——主轴或者交叉轴。所有子项要么排成一行，要么排成一列，剩余空间按规则分配。

容器属性管两件事：方向（`flex-direction`、`flex-wrap`）和对齐（`justify-content`、`align-items`、`align-content`）。子项属性管自身的伸缩和位置（`flex`、`align-self`、`order`）。

什么时候选 Flex：

- 一行/一列的对齐和间距问题
- 子元素之间按比例分剩余空间
- 单个元素的水平垂直居中（`display: flex; place-items: center;` 一行完成）
- 顶部导航栏左右分布、按钮组、表单项横向排列
- 左侧固定 + 右侧自适应这种经典两栏

什么时候不选：行和列同时要精确控制位置的场景，比如仪表盘那种网格，用 Grid 更好用。

### Flex 容器和项目常用属性有哪些？

容器上能用的：

- `flex-direction`：主轴方向，`row | row-reverse | column | column-reverse`
- `flex-wrap`：是否换行，`nowrap | wrap | wrap-reverse`
- `flex-flow`：上面两个的简写
- `justify-content`：主轴对齐，`flex-start | center | space-between | space-around | space-evenly`
- `align-items`：交叉轴对齐，`stretch | center | flex-start | flex-end | baseline`
- `align-content`：多行时的整体对齐，单行不生效
- `gap`：项之间的间距，比 margin 干净

子项上能用的：`order` 改顺序、`flex-grow` / `flex-shrink` / `flex-basis` 控制伸缩、`flex` 简写、`align-self` 单独调对齐。

### `flex: 1` 到底等于什么？为什么布局里到处都在用？

`flex: 1` 展开是 `flex: 1 1 0%`，对应 `flex-grow: 1; flex-shrink: 1; flex-basis: 0%`。意思是：起始尺寸按 0 算，剩余空间按比例分配，空间不够也按比例收缩。

它和 `width: 100%` 不是一回事。`width: 100%` 是直接占满父容器，多个子项同时写 `width: 100%` 会互相挤掉；`flex: 1` 是参与剩余空间的瓜分，多个子项各写 `flex: 1` 会均分剩余空间。

常见场景是：**某一项需要占据剩余宽度**。

```css
.row { display: flex; }
.row .label { width: 100px; }   /* 固定 */
.row .content { flex: 1; }      /* 占据剩余宽度 */
```

容易出现的问题：`flex: 1` 配合超长文本时，子项会被内容撑开并突破 flex 的收缩规则，需要加 `min-width: 0` 才能正常收缩。这个细节可以单独作为一个问题理解。

### 为什么 flex 子项里文字溢出时，需要加 `min-width: 0`？

flex 子项的默认 `min-width` 是 `auto`，意思是"至少要容纳内容的最小宽度"。一段不能换行的长文本，最小宽度就是这段文本本身的宽度，结果就是子项被撑开、`flex-shrink` 失效、整行溢出。

加 `min-width: 0` 把这个保护拆掉，文字才能正常截断或者换行：

```css
.row { display: flex; }
.row .text {
  flex: 1;
  min-width: 0;       /* 关键 */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

同理，flex 容器在垂直方向用 `min-height: 0`，grid 子项也有类似规则。

### 介绍一下 Grid 网格布局。

Grid 是二维布局模型，适合同时处理行和列。

它的优势：

1. 能清晰定义网格区域。
2. 对复杂页面布局更直观。
3. 比传统 float 和定位方案更易维护。

更适合的场景：

1. 后台首页卡片布局
2. 仪表盘
3. 图片墙
4. 复杂内容分区页面

如果布局同时涉及横向和纵向控制，Grid 往往比 Flex 更合适。

### Grid 常用属性有哪些？

常见 Grid 容器属性有：

1. `grid-template-columns`
2. `grid-template-rows`
3. `gap`
4. `justify-items`
5. `align-items`
6. `grid-template-areas`

常见 Grid 项目属性有：

1. `grid-column`
2. `grid-row`
3. `justify-self`
4. `align-self`
5. `grid-area`

Grid 的优势在于可以同时处理行和列。

### Flex 和 Grid 应该怎么选？

可以这样理解：

1. **Flex**
   - 更适合一维布局
   - 重点解决一行或一列上的排列、对齐和空间分配

2. **Grid**
   - 更适合二维布局
   - 重点解决行列同时控制的问题

可以概括成：

> 一维布局优先 Flex，二维布局优先 Grid。

### 什么是响应式设计？响应式设计的基本原理是什么？如何做？

响应式设计指的是同一套页面能够适配不同尺寸的设备和屏幕。

基本原理：

1. 使用流式布局。
2. 使用弹性图片和相对单位。
3. 使用媒体查询针对不同断点调整样式。

常见处理有这些：

1. `@media` 媒体查询。
2. `rem`、`vw`、百分比布局。
3. Flex、Grid 等现代布局。
4. 图片自适应和响应式图片。

核心目标不是“所有设备表面上完全一样”，而是“不同设备下都可用、可读、可操作”。

### 如何理解流式布局、自适应布局、响应式布局？

这三个概念容易混淆。

1. **流式布局**
   - 主要依赖百分比等相对单位
   - 页面会随视口自然缩放

2. **自适应布局**
   - 针对几个固定尺寸写多套样式
   - 到某个断点时切换布局

3. **响应式布局**
   - 综合使用流式布局、媒体查询、弹性图片、现代布局等方案
   - 让页面对不同设备都能较好响应

直接理解成：

1. 流式更偏自然缩放
2. 自适应更偏几个固定版本
3. 响应式是更完整的适配策略

### 什么是媒体查询？常见的使用场景有哪些？

媒体查询是根据设备或视口条件动态应用不同样式的机制。

常见条件有：

1. 宽度和高度
2. 横屏和竖屏
3. 分辨率
4. 打印设备

更适合的场景：

1. 响应式布局断点切换。
2. 不同屏幕尺寸下调整字号、间距和布局。
3. 打印样式单独控制。
4. 横竖屏样式适配。

### 如何实现单行／多行文本溢出的省略样式？

单行省略常见写法：

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

多行省略常见写法：

```css
display: -webkit-box;
-webkit-line-clamp: 2;
-webkit-box-orient: vertical;
overflow: hidden;
```

其中多行省略依赖特定浏览器实现，项目里要关注兼容性。

写完整一点就是这样：

1. 单行省略

```html
<div class="ellipsis-one">这是一段很长很长很长的文本内容</div>
```

```css
.ellipsis-one {
  width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

2. 多行省略

```html
<div class="ellipsis-multi">这是一段不少不少不少不少不少不少不少不少不少不少的文本内容</div>
```

```css
.ellipsis-multi {
  width: 200px;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}
```

多行省略麻烦的地方就在这里：

多行省略不像单行那样有统一标准写法，不少场景仍依赖浏览器私有实现，所以兼容性和可控性要单独验证。

### 让 Chrome 支持小于 12px 的文字方式有哪些？区别是什么？

Chrome 默认对最小字体有限制，常见处理方式有：

1. 使用 `transform: scale()` 缩放。
2. 使用图片或 SVG 渲染小号文字。
3. 在高分屏场景下配合缩放方案实现视觉尺寸更小。

常见区别：

1. `scale` 实现简单，但可能影响清晰度和布局计算。
2. SVG 更灵活，但不适合普通正文文本。

开发里不建议滥用极小字号，因为这通常会损害可读性。

### CSS 如何画一个三角形？原理是什么？

常见的写法是利用边框。

原理是：一个宽高为 0 的元素，设置不同方向的边框颜色，只有一个方向显示颜色，其余为透明，就能形成三角形。

示例：

```css
.triangle {
  width: 0;
  height: 0;
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  border-bottom: 20px solid red;
}
```

如果要把示例写完整，可以这样：

```html
<div class="triangle"></div>
```

```css
.triangle {
  width: 0;
  height: 0;
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  border-bottom: 20px solid red;
}
```

如果继续追问“为什么这样能形成三角形”，可以回答：

因为元素本身宽高为 0，真正显示出来的是四条边框，透明的三条边被隐藏，只保留一条有颜色的边，因此视觉上就形成了三角形。

## 五、CSS3、动画与性能

### CSS3 新增了哪些新特性？

按几个能力维度梳理：

- **视觉效果**：`border-radius`、`box-shadow`、`text-shadow`、渐变（`linear-gradient` / `radial-gradient` / `conic-gradient`）、`filter`、`backdrop-filter`、`clip-path`、多背景。
- **变换与动画**：`transform`、`transition`、`animation` + `@keyframes`。
- **布局**：`flex`、`grid`、`multi-column`、`object-fit`。
- **适配**：媒体查询、视口单位、`@supports`。
- **新单位与函数**：`rem` / `vw` / `vh`、`calc()` / `min()` / `max()` / `clamp()`。
- **自定义属性**：CSS Variables（`--brand-color`）。

后续这几年还在快速加新东西，下面单独列几个高频考点。

### `:is()` / `:where()` / `:has()` 都怎么用？

近几年规范新增的几个伪类，能少写不少重复嵌套，面试问得越来越多。

**`:is()`**：取一组选择器的并集，整体优先级取最高那个。

```css
/* 等价于 h1 a, h2 a, h3 a */
:is(h1, h2, h3) a { color: red; }
```

**`:where()`**：和 `:is()` 一模一样，唯一区别是优先级永远是 0。设计系统里用得不少——库内部样式用 `:where()` 包一层，外部覆盖时不用打 `!important`。

```css
:where(.btn) { padding: 8px 16px; }  /* 优先级 0，随便覆盖 */
```

**`:has()`**：父选择器，终于可以"根据子元素选父元素"了。

```css
/* 有图的卡片加边框 */
.card:has(img) { border: 1px solid #eee; }

/* 表单里有 invalid 项时，submit 按钮置灰 */
form:has(:invalid) button[type=submit] { opacity: .5; }
```

`:has()` 是真正改变 CSS 写法的能力，以前要靠 JS 加 class 来实现的逻辑，现在一行 CSS 完成。Safari 15.4 起、Chrome 105 起支持。

### 容器查询 `@container` 是什么？和媒体查询有什么区别？

媒体查询按**视口**判断，但组件经常出现在不同宽度的容器里——同一个卡片放在侧栏和主内容区想要不同布局，媒体查询力不从心。

容器查询按**父容器**的尺寸决定样式：

```css
.card-wrap { container-type: inline-size; }

@container (min-width: 400px) {
  .card { display: grid; grid-template-columns: 120px 1fr; }
}
```

父容器宽度超过 400px，里面的卡片就走两栏布局，跟整个页面宽度无关。组件化时代真正意义上的响应式方案。Chrome 105、Safari 16 起广泛可用。

### CSS 函数 `calc()` / `min()` / `max()` / `clamp()`

- `calc()`：算术运算，混用单位是价值。`width: calc(100% - 80px)` 这种以前要靠 JS。
- `min(a, b)`：取最小值。`width: min(800px, 90vw)`——大屏不超过 800，小屏跟着 vw 缩。
- `max(a, b)`：取最大值。
- `clamp(min, val, max)`：在 min 和 max 之间夹一个值。响应式字号神器：`font-size: clamp(14px, 2vw, 18px)`，小屏不小于 14，大屏不大于 18，中间跟视口走。

四个函数能省掉大量媒体查询断点。

### `content-visibility` 是什么？为什么能优化渲染？

`content-visibility: auto` 告诉浏览器：这个元素如果当前不在视口内，可以跳过 Layout 和 Paint，等它滚动到附近再实际渲染。

```css
.long-list-item {
  content-visibility: auto;
  contain-intrinsic-size: 200px;  /* 给个尺寸占位，避免滚动条乱跳 */
}
```

长列表、长文档、大量卡片堆叠的页面，加上这两行性能提升明显。

### CSS `contain` 属性

`contain` 告诉浏览器某个元素的内部变化不会影响外部，浏览器就可以把它当独立单元来优化。

- `contain: layout`：内部布局变化不会触发外部 reflow
- `contain: paint`：内部不会绘制到边界外
- `contain: size`：自身尺寸不依赖内部内容
- `contain: content`：等于 `layout + paint + style`
- `contain: strict`：等于 `layout + paint + size + style`，最强隔离

复杂组件、独立卡片、第三方嵌入区域加 `contain: content`，能减少不少无效渲染。

### scroll-snap 怎么实现滚动吸附？

CSS 原生滚动吸附，适合轮播、Tab 切换、长列表分页等场景，不用监听 scroll：

```css
.scroller {
  scroll-snap-type: x mandatory;
  overflow-x: auto;
  display: flex;
}
.scroller > section {
  scroll-snap-align: start;
  flex: 0 0 100%;
}
```

`scroll-snap-type` 在容器上声明方向和强制程度（`mandatory` 强制吸附，`proximity` 接近时才吸），子项用 `scroll-snap-align` 声明吸附点。整体体验流畅，而且不需要 JS。

### CSS3 动画有哪些？

CSS 动画常见分为两类：

1. `transition`：用于状态切换时的过渡动画。
2. `animation` + `@keyframes`：用于定义完整关键帧动画。

两者区别：

1. `transition` 更适合 hover、展开收起等简单过渡。
2. `animation` 更适合循环动画、复杂时间线动画。

### 怎么理解回流和重绘？什么场景下会触发？

**回流（reflow / layout）**：元素的几何信息（位置、尺寸）变了，浏览器要重新算整个或部分布局树。开销最大。

**重绘（repaint）**：元素的外观属性（颜色、背景、阴影）变了，但布局没动，浏览器只需重新画一遍像素。比回流便宜。

关系：回流必然导致重绘，重绘不一定引发回流。再补一个最便宜的——只改 `transform`/`opacity` 走合成层，连 Paint 都跳过。

触发回流的常用操作：

- 改尺寸、位置：`width`、`height`、`margin`、`padding`、`top/left`、`font-size`
- 增删 DOM、修改 textContent
- 改 `display`、`position`、`float`
- 窗口 resize、调字体
- **读取强制同步布局的属性**：`offsetTop/Width/Height`、`scrollTop`、`getComputedStyle`、`getBoundingClientRect`——这些会强制浏览器刷新待处理的布局变更（layout thrashing）

优化思路就两条：能合并的样式改动一次性做（写一个 class 比连写五行 style 强），读写分离（先把所有 offsetXX 读完，再统一写样式），避免在循环里交替读写。

### 什么是合成层（Compositing Layer）？和性能优化有什么关系？

浏览器把页面拆成多个图层独立绘制，最后由合成器（compositor）拼到一起。每个合成层都能交给 GPU 直接处理，移动、缩放、透明度变化不用 CPU 重新画。

什么情况会被提升为合成层（俗称硬件加速）：

- `transform: translate3d/translateZ` 或 `will-change: transform`
- `opacity` 小于 1 且有动画
- `<video>`、`<canvas>`、WebGL
- 3D `transform`
- `position: fixed` + 滚动
- `filter`、`backdrop-filter`

合成层的好处：动画走 GPU 不卡主线程，避免大范围重绘。

但**层不是越多越好**：每层都占内存（宽 × 高 × 4 字节），层之间还要排序合成；过度使用 `will-change` 会显著增加 GPU 显存压力，反而掉帧。原则是：如果需要做动画的元素加，做完了就移除。Chrome DevTools 的 Layers 面板可以看到当前页面有多少层。

### 如果要做优化，CSS 提高性能的方法有哪些？

CSS 优化不是把样式表压小几 KB 这种事，重点是减少浏览器在渲染流水线上的开销。浏览器把一帧画出来要走 Style → Layout → Paint → Composite，写得糟糕的 CSS 会让这条流水线变重，60fps 就保不住。

按渲染阶段拆开看：

**样式计算阶段**：选择器层级别太深，比如 `.list .item .row .col a span` 这种，浏览器要从右往左一路匹配上去，DOM 一多就慢。日常写到三层基本就到顶了。

**Layout 阶段**：改 `width / height / margin / top / left / font-size` 这类属性会触发 reflow，整棵子树都要重新算一遍坐标。批量改样式时尽量一次性改完（比如改 class 而不是逐个改 style 属性），避免读写交替（读 offsetHeight 会强制同步布局，俗称 layout thrashing）。

**Paint 阶段**：大面积阴影（`box-shadow: 0 0 50px`）、滤镜（`filter: blur`）、复杂渐变这些都很贵，能用 transform 替代就替代。

**Composite 阶段**：只改 `transform` 和 `opacity` 这两个属性，可以直接跳过 Layout 和 Paint，由 GPU 合成。这就是为什么所有动画推荐都告诉你"用 transform 不要用 left/top"。

**其他**：去掉无效样式、按页面拆 CSS 分包、关键 CSS 内联、`@font-face` 配 `font-display: swap` 避免 FOIT。

短一点说：CSS 优化的本质还是让浏览器渲染流水线少走几步，而不是文件体积本身。

### CSS 动画为什么优先使用 transform 和 opacity？

回到上面那张渲染流水线图。改 `top` 会触发 Layout → Paint → Composite 三步，改 `transform` 直接跳到 Composite 这一步，整个流程只剩 GPU 把图层挪一下位置。`opacity` 同理，只是合成层透明度调整，不影响布局也不重绘。

落到具体场景就是：

- 元素移动：用 `transform: translate()` 而不是 `left/top`
- 缩放：用 `transform: scale()` 而不是改 `width/height`
- 旋转：用 `transform: rotate()`
- 显隐过渡：用 `opacity` 而不是 `display`（display 不能补间）

配合 `will-change: transform` 可以提前把元素提到合成层，但用完最好移除，常驻会占显存。

### 如何使用 CSS 完成视差滚动效果？

视差滚动是指不同层元素在滚动时以不同速度移动，形成空间层次感。

实现方式常见有：

1. `background-attachment: fixed`
2. 不同元素配合 `transform: translateY()`
3. 结合 JavaScript 监听滚动动态修改位置

如果需要更流畅的效果，通常会结合 `requestAnimationFrame` 和 GPU 加速属性来优化。

### 说说对 CSS 预编语言的理解？有哪些区别？

CSS 预编语言是对原生 CSS 的增强，提供变量、嵌套、混入、函数、模块化等能力，提高样式开发效率。

常见预编语言有：

1. Sass/SCSS
2. Less
3. Stylus

它们的共同点：

1. 支持变量
2. 支持嵌套
3. 支持逻辑复用

差异主要看：

1. 语法风格不同。
2. 生态和团队习惯不同。
3. Sass/SCSS 在工程化项目中更常见。

预编语言解决的是样式开发效率问题，不是浏览器运行时性能问题。

### CSS Modules、BEM、CSS-in-JS 分别解决什么问题？

三个都在解决同一个老问题：**全局样式互相冲突**。但路线完全不同。

**BEM**：纯命名约定。`block__element--modifier` 这种格式，靠人为约束让类名几乎不会重复。

```html
<div class="card card--featured">
  <h2 class="card__title">标题</h2>
  <button class="card__action card__action--primary">买</button>
</div>
```

零工具依赖，可读性也不错。缺点是写起来啰嗦，重构时改类名一片片地改。

**CSS Modules**：构建时把类名加 hash，变成 `Card_title__a1b2c`，物理上让它不可能撞。

```jsx
import s from './Card.module.css'
<div className={s.card}><h2 className={s.title}>...</h2></div>
```

写法跟普通 CSS 一样，作用域由打包器保证。Vue 的 `<style scoped>` 思路类似但用属性选择器实现。

**CSS-in-JS**：把样式直接写在 JS 里，运行时生成类名注入。styled-components、emotion 这类。

```jsx
const Card = styled.div`
  padding: 16px;
  background: ${props => props.featured ? 'gold' : 'white'};
`
```

优点是样式能直接读组件 props，主题切换、动态样式实现成本低。缺点是运行时开销、SSR 复杂、调试看的是 hash 类名。新的方案更多往零运行时方向走（Linaria、Vanilla Extract、Panda CSS、Tailwind），编译期就把样式生成出来。

选型上：

- 多人协作老项目、强约定：BEM
- React/Vue 工程化项目：CSS Modules 或 `<style scoped>`
- 动态主题、设计系统重的项目：CSS-in-JS 或 Tailwind

### `@supports` 特性查询怎么用？

跟 `@media` 同款语法，但查的不是设备，是浏览器是不是支持某个 CSS 特性：

```css
@supports (display: grid) {
  .layout { display: grid; }
}

@supports not (display: grid) {
  .layout { display: flex; }
}

/* 组合 */
@supports (display: grid) and (gap: 10px) { ... }
```

兼容渐进增强场景：能用新特性就用，不能用走老方案。比直接判断浏览器版本更可靠。

### 暗色主题怎么做？`prefers-color-scheme` 怎么用？

最现代的做法是把颜色抽到 CSS 变量，根元素根据 `prefers-color-scheme` 切一组值：

```css
:root {
  --bg: #fff;
  --text: #111;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #111;
    --text: #eee;
  }
}

body { background: var(--bg); color: var(--text); }
```

跟随系统设置自动切换。如果还要让用户手动切，给 `<html>` 加个 `data-theme="dark"` 属性，用属性选择器再覆盖一组变量即可。

配合 `color-scheme: light dark` 让浏览器原生控件（滚动条、表单元素）也跟着切。

## 六、高频场景题

### 如何实现一个三角形 tooltip，并带一个小箭头？

这是很典型的 CSS 场景题。

一般就是这么做：

1. tooltip 主体用普通容器实现
2. 小箭头通常用伪元素 + 边框三角形实现
3. 箭头通过绝对定位放到对应位置

示例：

```html
<div class="tooltip">提示内容</div>
```

```css
.tooltip {
  position: relative;
  display: inline-block;
  padding: 8px 12px;
  color: #fff;
  background: #333;
  border-radius: 6px;
}

.tooltip::after {
  content: "";
  position: absolute;
  left: 50%;
  bottom: -6px;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
  border-top: 6px solid #333;
}
```

---

### 如何实现一个宽高自适应的正方形？

常见方式有两种：

1. 使用 `aspect-ratio`
2. 老方案用 `padding-top: 100%`

现代推荐写法：

```css
.box {
  width: 200px;
  aspect-ratio: 1 / 1;
  background: #eee;
}
```

兼容旧方案：

```css
.box {
  width: 200px;
  height: 0;
  padding-top: 100%;
  background: #eee;
}
```

---

### 如何实现一个固定表头、内容区域滚动的表格？

常见思路：

1. 表头和内容拆成两个区域
2. 内容区域单独滚动
3. 保证列宽对齐

如果是简单场景，也可以使用：

1. 外层固定高度
2. `thead` 配合 `position: sticky`

例如：

```css
.table-wrap {
  max-height: 300px;
  overflow: auto;
}

th {
  position: sticky;
  top: 0;
  background: #fff;
}
```

---

### 如何实现一个吸顶效果？

吸顶效果常见有两种思路：

1. `position: sticky`
2. `position: fixed` + JavaScript 监听滚动

现代推荐优先使用：

```css
.header {
  position: sticky;
  top: 0;
  z-index: 10;
}
```

为什么优先用 `sticky`：

1. 实现简单
2. 不需要额外 JS
3. 更符合浏览器原生布局机制

---

### 如何实现一个圣杯布局或双飞翼布局？项目里还常用吗？

这个问题既涉及传统布局，也涉及工程选型判断。

可以分两层：

1. **原理上会**
   - 圣杯布局：父容器 `padding` 预留空间
   - 双飞翼布局：中间内容容器 `margin` 预留空间

2. **工程上少用了**
   - 现代项目更推荐 `Flex` 或 `Grid`
   - 语义更清晰，维护成本更低

更准确的表述是：

> 我理解圣杯和双飞翼的实现原理，但在现代项目里多半优先用 Flex 或 Grid。

---

### 如何实现一个左侧固定、右侧内容过长自动出现省略的列表项？

这是业务开发中很常见的布局问题。

关键点：

1. 外层使用 Flex
2. 左侧固定宽度
3. 右侧 `flex: 1`
4. 右侧文本元素要允许收缩，必要时加 `min-width: 0`

示例：

```html
<div class="item">
  <span class="label">标题：</span>
  <span class="value">这是一段很长很长很长很长的文本内容</span>
</div>
```

```css
.item {
  display: flex;
}

.label {
  width: 80px;
}

.value {
  flex: 1;
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

这里容易漏掉的是：`min-width: 0`。

---

### 如何实现一个等高列布局？

传统布局时代这是经典题，现在较简单的方法是：

1. `Flex`
2. `Grid`

例如 Flex：

```css
.wrap {
  display: flex;
}

.col {
  flex: 1;
}
```

因为 Flex 同一行中的项目天然可以拉伸到相同高度。

---

### 如何解决 1px 问题？

在高 DPR 屏幕下，CSS 的 `1px` 可能映射成多个物理像素，视觉上偏粗。

常见方案：

1. 伪元素 + `transform: scale()`
2. 使用 `0.5px`
3. `box-shadow` 模拟边框
4. SVG 线条

这种题一般会这么说：

> 1px 问题本质还是 CSS 像素和物理像素在高 DPR 下的映射差异。

---

### 如何做移动端适配？

高频回答通常包含这几类：

1. `meta viewport`
2. 相对单位：`rem`、`vw`
3. 弹性布局：`Flex`、`Grid`
4. 媒体查询
5. 响应式图片

如果只看现在更常用的方案：

1. 设计稿统一基准
2. 页面使用 `rem` 或 `vw`
3. 布局使用 Flex / Grid
4. 特殊断点再配合媒体查询

---

### 如何实现一个全屏遮罩层和居中弹窗？

常见用的写法就是：

1. 遮罩层 `position: fixed; inset: 0`
2. 弹窗用 Flex 或绝对定位居中

示例：

```html
<div class="mask">
  <div class="dialog">弹窗内容</div>
</div>
```

```css
.mask {
  position: fixed;
  inset: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.5);
}

.dialog {
  width: 320px;
  padding: 20px;
  background: #fff;
  border-radius: 8px;
}
```

---

### 如何让一个元素覆盖在另一个元素上方，但又不影响下层布局？

一般就是这么做：

1. 使用定位脱离普通文档流
2. 必要时配合层叠上下文和 `z-index`

比如：

```css
.parent {
  position: relative;
}

.badge {
  position: absolute;
  top: 0;
  right: 0;
  z-index: 2;
}
```

这种题最后看的还是：

1. 定位上下文
2. 文档流
3. 层叠上下文

### CSS 变量（Custom Properties）有什么价值？

CSS 变量不只是“少写几个颜色值”，它最大的价值是把样式里的可变配置抽出来。

常见用途：

1. 主题切换
2. 设计系统变量统一
3. 组件样式参数化

它比预处理器变量更特别的地方在于：它是运行时生效的，能和 DOM 结构、继承机制一起工作。

### `position: sticky` 为什么有时候不生效？

常见原因有：

1. 没有设置 `top` / `left` 等阈值
2. 祖先容器的滚动上下文不符合预期
3. 父容器高度不够，没有真正形成滚动空间
4. 某些 `overflow` 设置改变了滚动参考容器

这种问题最后还是在看：你是否理解 sticky 不是“永远吸顶”，而是“在特定滚动容器里达到阈值后吸附”。

### CSS 里的 `transform` 为什么经常会影响定位和层叠？

因为 `transform` 不只是视觉变换，它还可能创建新的层叠上下文，某些情况下还会影响固定定位元素的参考系。

所以不少“z-index 设很大却盖不上去”或者“fixed 看起来不再相对视口”的问题，根源往往都在这里。

## 附：常见代码实现

### 1. Flex 两栏自适应

```html
<div class="layout">
  <aside class="sidebar">左侧</aside>
  <main class="content">右侧自适应</main>
</div>
```

```css
.layout {
  display: flex;
}
.sidebar {
  width: 220px;
}
.content {
  flex: 1;
}
```

### 2. 经典 BFC 清除浮动

```css
.container {
  overflow: hidden;
}
.float-item {
  float: left;
}
```

### 3. 单行与多行省略

```css
.one-line {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.multi-line {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

### 4. 水平垂直居中

```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### 5. 经典三角形

```css
.triangle {
  width: 0;
  height: 0;
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  border-bottom: 20px solid #f66;
}
```

### 6. 伪元素清除浮动

```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

### 7. 固定底部按钮

```css
.footer-btn {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
}
```

### 8. 吸顶效果

```css
.sticky {
  position: sticky;
  top: 0;
  z-index: 10;
}
```

### 9. 画一个圆形

```css
.circle {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: #409eff;
}
```

### 10. 实现一个正方形

```css
.square {
  width: 120px;
  aspect-ratio: 1 / 1;
  background: #ddd;
}
```

### 11. 左固定右省略

```css
.row {
  display: flex;
}

.left {
  width: 80px;
}

.right {
  flex: 1;
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

### 12. 全屏遮罩层

```css
.mask {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
}
```

### 13. Grid 两列布局

```css
.grid {
  display: grid;
  grid-template-columns: 200px 1fr;
  gap: 16px;
}
```

### 14. 文字两端对齐

```css
.justify {
  text-align: justify;
}
```

### 15. 不规则换行时防止单词撑破容器

```css
.content {
  overflow-wrap: break-word;
  word-break: break-word;
}
```
