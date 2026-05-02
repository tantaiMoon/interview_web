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

`DOCTYPE` 的作用是告诉浏览器当前文档使用哪种 HTML 规范，从而决定采用什么渲染模式。

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

一句话总结：`defer` 强调“顺序执行”，`async` 强调“谁先下完谁先执行”。

### 常用的 meta 标签有哪些？

`meta` 标签主要用于描述页面元信息。

常见的有：

1. `charset`：设置字符编码，如 `UTF-8`。
2. `viewport`：控制移动端视口缩放。
3. `description`：页面描述，利于 SEO。
4. `keywords`：页面关键词。
5. `http-equiv`：模拟 HTTP 响应头，比如缓存控制、兼容模式。
6. `author`：作者信息。

移动端开发里最常见的是：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

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

不过要注意，HTML5 中元素的显示特性可以通过 CSS 修改，所以“块级/行内”更偏默认表现，而不是绝对属性。

### title 与 h1 的区别，b 与 strong 的区别，i 与 em 的区别？

这类题主要考语义和用途。

1. `title` 是浏览器标签页标题，也影响搜索结果标题；`h1` 是页面正文中的一级标题。
2. `b` 主要表示样式上的加粗；`strong` 表示语义上的强调，通常也会加粗显示。
3. `i` 主要表示样式上的斜体；`em` 表示语义上的强调，通常也会斜体显示。

面试时要强调：`strong` 和 `em` 更有语义价值，更利于无障碍和搜索引擎理解。

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

优点：

1. 内容隔离，互不影响。
2. 适合嵌入第三方页面、地图、文档、支付页。
3. 可以复用现有系统或独立子应用。

缺点：

1. 性能开销更大。
2. 不利于 SEO。
3. 调试复杂，通信麻烦。
4. 存在安全风险，需要配合 `sandbox`、`CSP` 等控制。

现代前端里，`iframe` 更适合做隔离嵌入，而不是常规业务布局手段。

### 浏览器乱码的原因是什么？如何解决？

乱码本质上是“文件编码”和“浏览器解析编码”不一致。

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

## 二、HTML5 与浏览器能力

### HTML5 有哪些更新？

HTML5 的更新很多，面试时建议按模块回答：

1. 语义化标签：`header`、`nav`、`article`、`section`、`footer`。
2. 媒体标签：`audio`、`video`。
3. 表单增强：新增输入类型和校验能力。
4. 图形能力：`canvas`、SVG 支持增强。
5. 存储能力：`localStorage`、`sessionStorage`。
6. 浏览器 API：拖拽、地理定位、Web Worker 等。
7. 更好的跨设备和多媒体支持。

回答时重点不是罗列，而是说明 HTML5 从“文档标记语言”进一步走向“富应用基础设施”。

HTML5 让 Web 从单纯展示静态文档，逐步具备了构建富交互、富媒体、可离线、可接近原生体验应用的基础能力。

### img 的 srcset 属性有什么作用？

`srcset` 用于根据不同屏幕密度或视口条件加载更合适的图片资源，核心目标是做响应式图片。

它的价值：

1. 高清屏加载更高分辨率图片。
2. 普通屏避免加载过大图片。
3. 平衡清晰度与性能。

典型场景是移动端适配和多端图片优化。

### 对 Web Worker 的理解是什么？

Web Worker 的作用是把耗时计算放到浏览器的后台线程执行，避免阻塞主线程。

它适合：

1. 大数据计算。
2. 文件解析。
3. 图片处理。
4. 长时间运行任务。

它的特点：

1. 不能直接操作 DOM。
2. 通过 `postMessage` 与主线程通信。
3. 适合 CPU 密集型任务，不适合简单 UI 逻辑。

一句话总结：Web Worker 是前端“多线程能力”的主要实现方式之一，但它只负责计算，不负责页面渲染。

### HTML5 的离线存储怎么使用，它的工作原理是什么？

这题通常指早期的 Application Cache，也会延伸到现代离线能力。

如果从面试角度回答，建议这样说：

1. 早期 HTML5 提供过 AppCache，通过清单文件声明需要缓存的资源。
2. 浏览器首次访问后会把资源缓存到本地，断网时仍可读取。
3. 但 AppCache 机制复杂、易出错，现已被废弃。
4. 现代更推荐使用 Service Worker + Cache Storage 实现离线缓存。

