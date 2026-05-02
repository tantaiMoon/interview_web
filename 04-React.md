[toc]



## 一、React 基础认知

### 说说你对 React 的理解？有哪些特性？

React 是一个用于构建用户界面的 JavaScript 库，核心思想是组件化、声明式和单向数据流。

它的主要特性有：

1. 组件化开发，便于复用和维护。
2. 声明式 UI，开发者关注状态和视图映射关系。
3. 单向数据流，数据流向清晰，便于定位问题。
4. 虚拟 DOM 提高更新效率。
5. 生态完善，可以和路由、状态管理、构建工具组合成完整方案。

一句话总结：React 的本质是用组件和状态来组织 UI。

### 说说 Real DOM 和 Virtual DOM 的区别？优缺点是什么？

Real DOM 是浏览器中的真实 DOM 树，Virtual DOM 是对真实 DOM 的 JavaScript 对象描述。

区别在于：

1. 真实 DOM 直接对应浏览器页面结构。
2. 虚拟 DOM 是内存中的中间表示。
3. React 更新时先计算虚拟 DOM 差异，再把最小变更同步到真实 DOM。

虚拟 DOM 的优点：

1. 屏蔽底层 DOM 操作细节。
2. 便于跨平台渲染。
3. 配合 diff 降低不必要的真实 DOM 操作。

要注意，虚拟 DOM 不是“绝对更快”，它的核心价值是更可控、更易维护。

### 为什么需要虚拟 DOM？

虚拟 DOM 的出现并不是为了"比原生 DOM 更快"，而是为了在性能、可维护性和跨平台之间取得平衡。可以从以下几个角度理解：

1. **解决频繁、零散操作真实 DOM 的性能问题**
   真实 DOM 操作非常昂贵：每次修改都可能触发样式计算、回流（Reflow）和重绘（Repaint）。如果开发者手动操作 DOM，很容易出现"改一次状态就操作一次 DOM"的反复抖动。虚拟 DOM 把多次状态变化先在内存里合并、对比（diff），最后一次性以最小变更集（patch）同步到真实 DOM，避免不必要的回流重绘。

2. **抹平命令式 DOM 操作的复杂度，转向声明式 UI**
   没有虚拟 DOM 之前，开发者要自己关心"现在 DOM 是什么样、应该怎么改"。有了虚拟 DOM，只需要声明"UI 在某个状态下应该长什么样"，由框架根据 `state -> view` 的映射自动算出 DOM 变更。心智负担从"怎么改 DOM"变成"数据应该是什么"，可维护性大幅提升。

3. **提供跨平台渲染的中间层**
   虚拟 DOM 本质是一棵描述 UI 的 JavaScript 对象树，它和浏览器 DOM 解耦。同一份 React 组件可以通过不同的渲染器输出到浏览器（react-dom）、原生移动端（React Native）、Canvas、SSR 字符串等环境。这是手写 DOM 操作无法做到的。

4. **配合 diff 算法做最小化更新**
   React 通过 key、同层比较等策略，把 O(n³) 的树 diff 降到 O(n)，再结合 Fiber 的可中断调度，让大规模更新也能保持流畅。这种优化建立在"有一份可对比的上一次结果"基础上，而虚拟 DOM 正是这份"上一次结果"。

5. **隔离副作用，便于测试和调试**
   虚拟 DOM 是纯 JavaScript 对象，不依赖浏览器环境，因此可以在 Node 环境下做单元测试、快照测试、SSR，也方便 React DevTools 这类工具去观察组件树。

需要强调的一点：**虚拟 DOM 不是绝对更快**。在简单场景下（比如只改一个文本节点），手写 `el.textContent = ...` 一定比"创建 vdom -> diff -> patch"更快。虚拟 DOM 真正的价值在于：

- 在**复杂、频繁更新**的应用里，把零散操作合并为最小批量更新，平均性能更稳定；
- 提供了**声明式编程模型**，让大型项目的复杂度可控；
- 抽象出了一层**与平台无关的 UI 描述**，带来了跨端能力。

一句话总结：虚拟 DOM 的核心价值不是"更快"，而是"更可控、可维护、可跨平台"，它用一点额外的内存和计算开销，换来了开发体验和工程化能力上的巨大收益。

### 说说 React JSX 转换成真实 DOM 的过程。

JSX 本质上不是浏览器原生语法，而是语法糖。

大致过程：

1. JSX 经过 Babel 等工具编译成 `React.createElement` 调用或新的 JSX runtime 调用。
2. 这些调用生成 React Element 对象。
3. React 根据这些对象构建 Fiber 树。
4. 在协调阶段比较新旧节点差异。
5. 在提交阶段把变化同步到真实 DOM。

所以 JSX 到真实 DOM 不是一步完成，而是“编译成对象描述，再由 React 调度渲染”。

## 二、组件、状态与通信

### 说说 React 生命周期有哪些不同阶段？每个阶段对应的方法是什么？

如果从类组件角度回答，生命周期大致分为三个阶段：

1. 挂载阶段：`constructor`、`getDerivedStateFromProps`、`render`、`componentDidMount`
2. 更新阶段：`getDerivedStateFromProps`、`shouldComponentUpdate`、`render`、`getSnapshotBeforeUpdate`、`componentDidUpdate`
3. 卸载阶段：`componentWillUnmount`
4. 错误处理：`getDerivedStateFromError`、`componentDidCatch`

历史上还有一些旧生命周期，如 `componentWillMount`、`componentWillReceiveProps`、`componentWillUpdate`，从 React 16.3 起被标记为不安全（加 `UNSAFE_` 前缀），原因是它们在 Fiber 异步可中断渲染下可能被多次调用，导致副作用重复执行。具体演进可以参考"七、React 各版本演进与新特性"中的生命周期演进图。

如果面试官继续追问，可以补充：函数组件主要通过 Hooks 来组织这些生命周期逻辑，比如 `useEffect`、`useLayoutEffect`。

### state 和 props 有什么区别？

两者都是 React 组件中的核心概念，但职责不同。

1. `props` 是外部传入的，组件内部一般不直接修改。
2. `state` 是组件内部维护的，用来描述组件自身状态。
3. `props` 更像输入，`state` 更像组件内部数据源。

一句话总结：`props` 是父组件给的，`state` 是组件自己管的。

### super() 和 super(props) 有什么区别？

这是类组件中的问题。

1. `super()` 用于调用父类构造函数。
2. `super(props)` 则会把 `props` 传给父类构造函数。

在 React 类组件中，如果你在 `constructor` 里需要通过 `this.props` 访问传入属性，通常要写 `super(props)`。

在现代 React 中，这类问题出现得少了，因为函数组件已经成为主流。

### 说说 React 中的 setState 执行机制。

`setState` 的核心作用是更新状态并触发重新渲染，但它不是简单的“立刻同步赋值”。

关键点：

1. React 会对多次状态更新做合并和调度。
2. `setState` 可能是异步表现，具体取决于执行上下文和 React 版本。
3. 状态更新后会进入协调流程，再决定是否重新渲染。

如果新状态依赖旧状态，推荐使用函数式写法：

```js
setState(prev => ({ count: prev.count + 1 }))
```

这里的 `prev` 是 React 在调度队列里取出的"上一个最新值"，而不是闭包里被捕获的旧 state。这样即使在同一 tick 里连续调用三次 `setState(prev => ...)`，也能正确累加；如果写成 `setState({ count: count + 1 })`，三次调用拿到的 `count` 都是同一帧的旧值，最终只 +1。

### React 事件机制是什么？

React 没有完全直接使用原生 DOM 事件系统，而是在外层封装了一套合成事件机制。

它的特点：

1. 统一不同浏览器的事件行为。
2. 提供一致的事件对象接口。
3. 通过事件委托减少大量事件绑定开销。

早期 React 主要把事件委托到 `document`，后续版本对事件委托节点做过调整。

### React 事件绑定的方式有哪些？区别是什么？

常见方式有：

1. JSX 中直接传箭头函数。
2. JSX 中传已经绑定好的方法。
3. 在构造函数中手动 `bind`。
4. 类字段语法定义箭头函数。

区别主要在：

1. `this` 指向处理方式不同。
2. 是否会在每次渲染时创建新函数。
3. 可读性和性能取舍不同。

现代项目里更常见的是函数组件 + 内联回调或 `useCallback` 配合优化。

**为什么这条建议成立？** 各种绑定方式的取舍可以直接展开看：

1. **类组件构造函数 `bind`**：`this.handle = this.handle.bind(this)` 写在 constructor 里。优点是只 bind 一次，每次 render 复用同一引用，传给子组件不会因为新引用导致子组件 memo 失效；缺点是模板代码繁琐，每个方法都要列一次。
2. **类字段箭头函数**：`handle = () => {...}`，等价于"自动 bind"。简洁但每个实例会创建一份函数副本，多实例时内存占用略高。
3. **JSX 内联箭头函数**：`<button onClick={() => this.handle()}>`。每次 render 都创建新函数，传给被 memo 的子组件会让 memo 失效。简单场景没影响，但如果按钮在很重的列表里，这会成为性能问题。
4. **JSX 直接传方法 `onClick={this.handle}`**：必须配合上面三种方式之一才能保证 `this` 正确。

到了函数组件 + Hooks，没有 `this` 问题，但**重复创建函数**的问题依然存在：

```jsx
function Parent() {
  // ❌ 每次 render 都是新函数
  const handle = () => { ... }
  return <Child onClick={handle} />  // Child 即使被 memo 也会重渲

  // ✅ useCallback 缓存引用
  const handle = useCallback(() => { ... }, [deps])
  return <Child onClick={handle} />  // 只要 deps 没变，引用稳定
}
```

但**不是所有回调都该 `useCallback`**：

1. 没有传给被 `React.memo` 包裹的子组件 / 没作为 useEffect 依赖时，`useCallback` 反而是负优化——它自己有比较 deps 数组的成本。
2. 如果父组件本身很少 render，子组件即使因为新函数引用重渲，成本也可忽略。

简单决策：**先按"内联回调最简洁"写**，等 profiler 显示某个子组件被频繁不必要重渲且 props 中含函数时，再改成 `useCallback`。React 19 后的编译器（React Compiler）会自动做这种 memoization，未来这道题的答案会进一步简化。

### React 构建组件的方式有哪些？区别是什么？

常见方式主要有两种：

1. 类组件
2. 函数组件

区别：

1. 类组件有 `this`、生命周期方法和 `setState`。
2. 函数组件更轻量，通过 Hooks 管理状态和副作用。
3. 现在业务开发中函数组件是主流。

### React 中组件之间如何通信？

React 中常见通信方式有：

1. 父传子：`props`
2. 子传父：通过回调函数通知父组件
3. 跨层级传递：`Context`
4. 跨组件共享状态：Redux、Zustand、MobX 等状态管理工具
5. 通过 `ref` 获取子组件能力

面试时最好补一句：简单场景优先 `props` 和回调，跨层级或全局共享再考虑 `Context` 和状态管理。

**为什么有这个优先级？**

1. **`props` 透传越深越糟（"prop drilling"）**：父 → 子 → 孙 三层就开始繁琐，五层以上几乎不可维护——中间层组件被迫接收并不关心的 props，重构时改一个数据格式要动好几个文件。
2. **`Context` 解决了透传，但有重渲代价**：任何 Context value 的引用变化，都会让所有消费该 Context 的组件重新渲染——即使它们只用了 value 中的一小部分。这是 Context 不适合作为"高频写入的全局状态"的根本原因。
3. **状态管理库（Redux / Zustand / Jotai）是为高频更新设计的**：它们都内置了"selector + 浅比较"机制，组件只在自己关心的切片变化时重渲，性能上远胜 Context。

**所以选型分三档：**

| 场景 | 推荐 | 理由 |
| --- | --- | --- |
| 父子直接通信 | props + 回调 | 最简单，类型最清晰，不引入额外抽象 |
| 跨 2~3 层、低频变化（主题、用户信息、i18n） | Context | 简洁、无依赖、但要避免高频写入 |
| 跨多层、高频更新、多处订阅 | Zustand / Redux / Jotai | selector 优化、DevTools、可序列化、可持久化 |
| 服务端数据缓存 | React Query / SWR | 自带请求去重、缓存失效、乐观更新，远胜手撸 |

一个常见误用：把整个用户数据放到 Context 里然后每次 setState 都更新整个 user 对象——所有 inject 该 Context 的组件都会重渲。正确做法是要么拆 Context（分别 ProvideUserId 和 ProvideUserDetail），要么直接用 Zustand。

### React 中的 key 有什么作用？

`key` 是列表渲染中用于标识节点身份的唯一标记。

它的作用：

1. 帮助 React 在 diff 时识别节点是否可复用。
2. 提高列表更新效率。
3. 避免错误复用组件状态。

实际开发中应优先使用稳定唯一的 id，不建议在可变列表中直接使用数组下标。

**为什么数组下标会出问题？**

下标本质上是"位置编号"，不是"内容身份"——数组顺序一变，同一个下标就指向了不同数据。React 的 diff 根据相同 key 判断"这个节点可以复用"，所以位置变化的下标会让 React 把"换了内容的节点"当作"没变的节点"复用，结果是 DOM 被复用但内容、组件状态、用户输入全部对应错位置。

举个最常见的例子——列表头部插入新元素：

```jsx
{list.map((item, i) => (
  <li key={i}>
    <input />        {/* 用户在第一项 input 里输入了 "hello" */}
    <span>{item.name}</span>
  </li>
))}
```

初始数据 `[A, B, C]`，head 插入 X 后变成 `[X, A, B, C]`：

| 下标 | 旧 | 新 | diff 判定 | 表现 |
| --- | --- | --- | --- | --- |
| 0 | A | X | key 相同 → 复用 | A 的 DOM 被复用为 X，**"hello" 留在了 X 的输入框里** |
| 1 | B | A | 复用 | B 的 DOM 改渲染 A |
| 2 | C | B | 复用 | C 的 DOM 改渲染 B |
| 3 | - | C | 新建 | 全新创建 |

视觉上"插入了一项"，实际上 React 改写了几乎所有 DOM 的内容。更严重的是组件内部 state（受控 input、`useState`、动画进度等）全部错位。

如果用 `key={item.id}`：

| key | 旧位置 | 新位置 | diff 判定 | 表现 |
| --- | --- | --- | --- | --- |
| id-A | 0 | 1 | 找到 → 移动 | A 的 DOM 整体移到位置 1，"hello" 随 A 走 ✓ |
| id-X | - | 0 | 不存在 → 创建 | 全新 X 节点，input 是空的 ✓ |

**性能层面也一样：**

1. 下标作 key：表面 DOM 没新增，但 React 触发了几乎所有 li 的属性 patch，反而更慢。
2. 稳定 id 作 key：React 用"位置移动"完成更新，不需要修改节点内部属性，DOM 操作更少。

**什么时候用下标是安全的？**

满足以下三个条件之一才可放心用下标：

1. 列表纯渲染、永不增删、永不排序、永不过滤；
2. 列表项是无状态纯展示，不含 input、组件、动画状态；
3. 列表项内部完全没有受 React 管理的 state。

一旦项里出现 `<input>`、`useState`、动画 transition、子组件，下标就会埋雷。**默认就用稳定 id**，把"用下标"当成需要论证才能用的特例。

### 说说对 React refs 的理解？应用场景有哪些？

`ref` 用于直接访问 DOM 节点或子组件实例能力。

常见场景：

1. 输入框聚焦
2. 获取元素尺寸和位置
3. 调用子组件暴露的方法
4. 和第三方 DOM 库集成

在 React 中，`ref` 更像一种”逃生舱”，只有在声明式方案不合适时才使用。

**为什么把 ref 称为”逃生舱”？**

1. **它绕过了 React 的核心抽象**：React 的设计假设是”UI = f(state)”——状态决定视图，开发者不该直接操作 DOM。`ref` 让你直接拿到节点或子组件实例，相当于跳出这个数据流模型。
2. **直接 DOM 操作和 React 的 reconciliation 容易冲突**：你手动 `el.style.color = 'red'` 后，React 下次 render 时不会把这次修改保留下来，可能直接被覆盖；或者你 `el.remove()` 一个节点，React 还以为它在那里，下次更新就崩。
3. **可测试性、可推理性下降**：声明式代码”看 state 就知道结果”；命令式 ref 操作要追踪一串 effect 才能理解 UI 现在是什么样。

**什么时候必须用 ref？** 是 React 没法用声明式表达的能力：