所以如果面试官问到这里，最好顺带补一句：现在实际项目里不会再推荐 AppCache，而是用 PWA 方案。

### 浏览器是如何对 HTML5 的离线存储资源进行管理和加载的？

如果从 AppCache 的历史机制回答：

1. 浏览器根据清单文件缓存指定资源。
2. 再次访问时先检查清单是否更新。
3. 若资源未变，直接用本地缓存。
4. 若清单变化，则重新下载并更新缓存。

如果从现代实现回答：

1. Service Worker 拦截请求。
2. 优先走缓存或网络。
3. 根据缓存策略更新资源。
4. 最终实现离线访问、资源预缓存和增量更新。

面试里最好指出：这道题如果放到现在，主流答案应当转向 Service Worker。

如果你想把这题答得更完整，可以继续展开成下面这种说法：

1. 早期 HTML5 的离线存储方案是 AppCache，它会通过 manifest 清单文件声明哪些资源要缓存。
2. 浏览器第一次访问页面时，会把 manifest 中列出的 HTML、CSS、JS、图片等资源下载到本地。
3. 后续再次访问时，浏览器会先检查 manifest 文件有没有变化。
4. 如果 manifest 没变，就优先直接使用本地缓存资源；如果 manifest 变了，就会重新拉取资源并更新缓存。
5. 这个机制的问题在于更新不直观、缓存粒度粗、调试困难，所以后来被废弃了。

如果从现代浏览器角度回答，更推荐这样说：

1. 现在主流离线能力是 Service Worker + Cache Storage。
2. 浏览器先注册 Service Worker，之后由它拦截页面请求。
3. 请求到来后，Service Worker 可以按策略决定是优先走缓存、优先走网络，还是缓存和网络结合。
4. 浏览器中的离线资源因此不再是“被动缓存”，而是“可编程缓存”，开发者可以自己控制缓存哪些资源、何时更新、何时清理。
5. 这让浏览器具备了更稳定的离线访问、资源预缓存、增量更新和弱网优化能力。

如果面试官问“浏览器怎么加载”，可以再补一句：

浏览器本质上是在发起资源请求时，先经过 Service Worker 这一层拦截，再由它决定返回本地缓存内容还是网络内容，所以离线页面看起来像正常加载，但底层资源来源已经从网络切到了本地缓存。

### Canvas 和 SVG 的区别是什么？

两者都能绘图，但原理不同：

1. Canvas 是基于像素绘图，适合频繁重绘。
2. SVG 是基于矢量和 DOM 节点，适合结构化图形。

适用场景：

1. Canvas：游戏、图表、粒子动画、图像处理。
2. SVG：图标、流程图、可缩放图形、交互式图表。

一句话总结：Canvas 更适合高频绘制，SVG 更适合可交互、可维护的矢量图形。

### 渐进增强和优雅降级的区别是什么？

这两个概念本质上是不同的兼容策略。

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

现代复杂拖拽场景里，很多项目也会直接使用成熟拖拽库来保证兼容性和体验。

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

可以通过 `box-sizing` 控制：

1. `content-box`：标准盒模型。
2. `border-box`：更常用，布局计算更直观。

### CSS 选择器有哪些？优先级如何？哪些属性可以继承？

常见选择器有：

1. 标签选择器
2. 类选择器
3. id 选择器
4. 后代、子代、相邻兄弟选择器
5. 属性选择器
6. 伪类选择器
7. 伪元素选择器

优先级大致是：

1. `!important`
2. 行内样式
3. id 选择器
4. 类、属性、伪类
5. 标签、伪元素
6. 通配符和继承样式

常见可继承属性主要是文字相关属性，比如：

1. `color`
2. `font-size`
3. `font-family`
4. `line-height`
5. `text-align`
6. `visibility`

布局相关属性如 `margin`、`padding`、`border`、`width` 通常不会继承。

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

一句话区分：伪类强调“元素状态”，伪元素强调“元素内容片段或额外生成内容”。

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

这是前端适配的基础概念。

1. 设备像素：屏幕物理像素点。
2. CSS 像素：浏览器中的逻辑像素。
3. 设备独立像素：一种抽象单位，便于跨设备统一显示。
4. DPR：设备像素比，通常表示 `设备像素 / CSS 像素`。
5. PPI：每英寸像素数，反映屏幕精细度。

常见结论：