1. **聚焦/失焦/选区**：`input.focus()`、`input.select()` 没有声明式等价物。
2. **测量尺寸/位置**：`getBoundingClientRect`、`scrollTop` 是命令式 API。
3. **媒体控制**：`video.play()` / `pause()` / `currentTime`。
4. **集成第三方非 React 库**：比如 D3、Chart.js、地图 SDK——它们直接操作 DOM，需要把容器节点交给它们。
5. **动画**：用 GSAP、Web Animations API 时，性能上比 setState 驱动更平滑。
6. **跨渲染保留可变值**：`useRef(0)` 当成实例变量用，不触发重渲（这种场景已经不是 DOM 的 ref，而是 ref 的另一面）。

**用 ref 时的纪律：**

1. 优先尝试声明式方案——能用 CSS 切换的不用 ref，能用 state 控制的不动 DOM。
2. 用 ref 操作 DOM 时只读不写，能不修改 DOM 就不修改。
3. 子组件的 ref 暴露要克制，用 `useImperativeHandle` 显式声明白名单方法，而不是把整个实例暴露出去——后者一旦被外部依赖，重构会很痛。

简单判断：**问自己”能不能用 props/state 表达？”——能就别用 ref。**

### 说说对 React 中类组件和函数组件的理解？有什么区别？

类组件和函数组件都能描述 UI，但设计方式不同。

区别主要有：

1. 类组件依赖 `this` 和生命周期。
2. 函数组件依赖 Hooks 组织状态和副作用。
3. 函数组件通常更简洁，更适合逻辑复用。
4. 现代 React 更推荐函数组件。

### React 中 `PureComponent`、`React.memo`、`Component` 有什么区别？

它们都和渲染优化有关，但作用对象和方式不同。

1. `Component` 是普通类组件基类，不会默认帮你做浅比较。
2. `PureComponent` 会对 `props` 和 `state` 做浅比较，只有变化时才继续更新。
3. `React.memo` 主要用于函数组件，作用类似函数组件版本的浅比较优化。

面试时要强调：

1. 这些优化依赖浅比较。
2. 如果数据更新不是不可变更新，优化可能失效。
3. 不是所有组件都值得做 memo，优化前应先确认瓶颈。

**1. 浅比较是什么、靠不靠谱？**

`PureComponent` 和 `React.memo` 内部都用 `Object.is` 对 props/state 的**第一层**字段做比较：

1. 基本类型（string、number、boolean）直接比值。
2. 对象、数组、函数只比引用（`===`），**不会递归看内容**。

这就带来两类典型踩坑：

1. **引用相同但内容已变**：直接 `state.list.push(item)` 后 `setList(state.list)`，引用没变 → 浅比较判定相等 → 组件不更新，UI 与数据脱节。
2. **引用变了但内容相同**：父组件每次 render 都 `<Child config={{ a: 1 }} />`，`config` 每次都是新对象 → 浅比较判不等 → `memo` 直接失效。

**绕过办法：**

1. 函数 prop 用 `useCallback`、对象/数组 prop 用 `useMemo` 稳定引用。
2. 自定义 `React.memo(Component, areEqual)` 的第二参数做更精细比较，但深比较代价可能超过重渲本身，慎用。

**2. 为什么必须不可变更新？**

浅比较的整套机制都建立在"**引用变 = 内容变**"这个假设上。一旦你做了原地修改，假设就破了：

```js
// ❌ 反面教材：引用没变，memo 误判没更新
setUser(prev => {
  prev.name = 'Tom'
  return prev
})

// ✅ 正确：返回新引用
setUser(prev => ({ ...prev, name: 'Tom' }))
```

数组同理，要用 `[...list, item]`、`list.filter(...)`、`list.map(...)` 这类**返回新数组**的 API，而不是 `push`、`splice`、`sort` 这类原地修改。

工程实践上：

1. 简单结构手写展开（`{...obj}` / `[...arr]`）即可。
2. 嵌套深时用 **Immer 的 `produce`**：写起来像可变，背后产新对象，可读性和正确性兼得。
3. Redux Toolkit、Zustand 的 immer 中间件已经内置了 Immer，直接 `state.x.y = z` 也是不可变更新。

**3. memo 不是越多越好**

`React.memo` 自己也有成本：每次 render 都要遍历 props 做浅比较、还要缓存上一次结果。**对一个本身渲染就很轻的组件包 memo，比较的开销可能比直接重渲还高**，属于负优化。

判断要不要 memo 的实操路径：

1. **先用 React DevTools Profiler 录一段交互**，看哪些组件渲染时间长、渲染次数多。
2. 只对"**渲染开销大** + **父组件高频更新** + **props 大概率没变**"这类组件加 memo。常见对象：长列表 item、图表、复杂表单字段。
3. 加完后再 Profile 一次，确认确实减少了渲染时间，否则回滚——避免引入"看起来在优化"的代码。

最后补一句**前瞻**：React 19 的 **React Compiler** 会在编译期自动插入 memo / useMemo / useCallback，未来手动 memo 的场景会越来越少，但**理解上面这套原理依然是判断编译器优化是否生效、是否需要兜底的基础**。

### 说说对受控组件和非受控组件的理解？应用场景是什么？

受控组件指表单值由 React 状态控制，输入变化通过事件同步到状态。

非受控组件指表单值更多由 DOM 自己维护，通常通过 `ref` 获取值。

适用场景：

1. 受控组件适合需要实时校验、联动、统一状态管理的表单。
2. 非受控组件适合简单表单或与原生表单行为深度结合的场景。

React 项目里，大多数复杂表单更倾向使用受控组件。

**为什么复杂表单选受控？** 受控带来的能力是非受控做不到的：

1. **实时校验和联动**：手机号每输一个字符就校验长度、密码强度实时显示、省份联动城市——这些都要求"当前值随时可读"，受控组件天然可以 `useState` 拿到，非受控要监听 input 事件再 setState，反而更绕。
2. **格式化和限制**：自动给金额加千分位、自动转大写、限制只能输数字——这些是"输入时改变值"，必须由 state 控制；非受控的 `defaultValue` 不能干预后续输入。
3. **统一数据源便于提交**：复杂表单要序列化成 API 请求体，受控组件让所有字段都在 state 树里，提交时 `JSON.stringify(formData)` 即可；非受控要逐个 `ref.current.value` 收集，丢字段、忘 trim 是常见 bug。
4. **回显已有数据**：编辑场景需要把后端数据填回表单。受控组件 `setState(serverData)` 一行搞定；非受控要在 mount 后 `inputRef.current.value = ...`，而且不能拦截 submit 验证。
5. **跨字段错误提示**：密码和确认密码联动校验、订单总价随商品数量变化——这些跨字段计算依赖"所有字段都在同一份 state 里"，受控天然支持。

**那非受控什么时候用？** 三个典型场景：

1. **简单一次性表单**：联系我们、留言板，提交即销毁，不需要中间态。
2. **大型表单的性能优化**：受控表单每输一个字符触发整个表单组件重渲，1000 个字段时会卡。这时用 `react-hook-form` 这种"非受控但帮你收集"的库，性能比纯受控好一个数量级。
3. **必须用原生表单行为时**：`<input type="file">`（受控不被允许）、`<input type="password">` 在某些密码管理器场景。

**实际推荐：**

1. 字段少（10 个以内）、要校验联动 → 直接 `useState` 受控。
2. 字段多、性能敏感、想要好 DX → 用 `react-hook-form` + zod，是非受控但 DX 接近受控。
3. 一次性提交、无校验 → 非受控 + ref 取值即可。

### 说说对高阶组件的理解？应用场景是什么？

高阶组件 HOC 本质上是“接收组件，返回增强后组件”的函数。

常见场景：

1. 权限控制
2. 日志埋点
3. 通用状态注入
4. 逻辑复用

不过在现代 React 中，很多原先用 HOC 解决的问题，也会转向 Hooks 方案，因为逻辑来源更直观。

**HOC 和自定义 Hook 的本质差异：**

1. **嵌套地狱（Wrapper Hell）**：HOC 是层层包装组件，DevTools 里会看到 `withAuth(withTheme(withRouter(MyComp)))`，调试时 props 来源很难追溯。Hook 是平铺调用，不增加组件树层级。
2. **Props 命名冲突**：两个 HOC 都给组件注入 `data` prop 时，后一个会覆盖前一个，且没有任何编译警告。Hook 的返回值由调用方主动解构、命名，冲突不会发生。
3. **Props 来源不透明**：组件里看到 `this.props.user`，要回头去找哪个 HOC 注入的；Hook 调用 `const user = useUser()`，import 语句一目了然。
4. **静态方法/refs 转发繁琐**：HOC 默认会"吃掉"被包装组件的静态方法和 refs，要手动 hoist + forwardRef。Hook 不存在这个问题。
5. **TypeScript 友好度**：HOC 的类型签名（要保留泛型 + 注入新 props + 移除已注入 props）非常复杂，社区里的 HOC TS 类型几乎都有边界 case 出错；Hook 类型是普通函数，TS 完美推导。

**HOC 还有用武之地吗？** 少数场景仍然合适：

1. **需要在组件树中插入额外结构**（如 `<Provider>` 包裹、`<ErrorBoundary>` 包裹）——Hook 做不到改组件树，HOC 自然胜任。
2. **跨组件层级劫持渲染**（如 redux 的 `connect`、mobx 的 `observer`）——这些是 HOC 的天然场景。
3. **不能用 Hook 的旧类组件代码**——遗留代码迁移过渡期。

**所以现代写法：**

```js
// ❌ 旧做法
const EnhancedComp = withAuth(withTheme(MyComp))

// ✅ 现代做法
function MyComp() {
  const auth = useAuth()
  const theme = useTheme()
  // ...
}
```

例外是已有完整 HOC 体系的库（react-redux 的 `connect`），新代码用 hook（`useSelector`/`useDispatch`）即可，不必重写库。

### 说说对 React Hooks 的理解？解决了什么问题？

Hooks 是 React 16.8 为函数组件提供状态和副作用能力的一组 API。

它解决的问题主要有：

1. 函数组件以前不能方便管理状态，现在可以了。
2. 逻辑复用过去依赖 HOC 和 render props，现在可以抽成自定义 Hook。
3. 类组件中相关逻辑分散在生命周期里，Hooks 可以按业务逻辑聚合。

常见 Hooks 包括：`useState`、`useEffect`、`useMemo`、`useCallback`、`useRef`、`useContext`、`useReducer`、`useImperativeHandle`、`useLayoutEffect`、`useDebugValue`。

React 18 又新增了 `useId`、`useDeferredValue`、`useTransition`、`useSyncExternalStore`、`useInsertionEffect`，主要服务并发渲染和外部 store。

React 19 进一步新增 `useActionState`、`useFormStatus`、`useOptimistic` 以及 `use`，主要服务 Actions 和异步数据流。详见"七、React 各版本演进与新特性"。

使用 Hooks 时要记住几条规则：

1. 只能在函数组件或自定义 Hook 顶层调用，不能放在条件分支或循环里。
2. 自定义 Hook 必须以 `use` 开头。
3. 依赖数组要写完整，否则容易出现闭包陷阱。

**这三条规则的底层原因：**

1. **为什么不能在条件分支里调用？** Hooks 的实现依赖"调用顺序"——React 内部用一个数组按顺序存每次 render 的 hooks 状态。第一次 render 调用了 `useState/useEffect/useState`，下次 render 就期望同样的顺序。如果第二次 render 因为条件没调用第一个 useState，后面所有 hook 的 state 都会错位（"useEffect 拿到了第二个 useState 的状态"）。React 会直接报错。
2. **为什么必须 `use` 开头？** 这是给 ESLint 的 `react-hooks/rules-of-hooks` 规则用的——只有名字以 `use` 开头的函数才会被当作 Hook 校验"是否在条件分支里调用"。命名是约定，但ESLint 强制依赖它。
3. **为什么依赖数组必须完整？这就是经典的"闭包陷阱"：**

```jsx
function Counter() {
  const [count, setCount] = useState(0)

  // ❌ deps 写空数组，effect 里捕获的是首次 render 的 count（永远是 0）
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1)  // 每秒把 0 + 1，而不是累加
    }, 1000)
    return () => clearInterval(id)
  }, [])
}
```

每次 render 都会创建新的 `count` 闭包，但 effect 只在 mount 时跑一次，捕获的是初次 render 的 count（值 0）。所以 setInterval 里读到的永远是 0，`setCount(0 + 1)` 一直把 count 设成 1。

**修复方法：**

1. **正确写依赖**：`useEffect(..., [count])` 让 effect 在 count 变化时重启。但 setInterval 这样会导致每秒重新创建定时器，浪费。
2. **用函数式更新**：`setCount(c => c + 1)`，不依赖外部 count，deps 可以保持空数组。
3. **用 ref 保存最新值**：`useRef(count)` + 在 effect 里读 `ref.current`。

**`exhaustive-deps` ESLint 规则**会自动检测漏掉的依赖，**强烈建议开启**——它会警告"你这个 effect 用了 count 但没写到 deps 里"。React 19 的编译器进一步会自动 memoize，未来这条陷阱也会减少。

记住一句话：**"读了什么响应式值，就要把它写进 deps"——除非你明确用函数式更新或 ref 跳出闭包。**

### 详解 React Hooks 的闭包陷阱（Stale Closure）

闭包陷阱（Stale Closure，也叫"过期闭包"）几乎是函数组件最高频的 bug 来源。要彻底搞懂它，需要从**函数组件的执行模型**说起。

#### 一、为什么函数组件会有闭包陷阱？

函数组件**每次 render 都是一次完整的函数调用**：

```jsx
function Counter() {
  const [count, setCount] = useState(0)
  // 每次 render，整个函数体都会重新执行一遍
  // count、handleClick、effect 回调都是"这次 render"的全新变量
  const handleClick = () => console.log(count)
  return <button onClick={handleClick}>{count}</button>
}
```

关键认知：

1. **每次 render 都会创建一组全新的局部变量、函数、effect 回调**。
2. 这些函数通过**词法作用域**捕获了"那一次 render 时的 state 和 props"。
3. 一旦这个函数被"保存"到某个长寿命的位置（`setInterval`、订阅回调、`ref`、外部缓存…），它捕获的就是**那一帧的快照**，再也不会更新。

所以闭包陷阱不是 React 的 bug，而是 JS 闭包语义 + "每次 render 是新调用"叠加的必然结果。

#### 二、四种典型场景

**场景 1：useEffect 空依赖 + 定时器**

```jsx
function Counter() {
  const [count, setCount] = useState(0)
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1)  // ❌ count 永远是 0
    }, 1000)
    return () => clearInterval(id)
  }, [])  // ❌ 漏写 count
}
```

表现：UI 显示 1 之后就不动了。原因：定时器回调捕获了首次 render 的 `count = 0`。

**场景 2：事件订阅 / WebSocket 回调**

```jsx
function Chat({ roomId }) {
  const [messages, setMessages] = useState([])
  useEffect(() => {
    socket.on('message', (msg) => {
      setMessages([...messages, msg])  // ❌ messages 永远是 []
    })
  }, [roomId])
}
```

新消息到达时，回调里读到的 `messages` 永远是订阅那一刻的快照（空数组），导致**新消息会覆盖旧消息**，列表里只显示最新一条。

**场景 3：异步请求里读 state**

```jsx
function Search({ keyword }) {
  useEffect(() => {
    fetch(`/api?q=${keyword}`).then(res => {
      // 慢请求返回时，keyword 可能已经变了
      // 但这里读到的是 effect 启动时的 keyword
      console.log(keyword)
    })
  }, [keyword])
}
```

这本身不是 bug——但如果用户快速切换关键词，旧请求返回时会**覆盖新请求的结果**，这就是经典的"竞态（race condition）"问题。

**场景 4：自定义 Hook 里返回的函数**

```jsx
function useDebounce(fn, delay) {
  return useCallback(() => {
    setTimeout(fn, delay)  // ❌ 每次 render fn 都不同，但 useCallback 捕获了首次的 fn
  }, [])
}
```

调用方传进来的 `fn` 每次 render 都是新的，但 `useCallback` 空依赖锁住了首次的 `fn`，导致防抖触发时执行的是**过期逻辑**。

#### 三、五种修复方案

**方案 1：补全依赖数组**

```jsx
useEffect(() => {
  const id = setInterval(() => setCount(count + 1), 1000)
  return () => clearInterval(id)
}, [count])  // ✅
```

代价：每次 count 变化都会**销毁并重建定时器**。对 setInterval 这种长寿命副作用是浪费，对一次性 effect 没问题。

**方案 2：函数式更新（最推荐）**

```jsx
useEffect(() => {
  const id = setInterval(() => setCount(c => c + 1), 1000)  // ✅ 不读外部 count
  return () => clearInterval(id)
}, [])
```

`setCount(c => c + 1)` 里的 `c` 来自 React 的最新状态队列，和闭包无关。**只要更新只依赖旧值，永远优先用函数式更新。**

**方案 3：ref 保存最新值**

```jsx
function Chat({ roomId }) {
  const [messages, setMessages] = useState([])
  const messagesRef = useRef(messages)
  useEffect(() => { messagesRef.current = messages })  // 每次 render 同步

  useEffect(() => {
    socket.on('message', (msg) => {
      setMessages([...messagesRef.current, msg])  // ✅ 永远读最新值
    })
  }, [roomId])
}
```

适合"订阅回调里需要读多个最新值，但又不想让 effect 反复重启"的场景。**ref 是闭包陷阱的"逃生舱"**。

**方案 4：useEvent / useEventCallback 模式**

把"事件式逻辑"从 effect 里抽出来，让它始终拿到最新闭包：

```jsx
function useEventCallback(fn) {
  const ref = useRef(fn)
  useLayoutEffect(() => { ref.current = fn })
  return useCallback((...args) => ref.current(...args), [])
}

function Chat({ roomId }) {
  const [messages, setMessages] = useState([])
  const onMessage = useEventCallback((msg) => {
    setMessages([...messages, msg])  // ✅ 永远是最新 messages
  })
  useEffect(() => {
    socket.on('message', onMessage)
    return () => socket.off('message', onMessage)
  }, [roomId])
}
```

React 18 的 RFC 中曾计划提供官方的 `useEvent`，最终演变为 React 19 的 `useEffectEvent`（实验性）。

**方案 5：cleanup + 取消标志（专治竞态）**

```jsx
useEffect(() => {
  let cancelled = false
  fetch(`/api?q=${keyword}`).then(data => {
    if (!cancelled) setResult(data)  // ✅ 旧请求返回直接丢弃
  })
  return () => { cancelled = true }
}, [keyword])
```

或者用 `AbortController` 直接中断请求。

#### 四、如何主动避免？

1. **开启 `eslint-plugin-react-hooks` 的 `exhaustive-deps` 规则**——它会自动警告漏写的依赖，是闭包陷阱的第一道防线。**不要随便用 eslint-disable 关掉它**，每次想关都先问"我能不能用函数式更新或 ref 解决"。
2. **优先用函数式更新**——所有"新值只依赖旧值"的场景都不会触发陷阱。
3. **拆分 effect**——一个 effect 只做一件事，依赖数组就短，闭包就简单。
4. **长寿命订阅用 ref 兜底**——socket、定时器、全局事件监听等场景，用 ref 保存最新回调。
5. **升级到 React 19**——React Compiler 会自动 memoize，并且 `useEffectEvent` 提供了官方的"始终拿最新值"的能力，闭包陷阱出现频率会显著下降。

#### 五、面试一句话总结

> 闭包陷阱的本质是"**函数组件每次 render 都是一次新调用，effect/回调里捕获的是那一帧的 state 快照**"。修复思路只有两条：要么**让 effect 在依赖变化时重新创建闭包**（写全 deps），要么**让回调在执行时主动读取最新值**（函数式更新、ref、useEffectEvent）。

## 三、样式、动画与生态能力

### 说说 React 中引入 CSS 的方式有哪几种？区别是什么？

常见方式有：

1. 普通 CSS 文件
2. CSS Modules
3. Sass/Less 等预处理器
4. CSS-in-JS
5. 原子化 CSS 方案

区别主要在：

1. 样式作用域控制能力不同。
2. 工程组织方式不同。
3. 运行时开销和开发体验不同。

实际项目中常见组合是：普通全局样式 + CSS Modules 或原子化方案。

**为什么这样组合？** 各种方案各有不可替代的场景：

1. **必须有少量全局样式**：reset/normalize.css、全局字体、CSS 变量定义、深色模式根选择器——这些是"跨所有组件生效"的，写到 CSS Modules 里反而每个组件都要 import 一次。
2. **组件级隔离选 CSS Modules 或原子化**：业务组件的样式必须避免污染全局——CSS Modules 通过类名加 hash、原子化（Tailwind 等）通过预生成单一职责类，都能做到这点。
3. **运行时 CSS-in-JS 渐渐被冷落**：styled-components、emotion 这类运行时方案会带来包体积（约 12KB）和首次渲染开销（要解析 props 生成样式表），React Server Component 时代它们也水土不服——所以很多团队从 styled-components 迁回 CSS Modules + Tailwind。
4. **零运行时 CSS-in-JS 是新趋势**：Vanilla Extract、Linaria、Panda CSS、CSS Modules + CSS 变量 都是"在构建时生成 CSS"的方案，既保留了 JS 能力（主题、变量复用）又没有运行时开销。

**典型组合的具体分工：**

| 用途 | 方案 |
| --- | --- |
| 重置默认样式、字体、CSS 变量 | 普通全局 CSS（`index.css` 一份） |
| 业务组件、卡片、表单 | CSS Modules（`Foo.module.css` + `import`） |
| 间距、布局、颜色等高频小样式 | 原子化（Tailwind 等） |
| 主题切换、复杂动画 | CSS 变量 + media query / 零运行时 CSS-in-JS |
| 高度动态、与组件 prop 强绑定 | CSS-in-JS（仅在必要场景） |

**选型的本质**是在三个维度上找平衡：作用域隔离（避免污染）、动态性（能否根据 props 生成样式）、运行时成本（包体积 + 首屏）。三者无法同时拉满，根据项目对哪个最敏感来选。

### 什么是 CSS Modules？它在 React 项目中有什么价值？

CSS Modules 是一种让样式默认局部作用域化的方案，常见表现是编译后把类名转换成带哈希的唯一标识。

它的价值主要有：

1. 避免全局样式污染。
2. 减少命名冲突。
3. 更适合组件级样式组织。

它和普通全局 CSS 的核心区别是：普通 CSS 默认全局生效，而 CSS Modules 更强调组件级隔离。

### CSS-in-JS 在 React 中是怎么理解的？适合什么场景？

CSS-in-JS 指的是把样式逻辑写进 JavaScript 中，再通过运行时或编译时方案生成样式。

常见价值：

1. 更方便做动态样式。
2. 样式与组件逻辑更靠近。
3. 可以天然做局部作用域隔离。

适合场景：

1. 主题系统。
2. 动态样式较多的组件。
3. 组件库封装。

但也要说明：CSS-in-JS 会带来一定运行时或构建复杂度，不一定适合所有项目。

**具体的代价是什么？**

1. **运行时方案的开销**（styled-components、emotion）：每次组件 render，库都要根据 props 计算 className、把 CSS 字符串注入 `<style>` 标签、维护样式去重缓存。包体积约 12~15KB（min+gzip），SSR 时要做样式抽取，首屏 hydration 也要重建样式表。
2. **TypeScript 友好度参差**：styled-components 的 `styled.div<{primary: boolean}>` 写起来繁琐，泛型类型报错信息晦涩；emotion 的 `css` prop 在 JSX 类型推导上要手动配置 pragma。
3. **DevTools 调试不直观**：浏览器看到的是 `sc-abc123`、`css-1xy2z3` 这种 hash 类名，定位回源文件要靠 babel 插件加 displayName，且 source map 不总是准。
4. **和 React Server Component 不兼容**：styled-components 在 RSC 下完全无法工作（依赖 Context 和运行时挂载），是 Next.js App Router 时代它被逐渐放弃的关键原因。
5. **零运行时方案的代价是约束**：Vanilla Extract、Linaria 要求样式必须在构建时可静态分析，这意味着不能写"任意运行时 props 决定样式"的代码——动态颜色只能通过预定义的几个 variant 或 CSS 变量传入。

**所以选型时该问的问题：**

1. 项目用 Next.js App Router / RSC？→ 避开运行时 CSS-in-JS。
2. 团队对包体积敏感（C 端、移动端）？→ Tailwind 或 CSS Modules，不要运行时方案。
3. 主题/品牌色多变体、需要 props 动态计算样式？→ 零运行时 CSS-in-JS 是较好的折中。
4. 中后台、约束少、追求 DX？→ Tailwind 已经成为事实标准。
5. 老项目已用 styled-components？→ 没必要硬迁，等真有性能问题再考虑。

简言之：CSS-in-JS 不是"绝对不能用"，而是"它的代价在 RSC 时代变大了"，需要更明确的理由。

### 在 React 中组件间过渡动画如何实现？

常见实现方式：

1. 使用 CSS `transition` / `animation`
2. 使用 `react-transition-group`
3. 使用动画库，比如 `framer-motion`

如果是简单显隐切换，CSS 足够；如果涉及挂载卸载动画、列表动画、手势交互，通常会用专门库。

**为什么 CSS 解决不了某些场景？**

1. **挂载/卸载动画**：CSS 的 transition 需要元素先存在再切换 class，但 React 卸载组件时 DOM 直接被移除，没机会跑"离开动画"。`react-transition-group` 通过延迟卸载（先标记 leaving 类，等动画结束再 unmount）解决这件事。
2. **FLIP 列表动画**：列表重排时，CSS 不知道"这一项以前在哪"。FLIP 技术（First / Last / Invert / Play）需要在重排前后分别测量位置、计算 transform、再播放过渡——这是命令式的，必须用库（`framer-motion` 的 `<Reorder>`、`react-flip-toolkit`）。
3. **手势交互**：拖拽、滑动、捏合缩放需要追踪触摸/鼠标事件、计算速度、应用 inertia——纯 CSS 完全做不到，要么自己实现（很复杂），要么用 `react-spring` / `framer-motion` 这类带物理引擎的库。
4. **联动动画**：A 元素动画结束后 B 才开始、AB 同时但有延迟差、按数据驱动的复杂时间轴——CSS 的 `animation-delay` 只能写死，不能根据 props 动态生成；JS 库可以编程组合时间轴。
5. **可中断/可逆**：CSS 动画跑到一半被打断会"跳变"；spring 物理动画可以从中断处平滑反向继续，体验自然得多。

**选型决策树：**

| 场景 | 推荐 | 理由 |
| --- | --- | --- |
| 单元素 hover、show/hide、fade | CSS `transition` | 0 依赖、性能最好、GPU 加速 |
| Tab 切换、抽屉、Modal 进出 | CSS + `react-transition-group` | 处理 unmount 时机 |
| 列表重排、共享元素过渡（detail page） | `framer-motion` 的 layout 动画 | FLIP 自动化，零样板 |
| 拖拽、滑动、手势 | `react-spring` / `framer-motion` / `use-gesture` | 物理引擎、手势检测 |
| 复杂时间轴、SVG 路径动画 | GSAP（命令式）或 `framer-motion` | 时间轴控制、可逆 |
| 数据可视化大量动效 | D3 或 visx | 数据驱动 + DOM/SVG 精细控制 |

简单原则：**默认 CSS，不够用再上库**。但一旦决定上库，`framer-motion` 的 API 很现代，覆盖大部分日常需求，是现代 React 项目的首选。

### 说说你对 immutable 的理解？如何应用在 React 项目中？

Immutable 思想指的是数据更新时不直接修改原对象，而是返回一个新对象。

它在 React 中很重要，因为：

1. React 的渲染优化很多依赖引用变化判断。
2. 不可变更新更利于状态追踪和调试。
3. 能降低意外副作用。

实际应用方式：

1. 展开运算符创建新对象、新数组。
2. 使用 `map`、`filter` 等返回新数组的方法。
3. 在复杂场景用 Immer 等库辅助不可变更新。

### React 中不可变更新有哪些常见模式和坑？

不可变更新的核心原则只有一句：**永远不要原地修改 state，每次都返回新引用**。围绕这条原则，对象、数组、嵌套结构各有自己的写法，也各有自己的坑。

**1. 为什么 React 必须不可变更新？**

1. `setState` / `useState` 内部用 `Object.is` 比较新旧值，引用相同就会跳过本次更新——原地修改会让 UI 与数据脱节。
2. `PureComponent`、`React.memo`、`useMemo`、`useSelector` 这些优化全部依赖浅比较，引用没变就判定"没变"。
3. Redux DevTools、时间旅行调试、撤销/重做都依赖每个版本的 state 都是独立快照。

**2. 对象的不可变更新**

```js
// ❌ 原地修改：引用没变，组件不会重渲
state.user.name = 'Tom'
setState(state)

// ✅ 浅层字段：展开 + 覆盖
setState(prev => ({ ...prev, name: 'Tom' }))

// ✅ 删除字段：解构 + rest
setState(prev => {
  const { age, ...rest } = prev
  return rest
})
```

**3. 数组的不可变更新（重点记"哪些方法不能用"）**

| 操作 | ❌ 原地修改（会破坏不可变） | ✅ 返回新数组 |
| --- | --- | --- |
| 添加 | `push`、`unshift` | `[...list, item]`、`[item, ...list]` |
| 删除 | `pop`、`shift`、`splice` | `list.filter(...)`、`list.slice(...)` |
| 替换 | `arr[i] = x`、`splice` | `list.map((v, i) => i === idx ? x : v)` |
| 排序 | `sort`、`reverse` | `[...list].sort()`、`list.toSorted()`（ES2023） |

```js
// 在指定位置插入
const insertAt = (list, idx, item) => [...list.slice(0, idx), item, ...list.slice(idx)]

// 按 id 更新某一项
setList(prev => prev.map(item => item.id === id ? { ...item, done: true } : item))
```

**4. 嵌套结构的更新（最容易踩坑的地方）**

不可变更新只需要"**修改路径上**的所有节点都换新引用"，没修改的兄弟节点保持原引用即可。

```js
// state = { user: { profile: { name: 'A', age: 18 } } }
// 修改 user.profile.name
setState(prev => ({
  ...prev,
  user: {
    ...prev.user,
    profile: { ...prev.user.profile, name: 'B' },
  },
}))
```

层级一深，手写展开会变成"金字塔"。两种解法：

1. **状态扁平化**：把嵌套对象拆成 `byId` + `ids` 的范式结构，从源头避免深嵌套。
2. **用 Immer 的 `produce`**：写起来像可变，背后产新对象。

```js
import { produce } from 'immer'

setState(produce(draft => {
  draft.user.profile.name = 'B' // 看似可变，结果是全新对象
}))
```

Redux Toolkit 的 `createSlice`、Zustand 的 `immer` 中间件都内置了 Immer，直接 `state.x.y = z` 就是不可变更新。

**5. 常见坑**

1. **以为 `concat`、`slice` 是原地修改**：它们其实返回新数组，是不可变更新的好朋友；真正要警惕的是 `push/pop/shift/unshift/splice/sort/reverse/fill/copyWithin` 这一类。
2. **`useState` 传同一个引用不会触发更新**：`setList(list)` 即使内容变了，引用没换 React 也不会渲染。
3. **`useEffect` 依赖项是对象/数组**：每次 render 都是新引用 → 副作用每次都执行；要么用 `useMemo` 稳定引用，要么把依赖收敛成原始类型。
4. **深拷贝不是不可变更新**：`JSON.parse(JSON.stringify(x))` 会丢函数、Date、Map、Set，且性能差；只在路径上换引用就够了。
5. **冻结也不等于不可变更新**：`Object.freeze` 只是防止你修改，并不会自动产新对象，常用于开发期检查。

**6. 工程上的最小心智模型**

1. **简单结构**：手写 `{ ...obj }` / `[...arr]`。
2. **层级深 / 操作多**：上 Immer。
3. **大量列表更新**：状态扁平化（`byId` + `ids`），更新一项时间复杂度从 O(n) 降到 O(1)。
4. **新 API 可关注**：`structuredClone` 适合一次性拿到深拷贝（不带函数），`Array.prototype.toSorted/toReversed/toSpliced/with` 提供原生不可变数组操作（ES2023）。

一句话总结：**不可变更新不是为了"避免 bug"，而是为了让 React 的引用比较机制能正确工作**——这是 memo、selector、时间旅行调试、并发渲染一切上层能力的基石。

## 四、路由与状态管理

### 说说你对 Redux 的理解？其工作原理是什么？

Redux 是一个可预测的状态管理库，核心思想是用统一 store 管理全局状态，所有状态变更必须通过纯函数完成。

**核心概念：**

1. `store`：保存整棵应用状态树的对象，全局唯一。
2. `state`：store 中保存的当前数据快照，是不可变的。
3. `action`：描述"发生了什么"的纯对象，必须有 `type` 字段。
4. `reducer`：`(state, action) => newState` 的纯函数，根据 action 计算新状态。
5. `dispatch`：派发 action 触发状态更新的方法。
6. `subscribe`：订阅 store 变化的方法，状态更新后会通知所有订阅者。

**工作流程：**