1. DPR 越高，同样的 CSS 尺寸会映射更多物理像素。
2. 高清屏容易出现 1px 边框、图片模糊等适配问题。

如果面试官继续追问“那这些问题怎么解决”，可以这样展开：

1. 1px 边框问题
   在高 DPR 屏幕下，`1px` 的 CSS 边框可能会映射成多个物理像素，视觉上显得偏粗。
   常见解决方式有：
   1. 使用伪元素配合 `transform: scale()` 做 hairline 方案。
   2. 使用 `box-shadow` 模拟细线。
   3. 使用 SVG 绘制线条。
   4. 某些场景下直接使用 `0.5px`，但要注意兼容性。

2. 图片模糊问题
   图片模糊的本质是图片资源分辨率不够，高 DPR 设备会把低分辨率图片放大显示。
   常见解决方式有：
   1. 提供 2x、3x 高清图。
   2. 使用 `srcset` 做响应式图片。
   3. 图标优先使用 SVG。
   4. 背景图按不同 DPR 提供不同资源。
   5. 避免把小图强行放大显示。

一句话总结：高清屏适配的核心就是区分“CSS 像素”和“物理像素”，边框问题重点解决 hairline，图片问题重点提供更高分辨率资源。

### CSS 中有哪些方式可以隐藏页面元素？区别是什么？

常见隐藏方式有：

1. `display: none`：元素不占空间，不参与渲染。
2. `visibility: hidden`：元素占空间，但不可见。
3. `opacity: 0`：元素透明，但仍占空间，也可响应事件。
4. 绝对定位移出视口。
5. `clip-path` 或无障碍隐藏方案。

面试时重点说区别：

1. 是否占据文档流空间。
2. 是否能响应事件。
3. 是否会被屏幕阅读器读取。
4. 是否触发回流和重绘。

### 谈谈你对 BFC 的理解。

BFC 即块级格式化上下文，它是页面中的一个独立布局环境，内部元素布局不会影响外部。

BFC 常见作用：

1. 清除浮动。
2. 防止外边距重叠。
3. 实现两栏布局，避免文字环绕浮动元素。

常见触发方式：

1. `overflow: hidden/auto/scroll`
2. `display: flow-root`
3. `position: absolute/fixed`
4. `float` 不为 `none`
5. `display: inline-block`

一句话总结：BFC 是解决布局隔离问题的重要机制。

### `@import` 和 `link` 引入 CSS 的区别是什么？

两者都能引入样式，但行为和适用场景不同。

主要区别：

1. `link` 属于 HTML 标签，浏览器会并行加载资源。
2. `@import` 属于 CSS 语法，通常要等当前 CSS 解析到该语句时再继续加载。
3. `link` 兼容性和性能通常更好。
4. `@import` 除了引入样式，还可以配合媒体查询做条件导入。

实际项目中，更推荐使用 `link` 引入主样式文件，`@import` 更适合特定样式组织场景。

## 四、布局与适配

### CSS 中 position 有哪些取值？它们的区别是什么？

常见取值有：

1. `static`：默认定位，不参与偏移。
2. `relative`：相对自身原位置偏移，仍占据原文档流空间。
3. `absolute`：脱离文档流，相对最近的定位祖先定位。
4. `fixed`：脱离文档流，相对视口定位。
5. `sticky`：在达到阈值前表现像相对定位，达到阈值后表现像吸附定位。

面试时重点要说清楚：

1. 是否脱离文档流。
2. 相对谁定位。
3. 常见使用场景，比如弹层、吸顶、角标、回到顶部按钮等。

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

常见实现代码可以这样写：

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

如果面试官追问“最推荐哪个”，可以回答：

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

常见代码实现可以这样写：

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

如果面试官继续追问“什么是圣杯布局和双飞翼布局”，可以这样展开：

1. 圣杯布局
   指的是左右两栏固定宽度，中间栏自适应，并且中间内容在 HTML 结构中优先渲染。
   它的经典实现通常基于：
   1. 三列都浮动。
   2. 父容器设置左右 `padding` 预留空间。
   3. 中间栏宽度 `100%`。
   4. 左右栏通过负 `margin` 和相对定位回到两侧。

2. 双飞翼布局
   目标和圣杯布局一样，也是三栏布局、中间自适应、中间内容优先渲染。
   它和圣杯布局的主要区别是：
   1. 双飞翼布局通常不通过父容器 `padding` 预留左右空间。
   2. 而是给中间栏内部再包一层内容容器，通过内容容器的 `margin` 为左右两栏腾出位置。