1. 视图层调用 `store.dispatch(action)`。
2. store 内部把当前 state 和 action 交给 reducer。
3. reducer 返回一份新的 state，store 替换旧 state。
4. store 遍历所有订阅者，触发回调。
5. 通过 `react-redux` 的 `useSelector`，组件拿到自己关心的部分 state；如果通过 selector 算出来的值变了（默认引用比较），组件就重新渲染。

**Redux 的三大原则：**

1. **单一数据源**：整个应用只有一棵状态树，便于调试、回放、序列化。
2. **状态只读**：唯一改变状态的方式是 dispatch action，禁止直接修改。
3. **纯函数变更**：reducer 必须是纯函数，无副作用、相同输入相同输出，这是时间旅行调试的基础。

**为什么要用不可变更新？**

1. 引用比较代价远低于深比较，便于性能优化。
2. 历史快照可以保留，调试时能回放任意时刻状态。
3. 避免共享引用带来的意外副作用。

实践中往往配合 Immer 用 `produce` 写"看似可变、实际不可变"的代码。

### 说说对 Redux 中间件的理解？常用中间件有哪些？实现原理是什么？

Redux 中间件的作用是在 `dispatch` 和 `reducer` 之间插入一层切面，扩展 dispatch 的能力。

**典型用途：**

1. 处理异步逻辑（thunk、saga、observable）。
2. 日志记录、错误上报。
3. 请求拦截、权限校验。
4. 持久化、状态同步。

**常见中间件：**

1. `redux-thunk`：让 action 可以是函数，函数会被注入 `dispatch` 和 `getState`，常用于简单异步。
2. `redux-saga`：基于 generator 函数，把副作用抽离成可独立测试的 saga，适合复杂异步流。
3. `redux-observable`：基于 RxJS，把 action 流当作 Observable 处理。
4. `redux-logger`：开发环境打印 action 前后状态。
5. `RTK Query`：Redux Toolkit 内置的数据获取方案，自动管理缓存和请求状态。

**中间件签名：**

```js
const middleware = ({ getState, dispatch }) => next => action => {
  // next 是被前一个中间件包过的 dispatch
  // 调用 next(action) 表示放行到下一个中间件
  // 调用 dispatch(action) 表示重新从头走一遍中间件链
  return next(action)
}
```

这个签名是三层柯里化，对应中间件生命周期的三个阶段：

1. **第一层 `({getState, dispatch})`**：在 `applyMiddleware` 阶段调用一次，注入 store API，让中间件可以读状态和重新派发 action。
2. **第二层 `next => `**：把所有中间件的"下一层 dispatch"串成洋葱链。`next` 总是指向下一层中间件包过的 dispatch，最里层那个 `next` 就是 store 原始的 dispatch。
3. **第三层 `action => `**：每次业务调用 `dispatch(action)` 时进入。中间件可以在这里做任何事——log、判断 action 类型、改写 action、异步触发新的 dispatch、不调 next 直接吞掉等等。

**实现本质：**

1. `applyMiddleware` 接收所有中间件，给它们一个简化版 `dispatch`。
2. 用 `compose` 把所有 `next => action => result` 串起来，形成洋葱模型。
3. 把组合后的最外层函数赋给 store 的 dispatch，覆盖原始 dispatch。

`redux-thunk` 实现非常简短：

```js
const thunk = ({ dispatch, getState }) => next => action => {
  if (typeof action === 'function') {
    return action(dispatch, getState)
  }
  return next(action)
}
```

整个 thunk 就做一件事：判断 action 是不是函数。如果是，就把 `dispatch` 和 `getState` 注入这个函数并执行——这意味着用户可以写"action creator 返回一个异步函数"，函数里随便发请求、`dispatch` 多个普通 action、读当前 state；如果不是函数，就走默认的 `next(action)` 流程，把这个普通 action 交给后续中间件和 reducer。

短短几行就让 action creator 可以返回函数处理异步，这也是为什么很多人觉得"中间件就是装饰器"。

### Redux Toolkit 解决了什么？为什么是现在的官方推荐？

Redux 早期被诟病"样板代码太多"——一个简单功能要写 action types、action creators、reducers 三套文件，再加上不可变更新的繁琐。Redux Toolkit（RTK）就是为解决这些问题而生。

**主要 API：**

1. `configureStore`：开箱即用的 store 配置，默认集成 thunk、devtools、序列化检测。
2. `createSlice`：把 action types、action creators、reducer 合并为一个 slice，自动生成。
3. `createAsyncThunk`：标准化处理异步请求的 pending/fulfilled/rejected 三个状态。
4. `createEntityAdapter`：列表/字典数据的 CRUD 工具。
5. `RTK Query`：声明式数据获取，自动缓存、失效、重取。

**createSlice 示例：**

```js
const counter = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    // 看起来像可变更新，实际背后用 Immer 生成新对象
    increment(state) {
      state.value += 1
    },
    add(state, action) {
      state.value += action.payload
    },
  },
})

export const { increment, add } = counter.actions
export default counter.reducer
```

这一段相当于过去 Redux 三件套的完整替代：

1. **`name: 'counter'`** 是 slice 标识符，会自动拼接成 action type 前缀（如 `counter/increment`），不再需要手写 `const INCREMENT = 'counter/INCREMENT'`。
2. **`reducers` 里每个方法既是 reducer，也对应自动生成的 action creator**：访问 `counter.actions.increment` 得到的就是一个 action creator，`counter.actions.increment()` 返回 `{ type: 'counter/increment' }`。
3. **看似可变的写法**：`state.value += 1` 在原始 Redux 里是大忌，但 RTK 内部用 Immer 的 `produce` 包了一层 reducer，你直接修改 draft，Immer 在背后产出全新对象。代码可读性大幅提升，且不可变性依然保留。
4. **带 payload 的写法**：`add(state, action) { state.value += action.payload }`，action creator 接收的参数会自动放进 `action.payload`，无需手写 action creator 函数。
5. **导出方式**：`counter.actions` 是所有 action creators 的集合，`counter.reducer` 是合并好的 reducer 函数，可以直接交给 `configureStore({ reducer: { counter: counter.reducer } })`。

**RTK 的本质优势：**

1. 内置 Immer，告别手写 spread 链。
2. 默认配置合理，新手不用研究 store enhancer。
3. 自动生成 action types 和 creators，类型推导友好。
4. 配合 RTK Query 直接替换大部分手写请求 hook。

如果新项目还在用 Redux，应该直接用 RTK，而不是手撸 reducer。

### redux-thunk 和 redux-saga 有什么区别？

两者都解决"action 不能是异步"的问题，但思路不同。

| 维度 | redux-thunk | redux-saga |
| --- | --- | --- |
| 核心机制 | 让 action 可以是函数 | 用 generator 描述副作用 |
| 学习曲线 | 极低，几乎零成本 | 较高，需要理解 generator 和 effect |
| 测试友好度 | 一般，函数里直接调用 dispatch | 高，generator yield 出的是描述对象 |
| 复杂流程 | 写起来像回调地狱 | 擅长串行/并行/取消/竞态 |
| 包大小 | 极小（< 1KB） | 较大（约 20KB） |
| 适合场景 | 简单异步、CRUD | 实时数据流、长事务、轮询、WebSocket |

举个对比：处理"搜索请求需要防抖且能被新请求取消"的场景，thunk 写起来比较绕，saga 用 `takeLatest` 一行搞定。

### 你在 React 项目中是如何使用 Redux 的？项目结构如何划分？

实际项目中一般会这样用：

1. 按业务模块拆分 state。
2. 把 action、reducer、selector、异步逻辑按领域组织。
3. 组件尽量通过 hooks 或连接层消费状态，避免直接耦合过深。

常见目录思路：

1. `store`
2. `modules` 或 `features`
3. `hooks`
4. `api`
5. `components`

如果项目用 Redux Toolkit，会更推荐按 feature 组织（每个 feature 一个 slice + 对应组件 + 对应 RTK Query endpoint），而不是把 action 和 reducer 完全分离。

### 说说你对 MobX 的理解？它的实现思想是什么？

MobX 是一个基于响应式编程思想（TFRP，Transparent Functional Reactive Programming）的状态管理库，核心理念是："**任何源于应用状态的东西都应当自动地获得**"。换句话说，UI 是状态的派生，状态变了 UI 自然变。

**核心三件套：**

1. **State**：被 `observable` 标记的可变状态，是数据源。
2. **Derivation**：从状态派生出来的值，分两种：
   - `computed`：纯派生值，有缓存。
   - `reaction`：会产生副作用的派生（如重新渲染、发请求）。
3. **Action**：修改 state 的方法。MobX 严格模式下要求所有变更必须在 action 里，便于事务批处理和调试。

**关键 API：**

| API | 作用 |
| --- | --- |
| `observable` / `makeAutoObservable` | 把对象/类标记为可观察 |
| `computed` | 声明派生值 |
| `action` / `runInAction` | 标记修改 state 的函数 |
| `autorun(fn)` | 自动追踪依赖，依赖变就重跑 fn |
| `reaction(data, effect)` | 显式声明依赖，依赖变才跑 effect |
| `when(predicate, effect)` | 条件成立后跑一次 effect |
| `observer(Component)` | 让组件成为 reaction，自动按需重渲 |
| `flow` | 用 generator 写异步 action |

**完整示例：**

```js
import { makeAutoObservable, runInAction } from 'mobx'
import { observer } from 'mobx-react-lite'

class TodoStore {
  list = []
  filter = 'all'

  constructor() {
    makeAutoObservable(this, {}, { autoBind: true })
  }

  // computed —— 自动缓存，依赖未变直接用旧结果
  get visible() {
    if (this.filter === 'done') return this.list.filter(t => t.done)
    if (this.filter === 'todo') return this.list.filter(t => !t.done)
    return this.list
  }

  // action —— 同步变更
  add(text) {
    this.list.push({ text, done: false })
  }

  toggle(idx) {
    this.list[idx].done = !this.list[idx].done
  }

  // 异步 action 必须用 runInAction 或 flow 包裹
  async fetchInitial() {
    const data = await api.getTodos()
    runInAction(() => {
      this.list = data
    })
  }
}

const store = new TodoStore()

const TodoList = observer(() => (
  <ul>
    {store.visible.map((t, i) => (
      <li key={i} onClick={() => store.toggle(i)}>
        {t.done ? '✓' : '·'} {t.text}
      </li>
    ))}
  </ul>
))
```

这个示例可以拆成五个角色看：

1. **State**：`list` 和 `filter` 是普通的类字段，被 `makeAutoObservable` 包装后变成可观察值，每次读写都会被 MobX 拦截。
2. **Computed**：`get visible()` 是 getter，会被自动识别成 `computed`，结果会缓存——只要 `list` 和 `filter` 没变，多次访问 `store.visible` 不会重复计算。
3. **Action**：`add` 和 `toggle` 是同步 action，可以直接修改可观察对象，多次修改会被合并到同一次响应。
4. **异步 action**：`fetchInitial` 中 `await` 之后属于另一个 microtask，已经脱离原 action 上下文，所以必须用 `runInAction` 重新开一个 action 区块，否则严格模式会报错。
5. **Observer**：`observer(() => ...)` 把组件包成一个 reaction，render 期间读到的字段都会被收集为依赖；`store.filter` 改变时，组件就会重新执行 render。

这样写下来既保留了"直接改对象"的直观，又保证了响应式的精准更新。

### MobX 的依赖收集是怎么工作的？

MobX 的精髓在于"**自动**依赖收集"，无需手写 selector。底层模型由三类对象支撑：

1. **Atom**：可观察值的最小单元。每个 observable 属性背后对应一个 atom。
2. **Derivation**（observer / autorun / computed / reaction 都属于这一类）：在自己执行期间会进入"追踪态"。
3. **全局追踪栈**：MobX 维护一个"当前正在追踪的 derivation"指针。

**收集流程：**

1. derivation 开始执行（如 React 组件 render），把自己压进追踪栈。
2. 执行过程中读取了某个 observable.x，会触发 atom 的 `reportObserved()`。
3. atom 把当前栈顶的 derivation 加入自己的"观察者列表"，反向 derivation 也记录依赖了哪些 atom。
4. derivation 执行完，从追踪栈弹出。
5. 后续如果 atom 被修改（atom.reportChanged），就遍历观察者列表，把所有依赖它的 derivation 标脏。
6. 在事务结束时，统一调度被标脏的 derivation 重新执行。

这个机制带来几个特征：

1. **粒度极细**：组件只依赖它真正读过的字段，没读的字段变了不会触发重渲。
2. **依赖动态变化**：`if (a) return b; else return c;` 这种条件读取，依赖会随 a 变化而切换。
3. **零样板**：不用写 mapStateToProps 或 selector，怎么用就怎么收集。

**伪代码理解：**

下面这段伪代码刻画了上述三类对象之间的协作。`Atom` 负责"读时登记、写时通知"；`Derivation` 负责"运行前清空旧依赖、运行中收集新依赖"；全局变量 `currentDerivation` 是把读写挂钩起来的关键——它告诉 `Atom`："当前正在执行的 derivation 是谁，要把我加到它的依赖里"。

```js
let currentDerivation = null

class Atom {
  observers = new Set()
  reportObserved() {
    if (currentDerivation) {
      this.observers.add(currentDerivation)
      currentDerivation.deps.add(this)
    }
  }
  reportChanged() {
    this.observers.forEach(d => d.schedule())
  }
}

class Derivation {
  deps = new Set()
  run(fn) {
    this.deps.clear()
    const prev = currentDerivation
    currentDerivation = this
    try { fn() } finally { currentDerivation = prev }
  }
  schedule() {
    queueMicrotask(() => this.run(this.fn))
  }
}
```

逐行解读：

1. `reportObserved` 在每次属性被读取时调用，建立"双向引用"——Atom 知道有谁在监听自己，Derivation 也知道自己依赖了哪些 Atom。双向是为了卸载时能清理干净。
2. `reportChanged` 在属性被修改时调用，遍历所有观察者并安排重新执行。
3. `Derivation.run` 每次执行前先清空旧依赖（`deps.clear()`），是为了支持"依赖动态变化"——比如 if/else 切换分支后，依赖集合会自然更新。
4. 用 `try/finally` 恢复 `prev`，是为了支持嵌套的 derivation（一个 derivation 在执行过程中可能触发另一个 derivation 的初始化）。
5. `schedule` 用 `queueMicrotask` 是为了在同一个事件循环中合并多次变更，只触发一次 re-run。

这就是为什么"在 observer 组件里读 store.x"等价于"自动 useSelector(s => s.x)"。

### MobX 中 action、runInAction、flow 有什么区别？

三者都是"修改 state 的入口"，但适用场景不同。

1. **`action`**：标记一个**同步**函数为 action。多次修改会被合并到一个事务，只触发一次响应。
2. **`runInAction`**：在异步代码中临时开一个 action 区块，常用于 `await` 之后包裹 state 修改。
3. **`flow`**：用 generator 写异步 action，每个 `yield` 处自动 runInAction，避免到处写 try/catch + runInAction。

```js
// flow 示例
import { flow } from 'mobx'

class Store {
  list = []
  loading = false
  fetchList = flow(function* () {
    this.loading = true
    try {
      this.list = yield api.fetch()
    } finally {
      this.loading = false
    }
  })
}
```

为什么要用 generator 而不是 `async/await`？因为 MobX 需要在每个异步暂停点（即 `yield`）之后重新进入 action 上下文。用 `async/await` 时 await 之后的代码已经脱离原函数调用栈，MobX 没办法自动识别"这里仍在 action 中"；而 generator 是 MobX 自己驱动 `next()` 调用的，它知道每个 `yield` 之后的恢复点，可以自动包一层 `runInAction`。代码读起来像同步，写起来不用手动 wrapper，两全其美。

严格模式（`enforceActions: 'observed'` / `'always'`）下，不写 action 直接改 state 会报错，这是大型项目推荐的配置。

### MobX 4、5、6 有什么区别？

主要是底层拦截机制和 API 形式的演进。

| 版本 | 拦截机制 | 装饰器 | 主推 API |
| --- | --- | --- | --- |
| MobX 4 | `Object.defineProperty` | 必须用 | `@observable`、`@action` |
| MobX 5 | `Proxy`（兼容性要求 ES2015+） | 可选 | 同上 |
| MobX 6 | `Proxy` | 默认不用 | `makeObservable` / `makeAutoObservable` |

**MobX 4 → 5 的关键差异：**

1. v4 的 `defineProperty` 不能感知"动态新增属性"和"数组下标赋值"，要用 `set` API。
2. v5 用 Proxy 后，新增属性、数组下标都能自动响应，写起来更自然。