两者区别可以总结为：

1. 圣杯布局是“父容器留 padding 空间”。
2. 双飞翼布局是“中间内容自己留 margin 空间”。

它们的共同点是：

1. 都是为了解决三栏布局中间自适应问题。
2. 都强调中间主内容优先加载。
3. 都是早期在 Flex 和 Grid 普及前非常经典的布局方案。

如果面试时需要一句话收尾，可以说：

圣杯和双飞翼本质上都是传统 float 时代解决三栏自适应布局的经典方案，现在工程里通常更推荐直接使用 Flex 或 Grid。

经典实现代码可以这样写：

1. 圣杯布局

```html
<div class="holy-grail">
  <div class="main">中间</div>
  <div class="left">左侧</div>
  <div class="right">右侧</div>
</div>
```

```css
.holy-grail {
  min-width: 600px;
  padding: 0 200px 0 150px;
}

.holy-grail .main,
.holy-grail .left,
.holy-grail .right {
  float: left;
  min-height: 200px;
}

.holy-grail .main {
  width: 100%;
  background: #ddd;
}

.holy-grail .left {
  width: 150px;
  margin-left: -100%;
  position: relative;
  left: -150px;
  background: #f66;
}

.holy-grail .right {
  width: 200px;
  margin-left: -200px;
  position: relative;
  right: -200px;
  background: #66f;
}
```

2. 双飞翼布局

```html
<div class="double-wing">
  <div class="main-wrap">
    <div class="main">中间</div>
  </div>
  <div class="left">左侧</div>
  <div class="right">右侧</div>
</div>
```

```css
.double-wing {
  min-width: 600px;
}

.double-wing .main-wrap,
.double-wing .left,
.double-wing .right {
  float: left;
  min-height: 200px;
}

.double-wing .main-wrap {
  width: 100%;
}

.double-wing .main {
  margin: 0 200px 0 150px;
  background: #ddd;
}

.double-wing .left {
  width: 150px;
  margin-left: -100%;
  background: #f66;
}

.double-wing .right {
  width: 200px;
  margin-left: -200px;
  background: #66f;
}
```

代码理解重点：

1. 两种布局都让中间栏 `width: 100%`，这样主内容能先占满一行。
2. 左右两栏再通过负 `margin` 拉回同一行。
3. 圣杯布局靠父容器 `padding` 给左右两栏预留空间。
4. 双飞翼布局靠中间内容区自身的 `margin` 给左右两栏让位。

### 说说 Flexbox（弹性盒布局模型）以及适用场景。

Flex 是一维布局模型，适合处理行或列方向的排列。

核心概念：

1. 容器属性：`flex-direction`、`justify-content`、`align-items`、`flex-wrap`
2. 子项属性：`flex`、`align-self`、`order`

适用场景：

1. 水平或垂直居中
2. 两栏三栏布局
3. 导航栏、按钮组
4. 空间分配和对齐问题

一句话总结：Flex 最适合解决一维布局和对齐问题。

这里的“一维布局”指的是：Flex 更擅长处理“单一方向”的排列，也就是主要沿着一行或者一列去分配空间和控制对齐。

可以把它具体理解成下面几类问题：

1. 一行元素怎么水平排列。
2. 一列元素怎么垂直排列。
3. 多个子元素之间怎么分配剩余空间。
4. 子元素在主轴和交叉轴上如何对齐。
5. 某个元素如何单独靠左、靠右、居中或拉伸。

典型场景包括：

1. 导航栏左右分布。
2. 表单项一行内对齐。
3. 卡片内按钮区域左右排列。
4. 列表项中头像、文字、操作按钮横向排布。
5. 容器内单个元素水平垂直居中。
6. 左侧固定、右侧自适应布局。

如果和 Grid 对比，可以这样说：

1. Flex 更像是在处理“一条线上的元素关系”。
2. Grid 更像是在处理“行和列同时存在的二维网格关系”。

所以面试时一句更完整的话可以说：

Flex 最适合解决一行或一列上的元素排列、剩余空间分配、主轴交叉轴对齐这些一维布局问题，而如果页面需要同时精确控制行和列，通常 Grid 会更合适。

### Flex 容器和项目常用属性有哪些？

Flex 容器常用属性有：

1. `flex-direction`
2. `flex-wrap`
3. `flex-flow`
4. `justify-content`
5. `align-items`
6. `align-content`

Flex 项目常用属性有：

1. `order`
2. `flex-grow`
3. `flex-shrink`
4. `flex-basis`
5. `flex`
6. `align-self`

如果面试官追问 `flex: 1`，可以回答：它通常可以理解为让元素参与剩余空间分配，常见等价含义可近似理解为 `flex-grow: 1`，但完整语义还涉及 `flex-shrink` 和 `flex-basis`。

如果要把 `flex: 1` 讲得更完整，可以这样展开：

1. `flex` 是 `flex-grow`、`flex-shrink`、`flex-basis` 的简写。
2. `flex: 1` 在常见场景下通常可以近似理解为：
   `flex-grow: 1; flex-shrink: 1; flex-basis: 0%;`
3. 它表示当前元素可以增长、可以收缩，并且初始基准尺寸按 0 来参与剩余空间分配。

这里三个属性的含义分别是：

1. `flex-grow`：有剩余空间时，元素按什么比例扩张。
2. `flex-shrink`：空间不足时，元素按什么比例收缩。
3. `flex-basis`：在分配剩余空间之前，元素以多大基础尺寸参与计算。

为什么很多布局里喜欢写 `flex: 1`？

因为它非常适合“某一项自适应占满剩余空间”的场景，比如：

1. 左侧固定宽度，右侧内容自适应。
2. 多列布局中间区域自动撑开。
3. 列表项中间文本区域自动占满剩余宽度。

例如：

```html
<div class="wrap">
  <div class="left">左侧固定</div>
  <div class="right">右侧自适应</div>
</div>
```

```css
.wrap {
  display: flex;
}

.left {
  width: 200px;
}

.right {
  flex: 1;
}
```

这里 `right` 的含义就是：把 `left` 占掉 200px 之后，剩余空间都交给 `right`。

如果继续追问 `flex: 1` 和 `width: 100%` 的区别，可以这样回答：

1. `width: 100%` 是直接指定宽度。
2. `flex: 1` 是参与 Flex 规则下的剩余空间分配。
3. 在有多个 Flex 子项时，`flex: 1` 更强调“按比例分空间”，而不是单纯“设成 100%”。

面试时一句话总结可以说：

`flex: 1` 本质上不是简单的“宽度 100%”，而是让当前元素在 Flex 布局中以可伸缩的方式参与剩余空间分配。

### 介绍一下 Grid 网格布局。

Grid 是二维布局模型，适合同时处理行和列。

它的优势：

1. 能清晰定义网格区域。
2. 对复杂页面布局更直观。
3. 比传统 float 和定位方案更易维护。

常见场景：

1. 后台首页卡片布局
2. 仪表盘
3. 图片墙
4. 复杂内容分区页面

如果布局同时涉及横向和纵向控制，Grid 往往比 Flex 更合适。

### 什么是响应式设计？响应式设计的基本原理是什么？如何做？

响应式设计指的是同一套页面能够适配不同尺寸的设备和屏幕。

基本原理：

1. 使用流式布局。
2. 使用弹性图片和相对单位。
3. 使用媒体查询针对不同断点调整样式。

常见做法：

1. `@media` 媒体查询。
2. `rem`、`vw`、百分比布局。
3. Flex、Grid 等现代布局。
4. 图片自适应和响应式图片。

核心目标不是“所有设备看起来完全一样”，而是“不同设备下都可用、可读、可操作”。

### 什么是媒体查询？常见的使用场景有哪些？

媒体查询是根据设备或视口条件动态应用不同样式的机制。

常见条件有：

1. 宽度和高度
2. 横屏和竖屏
3. 分辨率
4. 打印设备

常见场景：

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

其中多行省略依赖特定浏览器实现，实际项目中要关注兼容性。

完整示例代码可以这样写：

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
<div class="ellipsis-multi">这是一段很多很多很多很多很多很多很多很多很多很多的文本内容</div>
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

如果面试官追问“多行省略为什么麻烦”，可以补一句：

多行省略不像单行那样有统一标准写法，很多场景仍依赖浏览器私有实现，所以兼容性和可控性要单独验证。

### 让 Chrome 支持小于 12px 的文字方式有哪些？区别是什么？

Chrome 默认对最小字体有限制，常见处理方式有：