**MobX 6 的最大变化：**

1. 不再依赖装饰器（装饰器规范多次反复，社区抗拒），改成在 constructor 里调用 `makeObservable(this, {...})` 显式声明。
2. 提供 `makeAutoObservable` 一行搞定，自动推断每个属性是 observable / computed / action。

新项目推荐直接用 MobX 6 + `makeAutoObservable`。

### MobX 在 React 中的渲染优化和常见陷阱？

**渲染优化机制：**

1. `observer` 组件本质是把"组件 render 函数"包成一个 reaction。
2. render 期间读到的每个 observable 字段，都会被收集为依赖。
3. 当且仅当这些字段变化时，组件才会重新渲染，粒度细到字段级。

**常见陷阱：**

1. **解构丢失响应**：`const { count } = store; return <div>{count}</div>` 不会响应；要在 render 内读 `store.count`。
2. **传递整个 store 给非 observer 子组件**：子组件不会响应变化。要么用 observer 包裹，要么传具体值。
3. **在 useEffect 里读 observable**：useEffect 不在追踪栈里，需要用 `reaction` 显式订阅。
4. **对 `Map`/`Set` 用 `observable` 时**：要用 MobX 提供的 `observable.map` / `observable.set`，不能直接 new。
5. **数组操作要用 MobX 友好的方法**：`push` / `splice` / 索引赋值都没问题，但避免 `for...in`。

`mobx-react` 和 `mobx-react-lite` 的区别：

1. `mobx-react-lite`：只支持函数组件，体积更小。
2. `mobx-react`：支持类组件 + Provider/inject 等老 API。

新项目优先选 `mobx-react-lite`。

### 说说你对 Zustand 的理解？它解决了什么问题？

Zustand（德语"状态"）是一个极简的 React 状态管理库，作者是 React Three Fiber 的开发者 Paul Henschel。它的设计理念是："**Bring your own boilerplate**"——不强制任何范式，最小 API 表面积，按需扩展。

**与 Redux/MobX 的根本不同：**

1. **不依赖 Context**：store 是模块级单例，直接 import 使用，不需要 Provider 套娃。
2. **状态本身可变看起来**：用 `set` 函数更新，但底层始终是替换式新对象（默认浅合并）。
3. **基于 hooks**：每个 store 就是一个自定义 hook。
4. **零依赖**：核心包压缩后约 1KB。

**主要特点：**

1. API 极简，一个 `create` 函数搞定 state、action、selector。
2. 选择器配合浅比较实现按需订阅，避免 Context "all or nothing" 重渲。
3. 内置中间件支持：`persist`、`devtools`、`immer`、`subscribeWithSelector`、`combine`。
4. 可在 React 之外使用（vanilla store）。
5. 完整支持 React 18 并发模式（基于 `useSyncExternalStore`）。

**完整示例：**

```js
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'

const useBearStore = create(
  devtools(
    persist(
      (set, get) => ({
        bears: 0,
        increase: () => set(state => ({ bears: state.bears + 1 })),
        reset: () => set({ bears: 0 }),
        // 可以在 action 内部读其他状态
        double: () => set({ bears: get().bears * 2 }),
      }),
      { name: 'bear-storage' },
    ),
  ),
)

// 使用：传 selector 实现按需订阅
function Counter() {
  const bears = useBearStore(state => state.bears)
  const increase = useBearStore(state => state.increase)
  return <button onClick={increase}>{bears}</button>
}
```

这个例子可以对照 Redux 的写法看出 Zustand 的极简：

1. **`create` 接收一个工厂函数**：参数是 `(set, get) => state`，返回值就是 store 的初始状态 + 操作方法。state 和 action 写在一起，没有 reducer / dispatch / action types 的拆分。
2. **`set` 默认是浅合并**：`set({ bears: 0 })` 等价于 `Object.assign(prevState, { bears: 0 })`，不需要展开旧状态。
3. **`get()` 用来在 action 内部读其他字段**：避免依赖外部闭包变量，保证读到的总是最新值。
4. **中间件像洋葱套娃**：从外到内是 `devtools → persist → 实际配置`，请求会逐层穿过包装，最终到达 store。`persist` 负责把 state 写入 localStorage（key 由 `name` 指定），`devtools` 负责把每次 set 上报到 Redux DevTools。
5. **消费时务必传 selector**：`useBearStore(s => s.bears)` 表示"我只关心 bears 字段"，只有它变化才会让组件重渲；如果直接 `useBearStore()` 不传 selector，整个 state 任何字段变都会触发重渲，严重退化性能。

### Zustand 的实现原理是什么？

Zustand 内部其实非常薄，核心思路三句话：**一个 vanilla store + useSyncExternalStore + selector 浅比较**。

**Vanilla store 实现：**

下面这段是 Zustand 内部 `createStore` 的简化版本——不依赖 React，可以独立运行在 Node、Worker 或测试环境里。它只做三件事：保存 state、提供 setState、维护订阅者列表。

```js
function createStore(createState) {
  let state
  const listeners = new Set()

  const setState = (partial, replace) => {
    const nextState = typeof partial === 'function' ? partial(state) : partial
    if (!Object.is(nextState, state)) {
      const prev = state
      state = replace ? nextState : Object.assign({}, state, nextState)
      listeners.forEach(l => l(state, prev))
    }
  }

  const getState = () => state
  const subscribe = (listener) => {
    listeners.add(listener)
    return () => listeners.delete(listener)
  }
  const destroy = () => listeners.clear()

  const api = { setState, getState, subscribe, destroy }
  state = createState(setState, getState, api)
  return api
}
```

代码要点逐条对应：

1. `setState` 支持两种形式——直接传对象，或者传 `(prev) => next` 函数，跟 React 的 `setState` 完全一致。
2. 用 `Object.is` 做引用比较，避免相同状态多次通知；这也是为什么 Zustand 推荐"返回新对象"的不可变更新。
3. `replace` 参数控制是覆盖还是浅合并：默认 `false` 走 `Object.assign(...)` 浅合并，`true` 则直接用新状态替换整个 state。
4. `listeners` 用 `Set` 而不是数组，是为了 `subscribe` 返回的取消订阅函数 `O(1)` 删除，并且天然去重。
5. 最后一行 `state = createState(setState, getState, api)` 把用户工厂函数运行一次，把返回值作为初始状态，同时把 setState/getState 注入给 action 闭包。

**接入 React：**

```js
function create(createState) {
  const api = createStore(createState)
  const useStore = (selector = s => s, equalityFn = Object.is) =>
    useSyncExternalStore(
      api.subscribe,
      () => selector(api.getState()),
      () => selector(api.getState()),  // SSR snapshot
    )
  return Object.assign(useStore, api)
}
```

接入层做了两件事：

1. **把 store 包成一个 hook**：每次组件 render 时调用 `useSyncExternalStore`，第一个参数是订阅函数，第二个参数是当前快照的 getter，第三个参数是 SSR 时的快照。React 会自己处理订阅、取消订阅、缓存快照、并发模式下避免 tearing 等所有边界。
2. **`Object.assign(useStore, api)`**：让返回的 hook 同时拥有 `setState`/`getState`/`subscribe` 等方法，所以可以在 React 之外这样用：`useBearStore.getState()`、`useBearStore.subscribe(...)`，不必再绕一圈。

`useSyncExternalStore` 是 React 18 提供的官方"接外部 store"接口，保证：

1. 并发模式下不会出现 tearing（不同组件读到不同版本的 state）。
2. 自动处理订阅/取消订阅时机。
3. SSR 下能拿到一致的快照。

**选择器机制：**

```js
const bears = useBearStore(s => s.bears)             // 默认 Object.is 比较
const both = useBearStore(s => ({ a: s.a, b: s.b }), shallow)  // 返回对象时配 shallow
```

为什么第二行要传 `shallow`？因为 selector 返回的是一个**新对象** `{ a, b }`，每次 render 引用都不同，默认 `Object.is` 永远判定为"变了"，组件就会无限重渲。`shallow` 比较的是对象内的一层字段是否相等，避免这种假阳性。等价的另一种写法是用两个 selector 各取一个字段，那样就不需要 `shallow`。

如果不传 selector，组件订阅整个 state，任何变化都会触发重渲染——这正是不推荐的用法。

### Zustand 中间件机制是怎么工作的？

Zustand 中间件本质是"包装 set 函数"的高阶函数，签名：

```ts
type Middleware = (config: StateCreator) => StateCreator
```

读这条签名：中间件接收一个 `StateCreator`（即用户写的 `(set, get) => state`），返回另一个 `StateCreator`。它通常会在内部调用原 config，但把 `set` 替换成自己包装过的版本，这样所有用户调用 `set(...)` 时都会先经过中间件这一层。

**`devtools` 中间件简化版：**

```js
const devtools = (config) => (set, get, api) =>
  config(
    (...args) => {
      set(...args)
      window.__REDUX_DEVTOOLS_EXTENSION__?.send('action', get())
    },
    get,
    api,
  )
```

逐行看：返回的新 StateCreator 在执行用户配置时，把原始 `set` 替换成"先调原 set 更新状态，再把最新 state 上报给 Redux DevTools"的版本。`get` 和 `api` 透传不变。这样就实现了"每次状态变更都能被时间旅行调试器抓到"，而用户代码完全无感知。

**`immer` 中间件让你写"可变"代码：**

```js
const immer = (config) => (set, get, api) =>
  config(
    (partial) => set(produce(partial)),
    get,
    api,
  )
```

`produce` 是 Immer 提供的工具：它接收一个"看似可变"的 recipe，内部用 Proxy 跟踪你对草稿的修改，最后产出新的不可变对象。这里把 `set` 包成 `set(produce(partial))`，意味着用户可以写 `set(state => { state.list.push(item) })` 这种直观写法，但底层 state 仍然是不可变更新，引用比较依然有效。

中间件可以层层嵌套（洋葱模型），比如 `devtools(persist(immer(config)))` 表示：用户调用 `set` → immer 把"可变"变"不可变" → persist 把新状态写入 storage → devtools 上报到面板。每一层都只关心自己那一段逻辑，互相解耦。

**常用中间件：**

1. `persist`：把 state 持久化到 localStorage / sessionStorage / AsyncStorage。
2. `devtools`：接入 Redux DevTools 调试。
3. `immer`：用 Immer 实现"可变写法、不可变结果"。
4. `subscribeWithSelector`：扩展 subscribe，可以选择性订阅。
5. `combine`：合并多个 state slice。

### 大型 Zustand 项目怎么组织？slices 模式是什么？

随着 store 变大，把所有 state 写在一个 create 里会变成"巨型函数"。slices 模式是社区主流方案：每个业务模块写成一个工厂函数，工厂函数接收 `(set, get, api)`，返回该模块的状态和方法；最后在统一的 `create` 调用里把所有工厂函数执行一遍，把结果展开合并。

```js
const createBearSlice = (set) => ({
  bears: 0,
  addBear: () => set(s => ({ bears: s.bears + 1 })),
})

const createFishSlice = (set) => ({
  fishes: 0,
  addFish: () => set(s => ({ fishes: s.fishes + 1 })),
})

const useStore = create((...a) => ({
  ...createBearSlice(...a),
  ...createFishSlice(...a),
}))
```

这里有几个设计细节值得注意：

1. `(...a)` 是 `(set, get, api)` 的简写，原封不动透传给每个 slice，让它们共享同一个 store 实例。
2. `set` 在所有 slice 内部都是同一个函数，因此 bear slice 也可以读写 fish slice 的字段，跨模块协作天然支持。
3. 用对象展开 `...createBearSlice(...a)` 合并状态，等价于把所有 slice 的字段拍平到同一个 state 上。如果两个 slice 出现同名字段，后展开的会覆盖前面——所以命名时建议加业务前缀。
4. TypeScript 项目里可以给每个 slice 定义 `StateCreator<RootState, [], [], BearSlice>` 类型，让 `set`/`get` 自动推导出整个 RootState 的形状。

每个 slice 是一个独立的工厂函数，最终合并成一个 store。优点：

1. 文件按业务拆分，便于维护。
2. 不同 slice 可以互相调用 `get()` 协作。
3. 类型推导可以用 `StateCreator` 精确化。

要避免的反模式：每个 slice 单独 `create` 一个 store，会失去跨 slice 协作能力，也增加心智负担。

### Zustand 还有什么高级用法？

1. **transient subscriptions**：用 `useStore.subscribe` 订阅但不触发重渲染，常用于动画、`requestAnimationFrame` 场景。
2. **vanilla store**：不依赖 React，可以在 Node、Web Worker、原生 JS 项目里用。
3. **跨 store 派生**：用 `useShallow` 或 `subscribeWithSelector` 结合多个 store 派生新值。
4. **配合 React Query / SWR**：服务端状态交给 React Query，客户端 UI 状态交给 Zustand，是当下最流行组合。

### Redux 的 store 与 MobX、Zustand 的 store 有何不同？

前面分别介绍了三者，但落到"store 这个实体"上，它们的形态、约束和使用方式差别很大。可以从六个维度对比。

**1. Store 的本质形态**

| | 它实际是什么 |
| --- | --- |
| **Redux** | 一个普通对象，对外暴露 `{ getState, dispatch, subscribe, replaceReducer }` 四个方法 |
| **MobX** | 一个被 `makeAutoObservable` 处理过的 **class 实例**（或 Proxy 包装的对象），没有统一 API 表面 |
| **Zustand** | **函数 + API 的混合体**：本身是个 hook（可在组件里调用），同时挂着 `getState/setState/subscribe/destroy` 静态方法 |

Redux store 必须通过 4 个固定 API 访问；MobX store 就是个普通对象，怎么定义就怎么访问；Zustand store 既能当 hook 用，也能当普通对象用。

**2. 数量与作用域**

| | 单/多 store | 挂载方式 |
| --- | --- | --- |
| **Redux** | 强制单一 store（官方建议） | 必须用 `<Provider store={store}>` 包裹整棵树 |
| **MobX** | 鼓励多 store（按业务领域拆） | 不需要 Provider，直接 `import store` 即可；要做依赖注入再用 Context |
| **Zustand** | 单 / 多 store 都行 | **完全不需要 Provider**，store 是模块级单例 |

这导致项目结构差异明显：Redux 有"根 store"，MobX/Zustand 有"任意多个 store 实例"。

**3. 内部状态的存储形式**

| | state 的物理形态 | 修改方式 |
| --- | --- | --- |
| **Redux** | 一棵**严格不可变**的对象树，每次更新返回全新引用 | 只能 `dispatch(action)` → reducer 纯函数返回新对象 |
| **MobX** | **可变的响应式对象**，每个字段背后是一个 Atom，由 Proxy 拦截读写 | 直接 `store.x = 1`，Proxy 自动通知观察者 |
| **Zustand** | 普通对象，但**每次 setState 生成新对象**（默认浅合并） | 只能调用 `set(partial)` 或 `set(prev => next)` |

举个直观例子，"把 count 加 1"三种写法：

```js
// Redux
store.dispatch({ type: 'INC' })
// 或 RTK：dispatch(counter.actions.increment())

// MobX
store.count++          // 直接改

// Zustand
useCounter.setState(s => ({ count: s.count + 1 }))
```

**4. 订阅与变更通知**

| | 谁在订阅 | 通知粒度 |
| --- | --- | --- |
| **Redux** | 所有 `useSelector` 注册的 listener；store 不知道谁关心哪个字段 | **store 级广播**——任何 dispatch 都把所有 listener 跑一遍，由 selector 比较返回值决定要不要重渲 |
| **MobX** | 每个 Atom 自己记录"谁读过我"，store 没有全局 listener 列表 | **字段级精准**——只通知真正读过该字段的观察者 |
| **Zustand** | 所有 `useStore(selector)` 注册的 listener | **store 级广播 + selector 过滤**，机制和 Redux 同构，但比 Redux 轻 |

这是性能上最大的分水岭：Redux/Zustand 是"广播 + 过滤"，MobX 是"按需精准推送"。

**5. 异步与副作用入口**

| | 怎么发请求 |
| --- | --- |
| **Redux** | store 本身不支持异步，必须靠**中间件**（thunk、saga、RTK Query）扩展 dispatch |
| **MobX** | store 的方法直接 `async` 即可，await 之后用 `runInAction` / `flow` 包一层 |
| **Zustand** | store 的方法直接 `async` 即可，没有任何额外约束 |

Redux 的"store 内核纯净 + 中间件扩展"是历史包袱也是优点：清晰但啰嗦。MobX/Zustand 的 store 自身就能装异步逻辑，但相应失去了"action 是可序列化数据"的好处。

**6. 调试能力**

| | 时间旅行 | DevTools |
| --- | --- | --- |
| **Redux** | **天然支持**——每次 dispatch 都是离散事件，重放只需把 action 数组喂回去 | Redux DevTools 一等公民 |
| **MobX** | 不支持时间旅行——可变状态没有"快照流" | mobx-devtools，能看依赖图但不能回放 |
| **Zustand** | 通过 `devtools` 中间件**复用** Redux DevTools | 能查看每次 set，但不是真正的事件溯源 |

这也是 Redux 在大型项目里仍有价值的核心理由：**可追溯性**。

**一句话总结：**

1. **Redux 的 store 是"事件流的容器"**：所有变更都是 action，store 只负责按 action 回放出 state。
2. **MobX 的 store 是"响应式对象图"**：字段会自己广播变化，直接改对象就行。
3. **Zustand 的 store 是"带订阅能力的普通对象"**：状态用 set 替换，组件靠 selector 订阅切片。

选型时本质是在问："**我希望状态变化是显式事件、隐式响应、还是简单替换？**"答案决定了用谁。

### Redux、MobX、Zustand 全方位对比？怎么选型？

**1. 数据模型**

| 维度 | Redux | MobX | Zustand |
| --- | --- | --- | --- |
| store 数量 | 单 store | 多 store（按业务拆） | 单/多皆可 |
| 状态可变性 | 严格不可变 | 可变（Proxy 拦截） | 看似可变，实际新对象替换 |
| 数据归一化 | 推荐（normalized state） | 自由（图状结构常见） | 自由 |
| 派生值 | selector / reselect | computed（自动缓存） | selector（无缓存，需自管） |

**2. 订阅与更新机制**

| 维度 | Redux | MobX | Zustand |
| --- | --- | --- | --- |
| 订阅方式 | useSelector + 比较返回值 | observer 自动追踪 | useStore(selector) + 浅比较 |
| 更新粒度 | selector 返回值级 | 字段级（最细） | selector 返回值级 |
| 是否需要手写 selector | 需要 | 不需要 | 需要 |
| 并发模式支持 | 18 起完整 | mobx-react ≥ 9 完整 | 完整 |
| Tearing 风险 | 已修复 | 已修复 | 已修复 |

**3. API 风格 & 心智模型**

| 维度 | Redux | MobX | Zustand |
| --- | --- | --- | --- |
| 编程范式 | 函数式（reducer + action） | 面向对象 + 响应式 | 函数式 + Hooks |
| 样板代码 | 多（RTK 后大幅减少） | 少 | 极少 |
| 异步处理 | thunk / saga / RTK Query | runInAction / flow | 直接 async/await |
| 事务批处理 | 中间件层面 | 内置 transaction | set 一次即一次 |
| 学习曲线 | 陡（要懂 reducer/中间件） | 中（要懂依赖收集） | 平 |

**4. 工程化维度**

| 维度 | Redux | MobX | Zustand |
| --- | --- | --- | --- |
| 包大小（min+gzip） | redux 约 1.6KB，rtk 约 12KB | 约 16KB | 约 1.1KB |
| TypeScript 友好度 | RTK 极好 | 好（class 推导友好） | 极好 |
| DevTools | 强大（时间旅行） | 中（mobx-devtools） | 复用 Redux DevTools |
| 测试 | 极易（reducer 是纯函数） | 中（要 mock store） | 易 |
| SSR | 需要 hydrate 处理 | 需要小心避免单例 | 推荐 per-request store |
| 生态 | 最大 | 中 | 快速增长 |

**5. 性能特征**

1. **Redux**：依赖 selector，selector 写得好性能好；写得不好（每次返回新对象）会大量重渲。
2. **MobX**：精准到字段级订阅，性能基线最高，几乎无需手动优化。
3. **Zustand**：和 Redux 类似，但 selector 更轻；返回对象时配 `shallow` 比较即可。

**6. 适用场景**

| 场景 | 推荐 |
| --- | --- |
| 大型项目、强调可追踪性、团队协作 | Redux Toolkit |
| 中后台、表单复杂、对象关系多、派生值多 | MobX |
| 中小型项目、组件库、想替代 Context | Zustand |
| 极细粒度原子状态（如复杂表单单元） | Recoil / Jotai |
| 服务端状态为主 | React Query / SWR + 任意客户端 store |

**选型建议：**

1. 老项目已用 Redux：升级到 RTK + RTK Query，不要换库。
2. 团队有 OOP 背景、业务对象关系复杂：MobX 体验最舒服。
3. 新项目、追求轻量、想替代 Context：Zustand 是首选。
4. 状态极度分散、原子化程度高：考虑 Jotai。
5. **服务端状态和客户端状态分开管**：现代最佳实践是 React Query/SWR 管接口缓存，Zustand/Redux 管 UI 状态。

一句话总结：

1. **Redux**：可预测、可追踪、可时间旅行 —— 适合需要工程纪律的团队。
2. **MobX**：透明响应式、字段级粒度 —— 适合对象关系复杂的业务。
3. **Zustand**：极简、无 Provider、按需订阅 —— 适合现代轻量场景。

没有银弹，选型核心是看团队习惯、项目规模和心智模型契合度。

### 简单实现一个 mini-Redux？

Redux 的核心实现并不复杂，本质就是发布订阅 + 状态快照。下面这版包含了 `createStore`、`combineReducers`、`compose`、`applyMiddleware` 四个核心 API，一一对应官方源码的关键设计。

```js
function createStore(reducer, initialState, enhancer) {
  if (typeof enhancer === 'function') {
    return enhancer(createStore)(reducer, initialState)
  }

  let state = initialState
  const listeners = []

  function getState() {
    return state
  }

  function dispatch(action) {
    state = reducer(state, action)
    listeners.slice().forEach(fn => fn())
    return action
  }

  function subscribe(listener) {
    listeners.push(listener)
    return () => {
      const idx = listeners.indexOf(listener)
      if (idx >= 0) listeners.splice(idx, 1)
    }
  }

  dispatch({ type: '@@INIT' })

  return { getState, dispatch, subscribe }
}

// reducer 组合
function combineReducers(reducers) {
  return (state = {}, action) => {
    const next = {}
    let changed = false
    for (const key in reducers) {
      const prev = state[key]
      const cur = reducers[key](prev, action)
      next[key] = cur
      if (cur !== prev) changed = true
    }
    return changed ? next : state
  }
}

// compose 工具
const compose = (...fns) =>
  fns.length === 0 ? x => x : fns.reduce((a, b) => (...args) => a(b(...args)))

// applyMiddleware
function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, initialState) => {
    const store = createStore(reducer, initialState)
    let dispatch = () => { throw new Error('dispatch in setup phase') }
    const api = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args),
    }
    const chain = middlewares.map(m => m(api))
    dispatch = compose(...chain)(store.dispatch)
    return { ...store, dispatch }
  }
}
```

按四个 API 的职责拆开看：

1. **`createStore`**：这是最核心的"发布订阅 + 状态快照"。`dispatch` 调用 reducer 算出新 state，再把所有 listener 拉一遍。`listeners.slice()` 是为了避免遍历过程中其他订阅者取消订阅导致索引错乱（典型的"边遍历边修改"问题）。最后那行 `dispatch({ type: '@@INIT' })` 是 Redux 的小技巧——首次 dispatch 一个内部 action，让每个 reducer 都返回各自的初始值，从而组装出完整的 state 树。
2. **`combineReducers`**：把多个分领域的 reducer 合成一个 root reducer。关键点是 `changed` 标志：如果所有子 reducer 都返回了和原来一样的引用，就直接返回旧 state，避免顶层引用变化误触发 `useSelector`。这是 Redux 性能优化的根基。
3. **`compose`**：函数式编程里的常见工具，`compose(f, g, h)(x) === f(g(h(x)))`。Redux 用它把多个中间件按右往左串联，形成洋葱模型。
4. **`applyMiddleware`**：是一个 store enhancer。注意第一行 `let dispatch = () => { throw ... }`——这是为了防止中间件在初始化阶段提前调用 dispatch；只有所有中间件都注册完，最终的 dispatch 才被赋上真正的实现。`api` 对象里 dispatch 写成 `(...args) => dispatch(...args)` 而不是 `dispatch`，是为了延迟取值，让中间件拿到的永远是最新版本而非初始抛错版本。

整套核心代码不到 50 行，但足以支撑 React-Redux 这样庞大的生态。

### 简单实现一个 mini-MobX？

抓住"依赖收集 + 通知更新"两件事，不到 30 行就能跑：

```js
let activeReaction = null

function observable(target) {
  const atoms = new Map()  // key -> Set<reaction>

  return new Proxy(target, {
    get(t, key) {
      if (!atoms.has(key)) atoms.set(key, new Set())
      if (activeReaction) {
        atoms.get(key).add(activeReaction)
      }
      return t[key]
    },
    set(t, key, value) {
      if (t[key] === value) return true
      t[key] = value
      atoms.get(key)?.forEach(r => r())
      return true
    },
  })
}

function autorun(fn) {
  const reaction = () => {
    activeReaction = reaction
    try { fn() } finally { activeReaction = null }
  }
  reaction()
}

// 使用
const state = observable({ count: 0, name: 'mobx' })
autorun(() => console.log('count:', state.count))   // 输出 count: 0
state.count = 1                                       // 输出 count: 1
state.name = 'x'                                      // 不输出（autorun 没读 name）
```

理解的关键是把 `activeReaction` 当成"暗号"：

1. `autorun(fn)` 包装出一个 `reaction` 函数，它每次执行前会把自己赋值给全局 `activeReaction`，相当于举手喊"现在轮到我了，谁被读取请记下我"。
2. Proxy 的 `get` 拦截器检查 `activeReaction` 是否有值——有就把它加到这个 key 对应的 Set 里。这样每个 key 都有自己的"观察者集合"。
3. `set` 拦截器在赋值后遍历该 key 的观察者集合，挨个调用——也就是触发依赖该字段的 autorun 重新执行。
4. 因为重新执行时 `reaction` 会再次进入 `activeReaction` 状态、重新走一遍 `fn`，所以新的依赖也会被自然收集，旧的依赖无人维护就会被遗忘——这就是为什么 if/else 切换分支后依赖能动态更新。

最后一行的"不输出"演示了关键特性：**没读过的字段变化不会触发响应**——这是 MobX 性能基线高的根本原因。

真实 MobX 还会处理：computed 缓存、reaction 调度、批处理事务、循环依赖检测、清理无效依赖等，但骨架就是这么简单。

### 简单实现一个 mini-Zustand？

```js
import { useSyncExternalStore } from 'react'

function createStore(createState) {
  let state
  const listeners = new Set()

  const setState = (partial, replace) => {
    const next = typeof partial === 'function' ? partial(state) : partial
    if (!Object.is(next, state)) {
      const prev = state
      state = replace ? next : { ...state, ...next }
      listeners.forEach(l => l(state, prev))
    }
  }
  const getState = () => state
  const subscribe = (l) => {
    listeners.add(l)
    return () => listeners.delete(l)
  }

  state = createState(setState, getState)
  return { setState, getState, subscribe }
}

function create(createState) {
  const store = createStore(createState)
  function useStore(selector = s => s) {
    return useSyncExternalStore(
      store.subscribe,
      () => selector(store.getState()),
    )
  }
  return Object.assign(useStore, store)
}

// 使用
const useCounter = create((set) => ({
  count: 0,
  inc: () => set(s => ({ count: s.count + 1 })),
}))
```

可以从两层来理解：

1. **`createStore` 是纯 JS 的 store**：state 是普通变量，listeners 是 `Set`，setState 触发所有 listener。和 mini-Redux 几乎同构，只是没有 reducer/action 概念——直接用闭包里的方法 + `set` 修改状态。这一层完全不依赖 React，可以直接 `import` 在 Node 里用。
2. **`create` 是 React 适配层**：用 `useSyncExternalStore` 把外部 store 接进 React。这个 hook 是 React 18 专门为外部 store 设计的，三个参数分别是订阅函数、获取当前快照的函数、SSR 快照函数。React 内部会管理订阅、缓存快照、处理并发模式下的一致性，库作者只需要"把订阅和取值告诉它"。
3. **`Object.assign(useStore, store)`**：把 hook 和 store API 合并成一个对象，调用方既能 `useCounter(...)` 当 hook 用，又能 `useCounter.getState()` 在 React 之外用，API 表面积极小但能力齐全。

短短 30 行就实现了 Zustand 90% 的核心能力。这也解释了为什么 Zustand 包大小只有 1KB 左右——它真的只做了"够用就好"的事。

### 说说你对 React Router 的理解？常用的 Router 组件有哪些？

React Router 是 React 生态中的路由方案，用来管理前端页面切换和 URL 映射关系。

常见 Router 组件有：

1. `BrowserRouter`
2. `HashRouter`
3. `MemoryRouter`

它的核心作用是：让 URL 和组件树建立对应关系。

### 说说 React Router 有几种模式？实现原理是什么？

常见模式有：

1. `BrowserRouter`：基于 History API
2. `HashRouter`：基于 URL hash
3. `MemoryRouter`：基于内存记录，常用于测试或非浏览器环境

原理上：

1. `BrowserRouter` 通过 `pushState`、`replaceState` 和 `popstate` 管理路由。
2. `HashRouter` 通过监听 `hashchange` 实现路由切换。

## 五、渲染原理、调度与性能优化

### 说说 React render 方法的原理？在什么时候会被触发？

`render` 的职责是根据当前 `props` 和 `state` 返回一份 UI 描述。

它会在以下场景被触发：

1. 组件初始化渲染
2. `state` 更新
3. 父组件重新渲染导致子组件重新执行
4. `props` 变化
5. `context` 变化

要注意：触发 render 不等于一定会产生真实 DOM 变更，React 还会经过 diff 和提交过程。

### 说说你是如何提高组件的渲染效率的？在 React 中如何避免不必要的 render？

常见手段有：

1. 合理拆分组件，减少更新范围。
2. 保持状态就近原则，不要把局部状态提升过度。
3. 使用 `React.memo`。
4. 对计算结果使用 `useMemo`。
5. 对回调函数使用 `useCallback`。
6. 列表正确使用 `key`。
7. 避免在 render 中创建高成本计算。
8. 保持不可变更新，便于浅比较。

面试时要强调：优化的前提是先找到真正的性能瓶颈，而不是盲目 memo。

### 说说 React diff 的原理是什么？

React 的 diff 目标是高效比较新旧虚拟节点，找出最小更新范围。

常见原则：

1. 不同类型节点直接替换。
2. 同类型节点比较属性差异。
3. 列表节点借助 `key` 判断是否可复用。
4. 同层比较，不做跨层复杂比对。

这套策略牺牲了理论上的最优解，换来了工程上更高的性能和更低的复杂度。

### 说说对 Fiber 架构的理解？解决了什么问题？

Fiber 是 React 16 引入的全新协调架构，本质上是用**链表结构 + 自实现的调度循环**替换掉原来基于 JS 调用栈的递归遍历，把"一口气跑完整棵树"拆成"一个个可暂停、可恢复、可调度的工作单元"。

"Fiber"这个词在 React 里有**两层含义**：

1. **数据结构层面**：每个 Fiber 节点对应一个组件实例（或 DOM 节点），整棵 Fiber 树替代了原来的虚拟 DOM 树。
2. **架构/算法层面**：基于 Fiber 节点之上的整套调度、协调、提交机制。

**1. 它要解决什么问题？**

Stack Reconciler 时代是同步递归——一旦开始 diff 就停不下来。组件树一深，主线程一卡就是几十甚至上百毫秒，表现出来就是输入延迟、动画掉帧。Fiber 的目标就三个：

1. **可中断**：渲染到一半能让出主线程响应用户输入。
2. **可恢复**：让出之后能从断点继续，不用从头来。
3. **可调度**：不同更新分优先级，紧急的（输入、动画）插队到前面。

**2. Fiber 节点：链表化的虚拟 DOM**

每个 Fiber 节点大致长这样（精简版）：

```js
const fiber = {
  // 节点身份
  type,           // 函数、类、字符串（'div'）等
  key,            // diff 用的 key
  stateNode,      // 真实 DOM 或类组件实例

  // 链表指针（替代递归调用栈的关键）
  return,         // 父 Fiber
  child,          // 第一个子 Fiber
  sibling,        // 下一个兄弟 Fiber
  index,          // 在父节点里的位置

  // 数据
  pendingProps,   // 新 props
  memoizedProps,  // 上次渲染的 props
  memoizedState,  // 上次渲染的 state（函数组件里是 hooks 链表）
  updateQueue,    // 待处理的状态更新队列

  // 副作用相关
  flags,          // 这个节点要做的操作（Placement / Update / Deletion 等）
  subtreeFlags,   // 子树是否有副作用（用来跳过没变化的子树）
  deletions,      // 待删除的子节点

  // 双缓冲
  alternate,      // 指向另一棵树上对应的 Fiber

  // 调度
  lanes,          // 这个 Fiber 上的更新优先级
  childLanes,     // 子树上的优先级
}
```