1. 使用 `transform: scale()` 缩放。
2. 使用图片或 SVG 渲染小号文字。
3. 在高分屏场景下配合缩放方案实现视觉尺寸更小。

常见区别：

1. `scale` 实现简单，但可能影响清晰度和布局计算。
2. SVG 更灵活，但不适合普通正文文本。

实际开发里不建议滥用极小字号，因为这通常会损害可读性。

### CSS 如何画一个三角形？原理是什么？

最常见做法是利用边框。

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

如果想补完整示例，可以这样写：

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

如果面试官继续追问“为什么这样能形成三角形”，可以回答：

因为元素本身宽高为 0，真正显示出来的是四条边框，透明的三条边被隐藏，只保留一条有颜色的边，因此视觉上就形成了三角形。

## 五、CSS3、动画与性能

### CSS3 新增了哪些新特性？

常见回答可以从以下几类展开：

1. 圆角：`border-radius`
2. 阴影：`box-shadow`、`text-shadow`
3. 渐变：`linear-gradient`、`radial-gradient`
4. 变换：`transform`
5. 过渡：`transition`
6. 动画：`animation`
7. 弹性布局和网格布局
8. 媒体查询
9. 边框图片、多背景等

面试时建议按“视觉效果、布局能力、动画能力、适配能力”来组织。

### CSS3 动画有哪些？

CSS 动画常见分为两类：

1. `transition`：用于状态切换时的过渡动画。
2. `animation` + `@keyframes`：用于定义完整关键帧动画。

两者区别：

1. `transition` 更适合 hover、展开收起等简单过渡。
2. `animation` 更适合循环动画、复杂时间线动画。

### 怎么理解回流和重绘？什么场景下会触发？

1. 回流：元素几何信息变化后，浏览器需要重新计算布局。
2. 重绘：元素样式变化但不影响布局时，浏览器重新绘制外观。

回流一定会引发重绘，重绘不一定引发回流。

常见触发回流的场景：

1. 元素尺寸、位置变化。
2. 增删 DOM。
3. 读取某些会强制刷新布局的属性。
4. 窗口尺寸变化。

优化关键是减少频繁 DOM 操作和强制同步布局。

### 如果要做优化，CSS 提高性能的方法有哪些？

常见优化手段：

1. 减少选择器层级，避免过度复杂匹配。
2. 避免频繁触发回流和重绘。
3. 动画优先使用 `transform` 和 `opacity`。
4. 合理拆分样式，减少无效 CSS。
5. 使用压缩、缓存、按需加载。
6. 避免大量阴影、滤镜等高开销效果。
7. 使用现代布局替代复杂 hack。

面试时要强调：CSS 性能优化核心是减少浏览器渲染成本，而不是单纯减少代码行数。

如果想把这句话讲得更具体，可以这样展开：

1. CSS 性能优化的重点，不是少写几行样式，而是尽量减少浏览器在渲染阶段的工作量。
2. 浏览器渲染页面时，通常要经历样式计算、布局、绘制、分层、合成这些步骤。
3. 如果 CSS 写法不合理，就可能让这些步骤变得更重，尤其是频繁触发回流和重绘时，页面就容易卡顿。
4. 所以 CSS 优化的本质，是让浏览器在“样式计算、布局、绘制”这些阶段少做事、快做事。

可以进一步拆成几个方向：

1. 减少样式计算成本
   不要写过深、过复杂的选择器，减少浏览器匹配样式时的无效计算。

2. 减少布局成本
   尽量避免频繁修改会影响布局的属性，比如 `width`、`height`、`margin`、`top`、`left`。

3. 减少绘制成本
   避免大量高开销样式，比如大面积阴影、滤镜、复杂渐变、模糊效果等。

4. 减少动画成本
   动画优先使用 `transform` 和 `opacity`，因为它们通常比直接改布局属性更高效。

5. 减少无效 CSS
   没用到的样式、重复样式、过大的样式文件不仅影响解析，也会增加维护成本。

如果面试时要一句话收尾，可以说：

CSS 性能优化的本质，不是单纯压缩文件，而是减少浏览器在样式计算、布局和绘制阶段的渲染成本。

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

差异主要体现在：

1. 语法风格不同。
2. 生态和团队习惯不同。
3. Sass/SCSS 在工程化项目中更常见。

面试时最好补一句：预编语言解决的是“样式开发效率问题”，而不是浏览器运行时性能问题。

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