关键点是 `return` / `child` / `sibling` 这三个指针——它们把树**线性化成一条可遍历的链表**，于是遍历可以变成一个 `while` 循环，而不是函数递归。能做到 `while` 循环，就能在每次循环时检查"时间够不够、要不要让出主线程"。

**3. 两棵树的双缓冲（current / workInProgress）**

React 在内存里**始终维护两棵 Fiber 树**：

1. `current`：当前显示在屏幕上的树。
2. `workInProgress`（WIP）：本次更新正在构建的树。

两棵树通过每个节点的 `alternate` 指针互相引用，复用上一次的 Fiber 节点（不是直接复用对象，而是把字段拷过来更新），减少 GC 压力。当 WIP 构建完成、commit 之后，WIP 摇身一变成为新的 `current`——这就叫**双缓冲（Double Buffering）**，思路类似图形渲染里的前后缓冲区切换。

带来的好处：

1. **构建中途可以丢弃**：渲染被中断或被更高优更新打断，直接扔掉 WIP 重新来过，`current` 完全不受影响、屏幕不会闪。
2. **可以在内存里安全 diff**：所有计算都在 WIP 上进行，不污染 `current`。

**4. 两阶段：Render 阶段（可中断）+ Commit 阶段（不可中断）**

这是 Fiber 架构最核心的拆分：

| 阶段 | 名称 | 是否可中断 | 做什么 |
| --- | --- | --- | --- |
| 1 | Render（也叫 Reconcile） | ✅ 可中断、可丢弃、可重做 | 构建 WIP 树，对每个 Fiber 调用 `beginWork` / `completeWork`，diff 出要做的副作用，挂在 `flags` 上 |
| 2 | Commit | ❌ 同步执行，不可中断 | 把副作用一次性应用到真实 DOM，触发生命周期 / effect |

Render 阶段为什么必须无副作用？因为它**可能被多次执行**——这就是 16.3 给 `componentWillMount`、`componentWillReceiveProps`、`componentWillUpdate` 打 `UNSAFE_` 前缀的根本原因：它们在 Fiber 下可能被调用多次，写副作用就会出 bug。

Render 阶段的工作循环（伪代码）：

```js
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress)
  }
}

function performUnitOfWork(fiber) {
  // 向下：创建子 Fiber
  const next = beginWork(fiber)
  if (next) return next
  // 没有子节点了，向上回溯并处理兄弟
  let node = fiber
  while (node) {
    completeWork(node)
    if (node.sibling) return node.sibling
    node = node.return
  }
  return null
}
```

`shouldYield()` 是关键——每处理完一个 Fiber，都问一句"浏览器要绘制了吗？"，时间快用完就主动 `return`，把控制权交还给浏览器。下次空闲时再从 `workInProgress` 接着跑。

Commit 阶段又分三小步：

1. **Before mutation**：读 DOM（`getSnapshotBeforeUpdate`）、调度 passive effect。
2. **Mutation**：执行 DOM 增删改、ref 解绑/绑定。
3. **Layout**：触发 `componentDidMount` / `componentDidUpdate` / `useLayoutEffect`，此时 DOM 已更新但浏览器还没绘制。

`useEffect` 不在 commit 里，是在 commit 后异步调度，避免阻塞 paint。

**5. 调度器（Scheduler）和时间切片**

Fiber 自带一个独立包 `scheduler`，提供：

1. **`requestIdleCallback` 的 polyfill**：早期用 `requestIdleCallback`，后来发现兼容性和帧率不稳，改用 `MessageChannel` 自己实现一套——每帧给 React 大约 5ms 的工作时间。
2. **优先级调度**：不同任务进不同的优先级队列，高优先级任务可以打断低优先级。

时间切片就是把一帧 16.6ms 切出一小段给 React，剩下时间还给浏览器去做样式计算、布局、绘制和响应输入。

**6. Lane 模型（React 17/18 替代 expirationTime）**

最初 Fiber 用一个递增的 `expirationTime` 数字表示优先级，后来发现表达力不够（无法表达"批量"和"重叠"），17 起换成 **Lane 模型**——31 个 bit 各自代表一条"车道"：

```
SyncLane              = 0b0000000000000000000000000000001
InputContinuousLane   = 0b0000000000000000000000000000100
DefaultLane           = 0b0000000000000000000000000010000
TransitionLane1       = 0b0000000000000000000000001000000
...
IdleLane              = 0b0100000000000000000000000000000
```

用位运算批量合并、判断包含、抽取最高优——既能表达单个优先级，也能表达"一组优先级一起处理"。`useTransition` 标记的更新会落到 Transition lane，可以被普通输入打断；`useDeferredValue` 类似。

**7. 副作用收集（flags / subtreeFlags）**

Render 阶段并不直接改 DOM，而是把"这个节点要做什么"打在 `flags` 字段上：

```
Placement   // 插入
Update      // 更新
Deletion    // 删除
Ref         // 处理 ref
Passive     // useEffect
Layout      // useLayoutEffect / componentDidMount
```

老版本用一条 `effectList` 链表收集所有有副作用的 Fiber，commit 时遍历执行。React 18 改成 `subtreeFlags` 位图——commit 时往下走，发现某个子树 `subtreeFlags` 全为 0 就直接跳过，省去链表维护的开销。

**8. 一次完整更新的完整流程**

以一次 `setState` 为例：

```
setState
  ↓
创建 update 对象，挂到 fiber.updateQueue
  ↓
scheduleUpdateOnFiber：从触发点 fiber 沿 return 指针向上冒泡，给祖先节点的 childLanes 都打上对应优先级
  ↓
ensureRootIsScheduled：通知 Scheduler 该 root 有任务了
  ↓
Scheduler 在合适时机调用 performConcurrentWorkOnRoot
  ↓
Render 阶段：从 root 开始构建 WIP，beginWork → completeWork，每步检查 shouldYield
  ↓ (可能被打断、丢弃、重启 N 次)
Render 完成，进入 Commit 阶段
  ↓
Before mutation → Mutation → Layout（同步、不可中断）
  ↓
WIP 变 current，flushPassiveEffects（异步执行 useEffect）
```

**9. Fiber 带来的能力总结**

1. **并发渲染（Concurrent Rendering）**：18 的 `startTransition`、`useDeferredValue`、自动批处理都建立在 Fiber 之上。
2. **Suspense**：渲染中途遇到未 ready 的资源，可以"挂起"这棵 WIP，先 commit 一个降级 UI，等数据回来再恢复。
3. **错误边界**：在 completeWork 阶段捕获错误，向上找最近的 Error Boundary。
4. **Hooks 实现的基础**：函数组件的 hooks 状态就挂在对应 Fiber 节点的 `memoizedState` 上（一条单链表），每次 render 按调用顺序遍历——这也是为什么 hooks 不能写在条件分支里。

**10. 一句话面试总结**

Fiber 不是某个新算法，而是 **一种把"组件树渲染"改造成"可被调度的工作单元流"的架构**——它通过链表结构让递归变成可中断的循环，通过双缓冲让中断可丢弃，通过双阶段让副作用集中可控，通过 Lane 模型支持优先级并发，最终让 React 从"同步递归框架"演化成了"并发 UI 运行时"。

### 说说 React 性能优化的手段有哪些？

可以从几个层面回答：

1. 代码层：组件拆分、`memo`、`useMemo`、`useCallback`
2. 数据层：不可变更新、状态最小化
3. 渲染层：虚拟列表、按需渲染、懒加载
4. 网络层：接口缓存、资源压缩、CDN
5. 构建层：代码分割、路由懒加载

面试时最好按“渲染优化 + 资源优化 + 交互优化”来组织答案。

## 六、工程实践与服务端渲染

### 说说你在 React 项目中是如何捕获错误的？

错误捕获通常分几类：

1. 组件渲染错误：Error Boundary
2. 异步请求错误：统一请求层处理
3. 全局运行时错误：`window.onerror`、`unhandledrejection`
4. 路由级兜底页和日志上报

React 中 Error Boundary 主要用于捕获子组件树中的渲染阶段异常，但不能覆盖所有异步错误。

### 什么是 Error Boundary？它能捕获哪些错误？

Error Boundary 即错误边界，是 React 用来捕获子组件树渲染错误的一种机制。

类组件里通常通过这些能力实现：

1. `static getDerivedStateFromError`
2. `componentDidCatch`

它适合处理：

1. 渲染阶段异常。
2. 生命周期中的异常。
3. 子组件树中的同步错误。

但它不能覆盖所有错误，比如：

1. 事件处理函数中的错误。
2. 异步代码里的错误。
3. 服务端渲染错误。

所以 Error Boundary 是重要兜底能力，但不是完整错误治理方案。

### 说说 React 服务端渲染怎么做？原理是什么？

React SSR 指在服务端先把组件渲染成 HTML，再返回给浏览器。

核心价值：

1. 提升首屏加载体验
2. 改善 SEO

基本流程：

1. 服务端根据路由和数据生成 HTML。
2. 浏览器收到后先展示首屏内容。
3. 客户端再接管并完成 hydration。

实际方案常见有 Next.js，或者基于 React 官方服务端渲染 API 自建。

### 说说你在使用 React 过程中遇到的常见问题？如何解决？

这类题考的是实战经验，回答时不要泛泛而谈，可以从几个高频问题说：

1. 状态提升过度导致重复渲染。
2. 列表 `key` 使用不当导致状态错乱。
3. Hooks 依赖数组写错导致闭包问题。
4. 异步请求竞态导致页面显示旧数据。
5. 组件卸载后仍更新状态导致告警。
6. Context 滥用导致大范围重渲染。

面试时最好用“问题现象 + 原因 + 解决办法”的结构回答，会更像真实项目经验。

## 七、React 各版本演进与新特性

### React 16 之前是什么架构？为什么要推倒重写？

React 0.x ~ 15 时代用的是 **Stack Reconciler（栈调和器）**，从 2013 年首次开源一直用到 2017 年。理解它能解释清楚 16 之后所有变化的动因。

**1. 核心模型：递归 + 同步**

Stack Reconciler 的工作过程是一次**深度优先递归**：

```
processComponent(root)
  └─ mountComponent(root)
       └─ render() → children
            └─ processComponent(child)
                 └─ mountComponent(child)
                      └─ ...一直递归到叶子
```

整个过程跑在 JS 调用栈上，**一旦开始就停不下来**，直到整棵树 diff 完、commit 到 DOM，控制权才还给浏览器。命名上的"Stack"指的就是它依赖原生函数调用栈做遍历。

**2. 为什么这种架构跑不动了？**

随着应用规模变大，问题集中爆发：

1. **长任务阻塞主线程**：组件树一深，递归一次轻松超过 50ms，浏览器在这期间没法响应输入、没法绘制动画——表现就是输入延迟、滚动卡顿、动画掉帧。
2. **没有优先级概念**：用户输入和后台数据更新被同等对待，高优交互排在低优更新后面。
3. **不可中断**：哪怕用户在输入，已经开始的渲染也得跑完，没法"先让一让"。
4. **SSR 是字符串拼接**：`renderToString` 同步把整棵树拼成 HTML，大页面阻塞 Node.js 主线程，没法流式输出。
5. **生命周期天然不安全**：`componentWillMount` / `componentWillReceiveProps` / `componentWillUpdate` 在同步模型下还能用，一旦改成可中断，这些钩子可能被调用多次，副作用会重复执行——这就是 16.3 给它们打 `UNSAFE_` 前缀的原因。

**3. 为什么不能"小步优化"，必须重写？**

栈递归是 V8 的调用栈，**JS 层没法暂停一个正在执行的函数**。要做"渲染到一半让出主线程"这件事，只能把递归改成迭代：把每个组件抽象成一个**可保存的工作单元**，自己维护遍历指针，需要让出时记录位置、下次从这个位置接着跑。

这就是 Fiber 架构的来历——它的本质是**用链表 + 自己实现的调度循环替换原生调用栈**，让"可中断、可恢复、可优先级调度"成为可能。

**4. 留下的历史包袱**

虽然 16 重写了底层，但 React 选择**对外 API 尽量兼容**——这也是为什么直到 17、18 还在持续清理：

1. 老生命周期保留多年，到 17 才彻底移除三个 `UNSAFE_` 钩子。
2. 老的事件系统（合成事件挂在 `document`）保留到 17 才下沉到根容器。
3. 老 SSR `renderToString` 至今仍可用，新 `renderToPipeableStream` 在 18 才登场。

理解 16 之前的架构，就能看懂后续所有重写的动机：**Fiber → 异步可中断 → 并发模式 → Suspense → Server Components**，每一步都在偿还 Stack Reconciler 留下的技术债。

### React 16 带来了哪些重大变化？

React 16（2017 年发布，代号 Fiber）是 React 历史上最大的一次重写。

**核心变化：**

1. **Fiber 架构**：底层调度模型完全重写，把不可中断的递归更新拆成可中断、可恢复的链表式任务单元。
2. **Error Boundaries**：新增 `componentDidCatch` 和 `static getDerivedStateFromError`，允许组件捕获子树渲染错误。
3. **Fragments**：`<React.Fragment>` 或 `<>...</>` 语法，render 不再必须返回单一根节点。
4. **Portals**：`ReactDOM.createPortal` 支持把子节点渲染到任意 DOM 节点，常用于弹窗、Tooltip。
5. **return 数组和字符串**：render 可以返回数组、字符串、null，不再强制包裹一层 div。
6. **新的服务端渲染**：`renderToNodeStream` 支持流式 SSR，性能大幅提升。
7. **自定义 DOM 属性**：未知 HTML 属性不再被丢弃，会原样输出。
8. **更小的体积**：核心包从 161KB 降到约 109KB。

### React 16.3 新增了哪些 API？为什么要引入新的生命周期？

16.3 是非常关键的一个小版本，主要为 Fiber 异步渲染做铺垫。

**新增 API：**

1. **新 Context API**：`React.createContext`，正式取代旧的 legacy context。
2. **`React.forwardRef`**：让函数组件也能转发 ref。
3. **`createRef`**：替代字符串 ref。
4. **StrictMode**：开发环境严格模式，提前暴露问题。

**新增生命周期：**

1. `static getDerivedStateFromProps(props, state)`：替代部分 `componentWillReceiveProps` 场景。
2. `getSnapshotBeforeUpdate(prevProps, prevState)`：在 DOM 更新前读取信息（如滚动位置），结果会传给 `componentDidUpdate`。

**标记为不安全的旧生命周期（加 UNSAFE_ 前缀）：**

1. `UNSAFE_componentWillMount`
2. `UNSAFE_componentWillReceiveProps`
3. `UNSAFE_componentWillUpdate`

**为什么废弃这些生命周期？**

因为 Fiber 的异步可中断渲染下，这三个生命周期可能被多次调用，使用 `setState` 或副作用容易引发难以排查的 bug。新 API 都是 `static` 或纯查询型，避免在协调阶段执行副作用。

### React 16.6 引入了什么？

1. **`React.memo`**：函数组件版本的 `PureComponent`，对 props 做浅比较。
2. **`React.lazy` + `Suspense`**：原生支持组件级代码分割。
3. **`static contextType`**：类组件消费 Context 的更优雅方式。
4. **`getDerivedStateFromError`**：让 Error Boundary 在渲染阶段就能更新降级 UI。

### React 16.8 的 Hooks 是怎么回事？解决了哪些问题？

16.8（2019 年 2 月）正式发布 Hooks，是函数组件能力的一次质变。

**Hooks 解决的核心问题：**

1. **逻辑复用难**：HOC 和 render props 都有嵌套地狱、命名冲突等问题。
2. **生命周期逻辑分散**：相关代码被强行拆到不同生命周期方法里。
3. **类组件心智负担重**：`this`、`bind`、绑定事件处理函数等问题。

**16.8 内置的 Hooks：**

1. `useState`：声明状态。
2. `useEffect`：处理副作用，类似 `componentDidMount + componentDidUpdate + componentWillUnmount`。
3. `useContext`：消费 Context。
4. `useReducer`：复杂状态用 reducer 模式管理。
5. `useCallback`：缓存函数引用。
6. `useMemo`：缓存计算结果。
7. `useRef`：跨渲染保持可变值，或获取 DOM 引用。
8. `useImperativeHandle`：自定义暴露给父组件的 ref 实例值。
9. `useLayoutEffect`：和 `useEffect` 相似，但在 DOM 更新后同步执行。
10. `useDebugValue`：在 React DevTools 中显示自定义 Hook 的标签。

### React 17 有什么变化？为什么被叫作"无新特性"版本？

React 17（2020 年 10 月）几乎没有面向开发者的新功能，但底层做了重要调整，是为后续版本铺路。

**关键变化：**

1. **事件委托从 `document` 移到 root 容器**：早期所有事件统一委托到 `document`，导致同页面多个 React 版本共存时事件冲突。17 之后委托到 `ReactDOM.render` 的 root 节点，支持渐进升级。
2. **新的 JSX Transform**：不再需要在每个文件 `import React`，由编译器自动注入运行时函数 `_jsx`。
3. **副作用清理时机调整**：`useEffect` 的 cleanup 现在是异步执行，避免阻塞屏幕更新。
4. **去掉事件池**：合成事件对象不再被复用，可以异步访问 `event.target` 等属性。
5. **`componentDidMount` 调用时序更严格**：父子组件 mount 顺序、effect 执行顺序更可预测。

一句话总结：React 17 是一座桥，让你以后能多版本并存、平滑升级。

### React 18 引入了哪些重磅特性？

React 18（2022 年 3 月）正式开启并发时代。

**核心特性：**

1. **自动批处理（Automatic Batching）**：18 之前只有 React 事件回调里的多次 `setState` 会合并；18 之后 Promise、setTimeout、原生事件回调里也会自动批处理。
2. **并发渲染（Concurrent Rendering）**：渲染过程可以中断、暂停、丢弃、重启，主线程不会被长时间阻塞。
3. **Transitions**：用 `startTransition` 标记非紧急更新，让紧急更新（如输入框）优先响应。
4. **Suspense 改进**：服务端流式渲染支持 Suspense，可以延迟某部分内容的 hydration。
5. **新的 `createRoot` API**：替代 `ReactDOM.render`，是开启并发模式的入口。
6. **严格模式更严格**：开发环境下 effect 会执行两次（mount → unmount → mount），用来暴露不健壮的清理逻辑。

**18 新增的 Hooks：**

1. **`useId`**：在 SSR 和客户端之间生成稳定唯一 ID。
2. **`useDeferredValue`**：延迟某个值的更新，用于把昂贵渲染推迟。
3. **`useTransition`**：返回 `[isPending, startTransition]`，标记低优先级更新。
4. **`useSyncExternalStore`**：让外部 store（如 Redux、Zustand）安全接入并发模式。
5. **`useInsertionEffect`**：CSS-in-JS 库专用，在 DOM 变更前同步插入样式。

**入口写法变化：**

React 18 之后启动应用必须用新的 `createRoot` API，旧的 `ReactDOM.render` 仍可用但会输出"已废弃"警告，且不会启用任何并发能力。换言之：升级到 React 18 但只改包版本不改入口，等于没真正升级。

```js
// React 17
import ReactDOM from 'react-dom'
ReactDOM.render(<App />, document.getElementById('root'))

// React 18
import { createRoot } from 'react-dom/client'
createRoot(document.getElementById('root')).render(<App />)
```

注意 `createRoot` 来自 `react-dom/client`（不是 `react-dom`），返回一个 root 对象，后续可以多次调用 `root.render(...)` 更新，也可以 `root.unmount()` 卸载。这种"先创建 root 再 render"的模式，是为了让 React 内部能持有更多渲染元数据（优先级、并发模式、错误处理回调等）。

### React 19 有哪些新特性？

React 19（2024 年 12 月正式发布）的关键词是"Actions、Server Components、Asset Loading"。

**核心新特性：**

1. **Actions**：表单 `action` 属性可以直接接收异步函数，自动处理 pending、error、optimistic 状态。
2. **Server Components 稳定**：服务端组件正式可用，能直接在组件里访问数据库、文件系统，无需写 API。
3. **`use` API**：可以在渲染期间读取 Promise 或 Context，配合 Suspense 实现更自然的数据获取。
4. **ref 作为 prop 传递**：函数组件不再需要 `forwardRef` 包裹，`ref` 可以直接作为 props 传入。
5. **ref 回调清理函数**：`ref` 回调可以返回清理函数，类似 `useEffect`。
6. **文档元数据支持**：组件里直接写 `<title>`、`<meta>`、`<link>`，React 自动 hoist 到 `<head>`。
7. **资源加载优化**：内置 `preload`、`preinit`、`prefetchDNS`、`preconnect` API。
8. **改进的错误处理**：`onUncaughtError`、`onCaughtError` 选项，错误信息更精简。
9. **Context 简化**：`<Context>` 可以直接当 Provider 用，不用再写 `<Context.Provider>`。

**19 新增的 Hooks：**

1. **`useActionState`**：管理表单 action 的状态（旧名 `useFormState`）。
2. **`useFormStatus`**：在表单内部子组件中获取 pending 状态。
3. **`useOptimistic`**：实现乐观更新，UI 先按预期变化，失败再回滚。
4. **`use`**：严格说不是 Hook，可以在条件分支中读取 Promise / Context。

**Actions 示例：**

```jsx
function UpdateName() {
  const [error, submitAction, isPending] = useActionState(
    async (prev, formData) => {
      const error = await updateName(formData.get('name'))
      if (error) return error
      redirect('/profile')
      return null
    },
    null,
  )
  return (
    <form action={submitAction}>
      <input name="name" />
      <button disabled={isPending}>Update</button>
      {error && <p>{error}</p>}
    </form>
  )
}
```

这个例子展示了 React 19 把"表单异步提交"完全收口的能力，几个细节：

1. **`useActionState`** 接收一个 reducer 风格的异步函数 `(prevState, formData) => nextState`，返回 `[当前状态, 包装过的 action, isPending]`。它会自动管理 pending 标志位、错误状态、表单值，省掉过去要用 `useState` + `useTransition` + try/catch 手写的几十行样板。
2. **`<form action={submitAction}>`**：React 19 让原生 form action 属性可以接收异步函数，提交时浏览器原生表单语义会被劫持，自动调用 `submitAction`，并把 `<form>` 内部所有 `name="xxx"` 的字段打包成 `formData` 传入。
3. **`isPending` 自动管理**：异步函数执行期间 `isPending` 为 true，结束后回到 false，UI 上的 disabled 按钮和 loading 态可以直接绑定，无需自己维护。
4. **错误就是返回值**：reducer 风格的好处是错误也是 state 的一部分——返回非空 error 表示失败，返回 null 表示成功；这样错误展示和状态流向完全统一。
5. **配合 Server Actions 一致**：在服务端 React 项目里，`updateName` 可以直接是一个 server action 函数，前端调用看起来跟本地函数一模一样，由编译器自动转 RPC。

### React 类组件生命周期完整演进图？

可以分两个阶段来记。

**16.3 之前的旧生命周期：**

```
挂载：
  constructor
  componentWillMount       ← 已废弃
  render
  componentDidMount

更新：
  componentWillReceiveProps  ← 已废弃
  shouldComponentUpdate
  componentWillUpdate         ← 已废弃
  render
  componentDidUpdate

卸载：
  componentWillUnmount
```

**16.3 及之后的新生命周期：**

```
挂载：
  constructor
  static getDerivedStateFromProps   ← 新增
  render
  componentDidMount

更新：
  static getDerivedStateFromProps   ← 新增
  shouldComponentUpdate
  render
  getSnapshotBeforeUpdate           ← 新增
  componentDidUpdate

卸载：
  componentWillUnmount

错误处理：
  static getDerivedStateFromError   ← 16.6 新增
  componentDidCatch                  ← 16 新增
```

新旧对比的核心思路是：**把"可能产生副作用、可能多次执行"的生命周期换成"纯函数 / 在 DOM 已就绪后执行"的生命周期**。

### 函数组件如何对应类组件的生命周期？

函数组件没有"生命周期"概念，而是通过 Hooks 表达副作用时机。

| 类组件生命周期 | 函数组件等价写法 |
| --- | --- |
| `constructor` 初始化 state | `useState` 初始值 |
| `componentDidMount` | `useEffect(fn, [])` |
| `componentDidUpdate` | `useEffect(fn, [deps])` |
| `componentWillUnmount` | `useEffect` return 的 cleanup |
| `shouldComponentUpdate` | `React.memo` + 自定义比较函数 |
| `getDerivedStateFromProps` | 渲染期间直接根据 props 计算（或 `useMemo`） |
| `getSnapshotBeforeUpdate` | `useLayoutEffect` |
| `componentDidCatch` | 仍需类组件 Error Boundary（19 之前） |

注意：`useEffect` 是异步执行（在浏览器绘制之后），`useLayoutEffect` 是同步执行（在 DOM 更新后、绘制前），这是它们和生命周期的关键时序差异。

### React 18 自动批处理具体怎么生效？

简单理解：18 之前只在合成事件回调里批处理，18 之后任何位置都批处理。

```jsx
// React 17 行为
function handleClick() {
  setCount(c => c + 1)   // 不立即重渲
  setFlag(f => !f)       // 不立即重渲
  // 一次重渲 ✓
}

setTimeout(() => {
  setCount(c => c + 1)   // 立即重渲一次
  setFlag(f => !f)       // 又立即重渲一次
  // 两次重渲 ✗
}, 0)

// React 18 行为
// 上面两段都只触发一次重渲
```

为什么 17 区别对待？因为 17 的批处理依赖一个全局变量 `isInsideEventHandler`——只有 React 自己的事件系统在执行回调前会把这个标志位置 true，结束时清回 false。setTimeout、Promise.then、原生事件监听都不在这个标志位的范围内，setState 看到 `false` 就直接同步走完整个更新流程。

18 通过新的并发渲染调度器，把"是否批处理"从"是否在 React 事件回调里"改成了"按更新优先级调度"，所以无论从哪个上下文调用 setState，都会进入同一个调度队列里被合并。

如果想退出批处理立即更新，可以用 `flushSync`：

```js
import { flushSync } from 'react-dom'
flushSync(() => setCount(c => c + 1))   // 这里就会立即提交
setFlag(f => !f)                         // 进入下一次批处理
```

`flushSync` 的常见使用场景是"必须先把 DOM 更新到某个状态，再执行后续逻辑"——典型例子是 `flushSync` 后立刻调用 `scrollTo`、读取 DOM 尺寸、调用第三方库 API。要尽量少用，因为它会破坏并发渲染的优化。

### React 18 的 Transition 解决了什么问题？

主要解决"高优先级更新被低优先级更新阻塞"的问题。典型例子是搜索框：用户输入需要立刻响应，但搜索结果列表渲染可能很重。

```jsx
function Search() {
  const [query, setQuery] = useState('')
  const [list, setList] = useState([])
  const [isPending, startTransition] = useTransition()

  function onChange(e) {
    setQuery(e.target.value)              // 紧急更新：输入框立刻刷新
    startTransition(() => {
      setList(filter(e.target.value))     // 低优先级：列表可被打断
    })
  }

  return (
    <>
      <input value={query} onChange={onChange} />
      {isPending ? <Spinner /> : <List items={list} />}
    </>
  )
}
```

代码读法：

1. **两个 state 分别承担不同角色**：`query` 是输入框的受控值，必须立即响应；`list` 是基于 query 过滤后的搜索结果，渲染可能耗时。
2. **`setQuery` 在 `startTransition` 之外**——它是高优先级更新，会在下一个 tick 之内立刻反映到屏幕，保证输入流畅。
3. **`setList` 在 `startTransition` 内**——React 把这次更新标记为"transition"，调度优先级降低；如果在它还没渲染完时用户又输了一个字符，新的 transition 会"取消"旧的，避免把性能浪费在已经过时的中间状态上。
4. **`isPending`** 在 transition 进行中为 true，可以用来展示 Spinner 或者保留旧 UI，明确告诉用户"新结果还在算"。

底层原理：被 `startTransition` 包裹的更新走低优先级队列，可以被新的高优先级更新打断和重新计算。

### React 19 的 Server Components 和 SSR 有什么区别？

很多人会混淆两者，但它们解决的是不同问题。

| 维度 | SSR | Server Components |
| --- | --- | --- |
| 执行时机 | 每次请求时执行 | 构建时或请求时 |
| 输出 | HTML 字符串 | RSC 序列化协议 |
| 是否 hydrate | 是 | 否（Server Component 永不在客户端运行） |
| 能否带交互 | 完整组件 | 只能渲染，不能用 useState、事件 |
| 包体积影响 | 计入客户端 bundle | 不计入客户端 bundle |
| 直接访问后端资源 | 可以但要小心 | 设计上鼓励 |

简单说：SSR 是"服务端预渲染 HTML"，Server Components 是"组件本身就在服务端运行，从不发到客户端"。两者可以组合使用，Next.js App Router 就是典型实现。

### React 16-19 整体演进的主线是什么？

可以用一句话概括：**从同步递归渲染，逐步演进到可中断、可调度、可流式、可服务端化的并发架构**。

具体里程碑：

1. **15 及以前**：Stack Reconciler，递归 + 同步，无法中断、无优先级——后续所有改造的起点。
2. **16**：底层换成 Fiber（链表结构 + 可中断），但还是同步消费。
3. **16.3**：为异步渲染清理生命周期。
4. **16.6 / 16.8**：上层 API 升级（memo、lazy、Hooks），开发体验跟上。
5. **17**：基础设施改造（事件、JSX、effect 时机），为并发铺路。
6. **18**：并发渲染正式可用，自动批处理、Transitions、新 SSR。
7. **19**：把"客户端 + 服务端"统一起来（RSC、Actions），开发心智回归"写组件就是写 UI"。

理解这条主线，就能解释为什么 React 不断废弃旧 API、为什么生命周期改革、为什么强调不可变更新、为什么推 Suspense——它们都在为同一个方向服务。

## 附：常见代码实现

### 1. Hooks 版计数器

最基础的函数组件 + `useState` 写法。`useState(0)` 返回一个二元组：当前值和 setter，setter 调用后会触发组件重新渲染。点击按钮时调用 `setCount(count + 1)`，React 把新值排进更新队列，下次 render 时 `count` 就是 `1`。

```jsx
import { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>count: {count}</button>
}
```

需要留意一个常见陷阱：如果在同一个 tick 里连续 `setCount(count + 1)` 调用多次，由于 `count` 是闭包里的旧值，多次调用拿到的都是同一个值，最终只 +1。要连续叠加，应该用函数式写法 `setCount(c => c + 1)`，让 React 把更新视为基于"上一次结果"的连续运算。

### 2. `React.memo` 示例

`React.memo` 是函数组件版本的浅比较优化，只有当 props 变化（默认按 `Object.is` 逐字段比较）时才重新渲染，否则复用上次的渲染结果。

```jsx
const Child = React.memo(function Child({ value }) {
  console.log('render child')
  return <div>{value}</div>
})
```

使用要点：

1. **只对 props 浅比较生效**：如果父组件每次都传新对象/新函数（如内联 `() => ...`），memo 形同虚设。配合 `useMemo`/`useCallback` 才能真正发挥效果。
2. **第二个参数可以自定义比较函数**：`React.memo(Comp, (prev, next) => ...)`，返回 true 表示视为相等不重渲。
3. **不是越多越好**：memo 自身有比较成本，组件本身轻量时反而是负优化。优化前先确认确实有"父组件频繁重渲、子组件渲染昂贵且 props 大多不变"的场景。

### 3. Error Boundary 示例

React 类组件版的错误边界：当子组件树渲染出现异常时，捕获错误并展示降级 UI，避免整棵组件树白屏崩溃。

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false }
  static getDerivedStateFromError() {
    return { hasError: true }
  }
  componentDidCatch(error, info) {
    console.error(error, info)
  }
  render() {
    return this.state.hasError ? <div>页面异常</div> : this.props.children
  }
}
```

两个回调各司其职：

1. **`getDerivedStateFromError`**：在渲染阶段调用，返回的对象会被 merge 到 state。它是纯函数（不能在里面发副作用），唯一的目的是把"出错了"这个事实落到 state 上，从而下次 render 走降级分支。
2. **`componentDidCatch`**：在 commit 阶段调用，可以执行副作用（上报、打日志），第二个参数 `info.componentStack` 包含组件调用栈，对线上排错非常重要。

能捕获：渲染阶段、生命周期、子组件树里的同步错误。
不能捕获：事件回调里的错误、`setTimeout`/Promise 等异步错误、SSR 错误、Error Boundary 自身错误——这些需要 `window.onerror`、`unhandledrejection`、try/catch 等其他手段兜底。
