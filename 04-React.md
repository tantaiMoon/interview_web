[toc]



## 一、React 基础认知

### 说说你对 React 的理解？有哪些特性？

React 是构建 UI 的库（不是框架），思路就三点：

- **组件化**：UI 拆成函数或类，每个组件管自己的状态和渲染，可组合可复用
- **声明式**：写"UI 应该长什么样"，不写"怎么改 DOM"，状态变了 React 自己算 diff
- **单向数据流**：父往子传 props，子要改状态就触发回调让父改，数据走向永远清晰

跟 Vue 最核心的区别：Vue 用模板，编译期能识别静态/动态部分做大量优化；React 用 JSX 是纯 JavaScript 表达式，靠运行时调度（Fiber + 调和算法），灵活性高但运行时开销也大。所以 React 18 引入 Concurrent Rendering、Server Components 这些都是在补运行时性能。

特性：

- **Hooks**：函数组件复用逻辑的标准方案，把状态和副作用从类组件里解放出来
- **Fiber 架构**：可中断、可恢复的协调过程，让 React 能做时间切片
- **Concurrent 模式**：`useTransition` / `useDeferredValue` / `Suspense` 把重活儿延后做，不阻塞高优先级更新
- **跨平台**：React Native / React Three Fiber / ReactDOM/server，渲染层抽象
- **庞大生态**：React Router、Redux / Zustand / Jotai、Next.js / Remix、TanStack Query 等

### 说说 Real DOM 和 Virtual DOM 的区别？优缺点是什么？

真实 DOM 是浏览器维护的页面节点对象，每个节点有上百个属性、几十个方法，创建和修改成本都不低。虚拟 DOM 是 React 用 JS 对象描述出的目标 DOM 结构——`{ type: 'div', props: {...}, children: [...] }`，几乎纯数据，操作便宜。

为什么需要这层中间表示：

1. **批量更新**：状态多次变化先在内存里收敛，最后一次 diff、一次 patch，避免反复触发 reflow
2. **跨平台**：同一棵 vnode 树可以走 ReactDOM（浏览器）、ReactNative（移动端原生组件）、ReactDOMServer（HTML 字符串）、Ink（终端）、React Three Fiber（Three.js）
3. **声明式开发**：开发者只写"想要什么样"，diff 算法负责"怎么改"

但要清楚一点：**虚拟 DOM 不是"比直接操作 DOM 快"**。手写 `el.textContent = 'x'` 比 diff 一遍再 patch 显然更快。虚拟 DOM 的真实价值是可维护性和工程化，不是单点性能。

### 说说 React JSX 转换成真实 DOM 的过程。

JSX 是 ECMAScript 提案，浏览器不认识，必须靠 Babel / SWC 编译。

```jsx
// 你写的
<div className="box" onClick={handler}>Hello</div>

// 编译后（老 transform）
React.createElement('div', { className: 'box', onClick: handler }, 'Hello')

// 编译后（新 JSX runtime，React 17+）
import { jsx as _jsx } from 'react/jsx-runtime'
_jsx('div', { className: 'box', onClick: handler, children: 'Hello' })
```

从代码到屏幕的链路：

1. **编译期**：Babel 把 JSX 转成 `_jsx()` 调用
2. **运行期 - 创建 element**：`_jsx()` 返回 `React.Element` 对象（type + props + key 等纯描述）
3. **协调（Reconciler）**：React 把 element 转成 Fiber 节点，构建工作树（work-in-progress），跟当前树 diff，标记需要做的副作用（插入/更新/删除）。这一步是**可中断**的——优先级高的任务来了能把这次 diff 停下让路。
4. **提交（Commit）**：把所有副作用一次性应用到真实 DOM，这一步**不可中断**（不然会看到半成品 UI）
5. **生命周期 / Effect**：DOM 更新后跑 `componentDidMount/Update`、`useLayoutEffect`（同步）和 `useEffect`（异步）

注意第 3 步——Fiber 之前（React 15 的 Stack Reconciler）是递归调用栈，一旦开始就停不下来；Fiber 把这棵树拆成可中断的链表节点，每个节点处理完都能检查时间片，必要时把控制权让出去，等下一帧再继续。这是 React 17/18 Concurrent 一切能力的基础。

## 二、组件、状态与通信

### 说说 React 生命周期有哪些不同阶段？每个阶段对应的方法是什么？

如果从类组件角度回答，生命周期大致分为三个阶段：

1. 挂载阶段：`constructor`、`getDerivedStateFromProps`、`render`、`componentDidMount`
2. 更新阶段：`getDerivedStateFromProps`、`shouldComponentUpdate`、`render`、`getSnapshotBeforeUpdate`、`componentDidUpdate`
3. 卸载阶段：`componentWillUnmount`
4. 错误处理：`getDerivedStateFromError`、`componentDidCatch`

历史上还有一些旧生命周期，如 `componentWillMount`、`componentWillReceiveProps`、`componentWillUpdate`，从 React 16.3 起被标记为不安全（加 `UNSAFE_` 前缀），原因是它们在 Fiber 异步可中断渲染下可能被多次调用，导致副作用重复执行。具体演进可以参考"七、React 各版本演进与新特性"中的生命周期演进图。

函数组件里的这类逻辑，一般会对应到 Hooks，比如 `useEffect`、`useLayoutEffect`。

### state 和 props 有什么区别？

两者都是 React 组件中的核心概念，但职责不同。

1. `props` 是外部传入的，组件内部一般不直接修改。
2. `state` 是组件内部维护的，用来描述组件自身状态。
3. `props` 更像输入，`state` 更像组件内部数据源。

直接区分的话，`props` 是父组件传进来的，`state` 是组件自己维护的。

### super() 和 super(props) 有什么区别？

这是类组件中的问题。

1. `super()` 用于调用父类构造函数。
2. `super(props)` 则会把 `props` 传给父类构造函数。

在 React 类组件中，如果你在 `constructor` 里需要通过 `this.props` 访问传入属性，通常要写 `super(props)`。

在现代 React 中，这种问题出现得少了，因为函数组件已经成为主流。

### 说说 React 中的 setState 执行机制。

`setState` 不是"立刻改 this.state 再触发 render"，而是把更新塞进调度队列，由 React 决定什么时候批量处理。

**关键事实**：

1. **批处理（Batching）**：同一事件回调里多次 setState，React 会合并成一次更新、一次 render。
   - React 17 之前：只有 React 自己接管的合成事件里才批处理；`setTimeout`、`fetch.then`、`Promise.then` 这些"逃出 React"的回调里调 setState 会**一次一次单独触发** render。
   - React 18 起：所有更新都自动批处理（Automatic Batching），不管你在哪里调，统一合并。需要立刻刷新的极特殊场景可以用 `flushSync` 强制同步。
2. **异步表现 ≠ 真的异步**：`setState` 是个标记，调用后立刻返回，下一行读 `this.state` 拿到的还是旧值。但更新动作本身是同步排入队列的。
3. **新状态依赖旧状态时必须用函数形式**：

   ```js
   // ❌ 三次调用拿到的 count 是同一个，最终 +1
   setState({ count: count + 1 })
   setState({ count: count + 1 })
   setState({ count: count + 1 })

   // ✅ 函数式：每次回调里的 prev 是队列里的"最新中间值"
   setState(prev => ({ count: prev.count + 1 }))
   setState(prev => ({ count: prev.count + 1 }))
   setState(prev => ({ count: prev.count + 1 }))   // 最终 +3
   ```

4. **setState 完成的回调用第二参数**：`setState({...}, () => { /* state 已更新、DOM 已渲染 */ })`。函数组件用 `useEffect` 监听 state 变化达到等效。

函数组件里 `useState` 的 setter 行为一样，但没有"setState 第二参"，要等更新完做事就用 `useEffect`。

### React 事件机制是什么？

React 自己造了一套**合成事件系统**（SyntheticEvent），不直接用原生 DOM 事件。

工作方式：

- **事件委托**：React 17 之前所有事件统一委托到 `document`，17+ 改成委托到 React 根容器（避免多个 React 应用同时挂到一个页面时事件冲突）
- 真实 DOM 触发原生事件 → React 监听器拦截 → 找到事件源对应的 Fiber → 沿着虚拟 DOM 树合成捕获+冒泡过程 → 触发组件里写的 `onClick` 等回调
- 传给回调的是 React 包装的 `SyntheticEvent`，跟原生 Event 接口几乎一样，但跨浏览器统一。React 17 之前还做了事件池复用（事件对象会在回调结束后被重置，所以异步访问 `e.target` 要先 `e.persist()`）；17+ 已经移除事件池

合成事件的实际意义：

- **统一浏览器差异**：再也不用关心 IE vs 标准的事件接口区别
- **性能**：把成百上千个 `addEventListener` 合并成根容器上的一个，内存开销低
- **可控的事件流**：React 能跟自己的渲染调度配合，比如 React 18 把事件分成 discrete（click、input，高优）和 continuous（mousemove、scroll，低优），驱动 Concurrent 调度

几个常踩的问题：

- 合成事件的 `e.stopPropagation()` 只能挡住 React 自己的事件流，挡不住已经从原生层冒泡上来的事件。原生 `document.addEventListener` 的回调比合成事件先跑。
- React 17+ 由于委托位置变了，原来在 `document` 上注册的 `addEventListener` 跟 React 事件的执行顺序会变化，老代码迁移要注意。

早期 React 主要把事件委托到 `document`，后续版本对事件委托节点做过调整。

### React 事件绑定的方式有哪些？区别是什么？

常用写法有：

1. JSX 中直接传箭头函数。
2. JSX 中传已经绑定好的方法。
3. 在构造函数中手动 `bind`。
4. 类字段语法定义箭头函数。

主要差别：

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

开发里更稳妥的做法是：先按最直接的写法来，真的看到 profiler 里某个子组件因为函数引用变化频繁重渲了，再考虑 `useCallback`。React 19 之后编译器会进一步接管这类 memoization，这个点以后会越来越弱化。

### React 构建组件的方式有哪些？区别是什么？

主要有两种写法：

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

简单场景优先 `props` 和回调；跨层级共享或全局状态，再考虑 `Context` 和专门的状态管理方案。

**为什么有这个优先级？**

1. **`props` 透传越深越糟（"prop drilling"）**：父 → 子 → 孙 三层就开始繁琐，五层以上几乎不可维护——中间层组件被迫接收并不关心的 props，重构时改一个数据格式要动好几个文件。
2. **`Context` 解决了透传，但有重渲代价**：任何 Context value 的引用变化，都会让所有消费该 Context 的组件重新渲染——即使它们只用了 value 中的一小部分。这是 Context 不适合作为"高频写入的全局状态"的根本原因。
3. **状态管理库（Redux / Zustand / Jotai）是为高频更新设计的**：它们都内置了"selector + 浅比较"机制，组件只在自己关心的切片变化时重渲，性能上远胜 Context。

选型可以按下面方式判断：

1. 父子直接通信，优先 `props + 回调`
2. 跨 2~3 层、低频变化，比如主题、用户信息、i18n，可以用 `Context`
3. 跨多层、高频更新、多处订阅，更适合 `Zustand / Redux / Jotai`
4. 服务端数据缓存，优先 `React Query / SWR`

一个常见误用：把整个用户数据放到 Context 里然后每次 setState 都更新整个 user 对象——所有 inject 该 Context 的组件都会重渲。正确做法是要么拆 Context（分别 ProvideUserId 和 ProvideUserDetail），要么直接用 Zustand。

### React 中的 key 有什么作用？

`key` 是列表渲染中用于标识节点身份的唯一标记。

作用：

1. 帮助 React 在 diff 时识别节点是否可复用。
2. 提高列表更新效率。
3. 避免错误复用组件状态。

开发时优先使用稳定唯一的 id，不建议在可变列表中直接使用数组下标。

**为什么数组下标会出问题？**

下标表示的是位置，不是身份。列表一旦插入、删除、排序，同一个下标对应的数据就变了。但 React 做 diff 时只会看到 key 没变，于是继续复用原来的节点。

问题往往不是文本错了这么简单，而是节点里的状态也会跟着错位。常见的例子就是列表项里有输入框或局部 state，头部插一项以后，后面所有节点都可能“内容换了，状态没换”。

拿列表头部插入新元素举例：

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

看起来只是插入了一项，实际上后面大部分节点都被错位复用了。输入框内容、组件局部 state、动画状态都会随之错位。

如果用 `key={item.id}`：

| key | 旧位置 | 新位置 | diff 判定 | 表现 |
| --- | --- | --- | --- | --- |
| id-A | 0 | 1 | 找到 → 移动 | A 的 DOM 整体移到位置 1，"hello" 随 A 走 ✓ |
| id-X | - | 0 | 不存在 → 创建 | 全新 X 节点，input 是空的 ✓ |

性能上也是同一个结论：稳定 id 更容易让 React 做出正确复用，而不是把一长串节点重新 patch 一遍。

下标不是绝对不能用，但前提很苛刻：列表稳定、不重排、没有内部状态。只要列表项里出现输入框、动画、子组件本地 state，就不该把下标当默认选项。

### 说说对 React refs 的理解？应用场景有哪些？

`ref` 用于直接访问 DOM 节点或子组件实例能力。

常见的用法有：

1. 输入框聚焦
2. 获取元素尺寸和位置
3. 调用子组件暴露的方法
4. 和第三方 DOM 库集成

在 React 里，`ref` 更像一个例外通道。大多数 UI 问题都应该先用 `props`、`state`、`effect` 解决；实在需要碰 DOM 或拿实例能力的时候，才动 `ref`。

实际碰到的情况主要有这几类：

1. 聚焦、选区、滚动定位
2. 读取尺寸和位置
3. 控制视频、音频这类原生对象
4. 接第三方库，比如图表、地图、编辑器
5. 保存不会触发重渲的可变值

之所以不能滥用，是因为它绕开了 React 正常的数据流。你一旦开始手动改 DOM，就要自己承担和 React 更新冲突的风险。

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

这里有个点要说透：

1. 这些优化依赖浅比较。
2. 如果数据更新不是不可变更新，优化可能失效。
3. 不是所有组件都值得做 memo，优化前应先确认瓶颈。

**1. 浅比较是什么、靠不靠谱？**

`PureComponent` 和 `React.memo` 内部都用 `Object.is` 对 props/state 的**第一层**字段做比较：

1. 基本类型（string、number、boolean）直接比值。
2. 对象、数组、函数只比引用（`===`），**不会递归看内容**。

这就带来两类典型问题：

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

落地时：

1. 简单结构手写展开（`{...obj}` / `[...arr]`）即可。
2. 嵌套深时用 **Immer 的 `produce`**：写起来像可变，背后产新对象，可读性和正确性兼得。
3. Redux Toolkit、Zustand 的 immer 中间件已经内置了 Immer，直接 `state.x.y = z` 也是不可变更新。

**3. memo 不是越多越好**

`React.memo` 自己也有成本：每次 render 都要遍历 props 做浅比较、还要缓存上一次结果。**对一个本身渲染就很轻的组件包 memo，比较的开销可能比直接重渲还高**，属于负优化。

判断要不要 memo 的实操路径：

1. **先用 React DevTools Profiler 录一段交互**，看哪些组件渲染时间长、渲染次数多。
2. 只对"**渲染开销大** + **父组件高频更新** + **props 大概率没变**"这类组件加 memo。常见对象：长列表 item、图表、复杂表单字段。
3. 加完后再 Profile 一次，确认确实减少了渲染时间，否则回滚——避免引入"看起来在优化"但没有收益的代码。

补充一点**前瞻**：React 19 的 **React Compiler** 会在编译期自动插入 memo / useMemo / useCallback，未来手动 memo 的场景会越来越少，但**理解上面这套原理依然是判断编译器优化是否生效、是否需要兜底的基础**。

### 说说对受控组件和非受控组件的理解？应用场景是什么？

受控组件指表单值由 React 状态控制，输入变化通过事件同步到状态。

非受控组件指表单值更多由 DOM 自己维护，通常通过 `ref` 获取值。

适合用在：

1. 受控组件适合需要实时校验、联动、统一状态管理的表单。
2. 非受控组件适合简单表单或与原生表单行为深度结合的场景。

React 项目里，大多数复杂表单更倾向使用受控组件。

**为什么复杂表单选受控？** 受控带来的能力是非受控做不到的：

1. **实时校验和联动**：手机号每输一个字符就校验长度、密码强度实时显示、省份联动城市——这些都要求"当前值随时可读"，受控组件天然可以 `useState` 拿到，非受控要监听 input 事件再 setState，反而更绕。
2. **格式化和限制**：自动给金额加千分位、自动转大写、限制只能输数字——这些是"输入时改变值"，必须由 state 控制；非受控的 `defaultValue` 不能干预后续输入。
3. **统一数据源便于提交**：复杂表单要序列化成 API 请求体，受控组件让所有字段都在 state 树里，提交时 `JSON.stringify(formData)` 即可；非受控要逐个 `ref.current.value` 收集，丢字段、忘 trim 是常见 bug。
4. **回显已有数据**：编辑场景需要把后端数据填回表单。受控组件 `setState(serverData)` 一行完成；非受控要在 mount 后 `inputRef.current.value = ...`，而且不能拦截 submit 验证。
5. **跨字段错误提示**：密码和确认密码联动校验、订单总价随商品数量变化——这些跨字段计算依赖"所有字段都在同一份 state 里"，受控天然支持。

**那非受控什么时候用？** 三个常见场景：

1. **简单一次性表单**：联系我们、留言板，提交即销毁，不需要中间态。
2. **大型表单的性能优化**：受控表单每输一个字符触发整个表单组件重渲，1000 个字段时会卡。这时用 `react-hook-form` 这种"非受控但帮你收集"的库，性能比纯受控好一个数量级。
3. **必须用原生表单行为时**：`<input type="file">`（受控不被允许）、`<input type="password">` 在某些密码管理器场景。

**实际推荐：**

1. 字段少（10 个以内）、要校验联动 → 直接 `useState` 受控。
2. 字段多、性能敏感、想要好 DX → 用 `react-hook-form` + zod，是非受控但 DX 接近受控。
3. 一次性提交、无校验 → 非受控 + ref 取值即可。

### 说说对高阶组件的理解？应用场景是什么？

高阶组件 HOC 就是接收一个组件，再返回一个增强后的组件。

常见用法：

1. 权限控制
2. 日志埋点
3. 通用状态注入
4. 逻辑复用

不过在现代 React 中，不少原先用 HOC 解决的问题，也会转向 Hooks 方案，因为逻辑来源更直观。

**HOC 和自定义 Hook 的本质差异：**

1. **嵌套地狱（Wrapper Hell）**：HOC 是层层包装组件，DevTools 里会看到 `withAuth(withTheme(withRouter(MyComp)))`，调试时 props 来源很难追溯。Hook 是平铺调用，不增加组件树层级。
2. **Props 命名冲突**：两个 HOC 都给组件注入 `data` prop 时，后一个会覆盖前一个，且没有任何编译警告。Hook 的返回值由调用方主动解构、命名，冲突不会发生。
3. **Props 来源不透明**：组件里看到 `this.props.user`，要回头去找哪个 HOC 注入的；Hook 调用 `const user = useUser()`，import 语句一目了然。
4. **静态方法/refs 转发繁琐**：HOC 默认不会自动保留被包装组件的静态方法和 refs，需要手动 hoist + forwardRef。Hook 不存在这个问题。
5. **TypeScript 友好度**：HOC 的类型签名（要保留泛型 + 注入新 props + 移除已注入 props）复杂，社区里的 HOC TS 类型几乎都有边界 case 出错；Hook 类型是普通函数，TS 完美推导。

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

使用 Hooks 时有几条规则要遵守：

1. 只能在函数组件或自定义 Hook 顶层调用，不能放在条件分支或循环里。
2. 自定义 Hook 必须以 `use` 开头。
3. 依赖数组要写完整，否则容易出现闭包陷阱。

**为什么 React 要立这三条规矩：**

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

**`exhaustive-deps` ESLint 规则**会自动检测漏掉的依赖，**建议开启**——它会警告"你这个 effect 用了 count 但没写到 deps 里"。React 19 的编译器进一步会自动 memoize，未来这条陷阱也会减少。

规则就是：**"读了什么响应式值，就要把它写进 deps"——除非你明确用函数式更新或 ref 跳出闭包。**

### 详解 React Hooks 的闭包陷阱（Stale Closure）

闭包陷阱（Stale Closure，也叫"过期闭包"）几乎是函数组件最高频的 bug 来源。要彻底理解它，需要从**函数组件的执行模型**说起。

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

这里需要明确：

1. **每次 render 都会创建一组全新的局部变量、函数、effect 回调**。
2. 这些函数通过**词法作用域**捕获了"那一次 render 时的 state 和 props"。
3. 一旦这个函数被"保存"到某个长寿命的位置（`setInterval`、订阅回调、`ref`、外部缓存…），它捕获的就是**那一帧的快照**，再也不会更新。

所以闭包陷阱不是 React 的 bug，而是 JS 闭包语义 + "每次 render 是新调用"叠加的必然结果。

#### 二、四种常见场景

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
3. **拆分 effect**——一个 effect 只负责一个职责，依赖数组就短，闭包就简单。
4. **长寿命订阅用 ref 兜底**——socket、定时器、全局事件监听等场景，用 ref 保存最新回调。
5. **升级到 React 19**——React Compiler 会自动 memoize，并且 `useEffectEvent` 提供了官方的"始终拿最新值"的能力，闭包陷阱出现频率会显著下降。

#### 五、小结

> 闭包陷阱来自"**函数组件每次 render 都是一次新调用，effect/回调里捕获的是那一帧的 state 快照**"。修复思路只有两条：要么**让 effect 在依赖变化时重新创建闭包**（写全 deps），要么**让回调在执行时主动读取最新值**（函数式更新、ref、useEffectEvent）。

## 三、样式、动画与生态能力

### 说说 React 中引入 CSS 的方式有哪几种？区别是什么？

常用方案有：

1. 普通 CSS 文件
2. CSS Modules
3. Sass/Less 等预处理器
4. CSS-in-JS
5. 原子化 CSS 方案

主要差别：

1. 样式作用域控制能力不同。
2. 工程组织方式不同。
3. 运行时开销和开发体验不同。

项目里常见组合是：普通全局样式 + CSS Modules 或原子化方案。

**为什么这样组合？** 各种方案各有不可替代的场景：

1. **必须有少量全局样式**：reset/normalize.css、全局字体、CSS 变量定义、深色模式根选择器——这些是"跨所有组件生效"的，写到 CSS Modules 里反而每个组件都要 import 一次。
2. **组件级隔离选 CSS Modules 或原子化**：业务组件的样式必须避免污染全局——CSS Modules 通过类名加 hash、原子化（Tailwind 等）通过预生成单一职责类，都能做到这点。
3. **运行时 CSS-in-JS 渐渐被冷落**：styled-components、emotion 这类运行时方案会带来包体积（约 12KB）和首次渲染开销（要解析 props 生成样式表），React Server Component 时代它们也水土不服——所以不少团队从 styled-components 迁回 CSS Modules + Tailwind。
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

它和普通全局 CSS 的主要区别是：普通 CSS 默认全局生效，而 CSS Modules 更强调组件级隔离。

### CSS-in-JS 在 React 中是怎么理解的？适合什么场景？

CSS-in-JS 指的是把样式逻辑写进 JavaScript 中，再通过运行时或编译时方案生成样式。

常见价值：

1. 更方便做动态样式。
2. 样式与组件逻辑更靠近。
3. 可以天然做局部作用域隔离。

适合用在：

1. 主题系统。
2. 动态样式较多的组件。
3. 组件库封装。

但也要说明：CSS-in-JS 会带来一定运行时或构建复杂度，不一定适合所有项目。

**具体的代价是什么？**

1. **运行时方案的开销**（styled-components、emotion）：每次组件 render，库都要根据 props 计算 className、把 CSS 字符串注入 `<style>` 标签、维护样式去重缓存。包体积约 12~15KB（min+gzip），SSR 时要做样式抽取，首屏 hydration 也要重建样式表。
2. **TypeScript 友好度参差**：styled-components 的 `styled.div<{primary: boolean}>` 写起来繁琐，泛型类型报错信息晦涩；emotion 的 `css` prop 在 JSX 类型推导上要手动配置 pragma。
3. **DevTools 调试不直观**：浏览器看到的是 `sc-abc123`、`css-1xy2z3` 这种 hash 类名，定位回源文件要靠 babel 插件加 displayName，且 source map 不总是准。
4. **和 React Server Component 不兼容**：styled-components 在 RSC 下完全无法工作（依赖 Context 和运行时挂载），是 Next.js App Router 时代它被逐渐放弃的关键原因。
5. **零运行时方案的代价是约束**：Vanilla Extract、Linaria 要求样式必须在构建时可静态分析，也就是说不能写"任意运行时 props 决定样式"的代码——动态颜色只能通过预定义的几个 variant 或 CSS 变量传入。

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

1. **挂载/卸载动画**：CSS 的 transition 需要元素先存在再切换 class，但 React 卸载组件时 DOM 直接被移除，没机会跑"离开动画"。`react-transition-group` 通过延迟卸载（先标记 leaving 类，等动画结束再 unmount）来处理。
2. **FLIP 列表动画**：列表重排时，CSS 不知道"这一项以前在哪"。FLIP 技术（First / Last / Invert / Play）需要在重排前后分别测量位置、计算 transform、再播放过渡——这是命令式的，必须用库（`framer-motion` 的 `<Reorder>`、`react-flip-toolkit`）。
3. **手势交互**：拖拽、滑动、捏合缩放需要追踪触摸/鼠标事件、计算速度、应用 inertia——纯 CSS 完全做不到，要么自己实现（复杂），要么用 `react-spring` / `framer-motion` 这类带物理引擎的库。
4. **联动动画**：A 元素动画结束后 B 才开始、AB 同时但有延迟差、按数据驱动的复杂时间轴——CSS 的 `animation-delay` 只能写死，不能根据 props 动态生成；JS 库可以编程组合时间轴。
5. **可中断/可逆**：CSS 动画跑到一半被打断会"跳变"；spring 物理动画可以从中断处平滑反向继续，体验自然得多。

选型压缩一下，大致就是：

1. 单元素 hover、显隐、淡入淡出，CSS `transition` 即可
2. Tab、抽屉、Modal 进出，常用 CSS 配 `react-transition-group`
3. 列表重排、共享元素过渡，更适合 `framer-motion`
4. 拖拽、滑动、手势，用 `react-spring`、`framer-motion` 或 `use-gesture`
5. 复杂时间轴、SVG 路径动画，常见是 GSAP 或 `framer-motion`
6. 数据可视化的大量动效，通常会落到 D3 或 visx

简单原则：**默认 CSS，不够用再上库**。但一旦决定上库，`framer-motion` 的 API 很现代，覆盖大部分日常需求，是现代 React 项目的首选。

### 说说你对 immutable 的理解？如何应用在 React 项目中？

Immutable 思想指的是数据更新时不直接修改原对象，而是返回一个新对象。

它在 React 中重要，因为：

1. React 的渲染优化不少依赖引用变化判断。
2. 不可变更新更利于状态追踪和调试。
3. 能降低意外副作用。

实际应用方式：

1. 展开运算符创建新对象、新数组。
2. 使用 `map`、`filter` 等返回新数组的方法。
3. 在复杂场景用 Immer 等库辅助不可变更新。

### React 中不可变更新有哪些常见模式和问题？

不可变更新的核心原则只有一句：**永远不要原地修改 state，每次都返回新引用**。围绕这条原则，对象、数组、嵌套结构各有自己的写法，也各有自己的问题。

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

**4. 嵌套结构的更新（容易问题的地方）**

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

**5. 常见问题**

1. **以为 `concat`、`slice` 是原地修改**：它们实际返回新数组，是不可变更新的好朋友；真正要警惕的是 `push/pop/shift/unshift/splice/sort/reverse/fill/copyWithin` 这一类。
2. **`useState` 传同一个引用不会触发更新**：`setList(list)` 即使内容变了，引用没换 React 也不会渲染。
3. **`useEffect` 依赖项是对象/数组**：每次 render 都是新引用 → 副作用每次都执行；要么用 `useMemo` 稳定引用，要么把依赖收敛成原始类型。
4. **深拷贝不是不可变更新**：`JSON.parse(JSON.stringify(x))` 会丢函数、Date、Map、Set，且性能差；只在路径上换引用即可。
5. **冻结也不等于不可变更新**：`Object.freeze` 只是防止你修改，并不会自动产新对象，常用于开发期检查。

**6. 工程上的最小理解模型**

1. **简单结构**：手写 `{ ...obj }` / `[...arr]`。
2. **层级深 / 操作多**：上 Immer。
3. **大量列表更新**：状态扁平化（`byId` + `ids`），更新一项时间复杂度从 O(n) 降到 O(1)。
4. **新 API 可关注**：`structuredClone` 适合一次性拿到深拷贝（不带函数），`Array.prototype.toSorted/toReversed/toSpliced/with` 提供原生不可变数组操作（ES2023）。

不可变更新不是为了形式统一，而是为了让 React 的引用比较机制能稳定工作。`memo`、selector、时间旅行调试、并发渲染这些能力都建立在这个前提上。

## 四、路由与状态管理

### 说说你对 Redux 的理解？其工作原理是什么？

Redux 是一个可预测的状态管理库，思路是用统一 store 管理全局状态，所有状态变更必须通过纯函数完成。

**先分清几个概念：**

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

**中间件的签名与三层结构：**

中间件这一层本质是个三层柯里化函数，每一层都对应不同的执行时机：

1. **第一层（注入 store API 阶段）**：在 `applyMiddleware` 时调用一次，把 `getState` 和 `dispatch` 注入到中间件，让中间件具备读状态和重新派发 action 的能力。
2. **第二层（串联 next 阶段）**：接收一个 `next` 参数，它指向"下一层中间件包过的 dispatch"。所有中间件的 next 串起来就形成了洋葱链，最里层的 next 才是 store 原始的 dispatch。
3. **第三层（处理 action 阶段）**：每次业务代码调用 `dispatch(action)` 时进入，中间件可以在这里做任何事——打日志、判断 action 类型、改写 action、异步触发新的 dispatch，甚至直接吞掉不调 next。

**实现本质：**

1. `applyMiddleware` 接收所有中间件，给它们一个简化版 `dispatch`。
2. 用 `compose` 把所有"处理 action"的函数串成洋葱模型，从外到内依次执行。
3. 把组合后的最外层函数赋给 store 的 dispatch，覆盖原始 dispatch。

**redux-thunk 的做法：**

thunk 的实现很精简，逻辑只有一句：**判断 action 是不是函数**。如果是函数，就把 `dispatch` 和 `getState` 注入并执行它——也就是说用户可以写"action creator 返回一个异步函数"，函数里随便发请求、dispatch 多个普通 action、读当前 state；如果不是函数，就直接走 `next(action)` 流程，把这个普通 action 交给后续中间件和 reducer。短短几行代码就让异步处理成为可能，所以不少人把"中间件"理解为"装饰器"。

### Redux Toolkit 解决了什么？为什么是现在的官方推荐？

Redux 早期被诟病"样板代码太多"——一个简单功能要写 action types、action creators、reducers 三套文件，再加上不可变更新的繁琐。Redux Toolkit（RTK）就是为解决这些问题而生。

**主要 API：**

1. `configureStore`：上手快的 store 配置，默认集成 thunk、devtools、序列化检测。
2. `createSlice`：把 action types、action creators、reducer 合并为一个 slice，自动生成。
3. `createAsyncThunk`：标准化处理异步请求的 pending/fulfilled/rejected 三个状态。
4. `createEntityAdapter`：列表/字典数据的 CRUD 工具。
5. `RTK Query`：声明式数据获取，自动缓存、失效、重取。

**createSlice 的思路：**

`createSlice` 是 RTK 最具代表性的 API，它把过去 Redux"三件套"合成一份声明式配置：

1. **`name` 字段是 slice 标识符**：会自动拼接成 action type 前缀（如 `counter/increment`），开发者不再需要手写 `const INCREMENT = 'counter/INCREMENT'` 这类常量。
2. **`reducers` 里每个方法既是 reducer，也对应自动生成的 action creator**：声明一个名为 `increment` 的方法，RTK 就会同时生成同名的 action creator，调用它返回 `{ type: 'counter/increment' }`。
3. **看似可变的写法**：`state.value += 1` 在原始 Redux 里是禁忌，但 RTK 内部用 Immer 的 `produce` 包了一层 reducer，开发者直接修改 draft，Immer 在背后产出全新对象。可读性大幅提升，且不可变性依然保留。
4. **payload 自动注入**：action creator 接收的参数会自动放进 `action.payload`，不需要手写 action creator 函数。
5. **导出整齐**：slice 对象上的 `actions` 是所有 action creators 的集合，`reducer` 是合并好的 reducer 函数，可以直接交给 `configureStore`。

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

举个对比：处理"搜索请求需要防抖且能被新请求取消"的场景，thunk 写起来比较绕，saga 用 `takeLatest` 一行完成。

### 你在 React 项目中是如何使用 Redux 的？项目结构如何划分？

项目里多半会这样用：

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

MobX 是一个基于响应式编程思想（TFRP，Transparent Functional Reactive Programming）的状态管理库，核心理念是："**任何源于应用状态的东西都应当自动地获得**"——UI 是状态的派生，状态变了 UI 自然变。

**核心三件套：**

1. **State**：被 `observable` 标记的可变状态，是数据源。
2. **Derivation**：从状态派生出来的值，分两种：
   - `computed`：纯派生值，有缓存。
   - `reaction`：会产生副作用的派生（如重新渲染、发请求）。
3. **Action**：修改 state 的方法。MobX 严格模式下要求所有变更必须在 action 里，便于事务批处理和调试。

常用 API 实际不用表格背，按用途记即可：

1. `observable` / `makeAutoObservable`：把对象或类变成可观察对象
2. `computed`：做派生值
3. `action` / `runInAction`：包住修改 state 的逻辑
4. `autorun` / `reaction` / `when`：做依赖追踪和副作用触发
5. `observer`：把组件接进响应式更新
6. `flow`：用 generator 处理异步 action

**典型 store 的角色划分：**

一个常见的 MobX store（以 TodoStore 为例）通常包含五种角色：

1. **State**：普通的类字段（如 `list` 和 `filter`），被 `makeAutoObservable` 包装后变成可观察值，每次读写都会被 MobX 拦截。
2. **Computed**：用 getter 声明的派生值（如 "可见 todo 列表"），结果会自动缓存——只要依赖的字段没变，多次访问不会重复计算。
3. **Action**：同步修改 state 的方法，可以直接修改可观察对象，多次修改会被合并到同一次响应。
4. **异步 action**：`async` 方法中 `await` 之后属于另一个 microtask，已经脱离原 action 上下文，所以必须用 `runInAction` 重新开一个 action 区块；或者用 `flow` 把 generator 函数包成异步 action。
5. **Observer 组件**：`observer(Component)` 把组件包成一个 reaction，render 期间读到的字段都会被收集为依赖；这些字段改变时，组件就会重新执行 render。

这种组织方式既保留了"直接改对象"的直观写法，又保证了响应式的精准更新。

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

**核心机制概括：**

整个依赖收集本质是 Atom 和 Derivation 之间的双向引用，再加上一个全局指针：

1. **读时登记**：每次属性被读取时，Atom 检查全局指针上的"当前正在执行的 derivation"，把这个 derivation 加入自己的观察者集合，同时让 derivation 也记录"我依赖了这个 Atom"——双向引用是为了卸载时能彻底清理。
2. **写时通知**：属性被修改时，Atom 遍历自己的观察者集合，把所有依赖它的 derivation 安排重新执行。
3. **执行前清空旧依赖**：每个 derivation 在重新执行前会先清空旧依赖集合，是为了支持"依赖动态变化"——比如 if/else 切换分支后，依赖集合会自然更新。
4. **嵌套支持**：用栈式保存/恢复"全局指针"的旧值，可以支持嵌套的 derivation（一个 derivation 在执行过程中触发另一个 derivation 的初始化）。
5. **批处理调度**：触发更新时常用 `queueMicrotask` 等机制延迟执行，把同一事件循环里的多次变更合并为一次重跑。

这就是为什么"在 observer 组件里读 store.x"等价于"自动 useSelector(s => s.x)"——MobX 在背后做了所有依赖追踪的脏活。

### MobX 中 action、runInAction、flow 有什么区别？

三者都是"修改 state 的入口"，但适用场景不同。

1. **`action`**：标记一个**同步**函数为 action。多次修改会被合并到一个事务，只触发一次响应。
2. **`runInAction`**：在异步代码中临时开一个 action 区块，常用于 `await` 之后包裹 state 修改。
3. **`flow`**：用 generator 写异步 action，每个 `yield` 处自动 runInAction，避免到处写 try/catch + runInAction。

**为什么 flow 要用 generator 而不是 `async/await`？** 因为 MobX 需要在每个异步暂停点（即 `yield`）之后重新进入 action 上下文。用 `async/await` 时 await 之后的代码已经脱离原函数调用栈，MobX 没办法自动识别"这里仍在 action 中"；而 generator 是 MobX 自己驱动 `next()` 调用的，它知道每个 `yield` 之后的恢复点，可以自动包一层 `runInAction`。代码读起来像同步，写起来不用手动 wrapper，两全其美。

**严格模式建议：** `enforceActions: 'observed'` / `'always'` 下，不写 action 直接改 state 会报错，这是大型项目推荐的配置——它强制所有变更都通过显式 action 完成，便于事务批处理和调试追踪。

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
2. 提供 `makeAutoObservable` 一行完成，自动推断每个属性是 observable / computed / action。

新项目推荐直接用 MobX 6 + `makeAutoObservable`。

### MobX 在 React 中的渲染优化和常见陷阱？

**渲染优化机制：**

1. `observer` 组件会把"组件 render 函数"包成一个 reaction。
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
2. **状态写法接近可变更新**：用 `set` 函数更新，但底层始终是替换式新对象（默认浅合并）。
3. **基于 hooks**：每个 store 就是一个自定义 hook。
4. **零依赖**：核心包压缩后约 1KB。

**主要特点大概是：**

1. API 极简，一个 `create` 函数完成 state、action、selector。
2. 选择器配合浅比较实现按需订阅，避免 Context "all or nothing" 重渲。
3. 内置中间件支持：`persist`、`devtools`、`immer`、`subscribeWithSelector`、`combine`。
4. 可在 React 之外使用（vanilla store）。
5. 完整支持 React 18 并发模式（基于 `useSyncExternalStore`）。

**Zustand 的核心使用心法：**

对照 Redux 的写法，Zustand 极简体现在五个细节：

1. **`create` 接收一个工厂函数**：参数是 `(set, get) => state`，返回值就是 store 的初始状态 + 操作方法。state 和 action 写在一起，没有 reducer / dispatch / action types 的拆分。
2. **`set` 默认是浅合并**：传一个对象进去等价于 `Object.assign(prevState, partial)`，不需要展开旧状态；也支持函数式写法 `set(prev => next)`。
3. **`get()` 用于在 action 内部读其他字段**：避免依赖外部闭包变量，保证读到的总是最新值，特别是异步 action 里。
4. **中间件像洋葱套娃**：例如 `devtools(persist(config))`，请求会逐层穿过包装。`persist` 把 state 写入 localStorage，`devtools` 把每次 set 上报到 Redux DevTools。
5. **消费时务必传 selector**：`useStore(s => s.bears)` 表示"我只关心 bears 字段"，只有它变化才会让组件重渲；如果直接 `useStore()` 不传 selector，整个 state 任何字段变都会触发重渲，严重退化性能。

### Zustand 的实现原理是什么？

Zustand 内部实际很薄，主要就是三块：**一个 vanilla store + useSyncExternalStore + selector 浅比较**。

 1. Vanilla store 提供"状态 + 订阅"的纯 JS 能力。
  2. useSyncExternalStore 把这个外部 store 安全接入 React 的并发渲染体系。
  3. Selector 浅比较 让每个组件只订阅自己关心的切片，避免无关重渲。

**Vanilla store 的本质：**

最底层是一个不依赖任何框架的纯 JS store，可以独立运行在 Node、Worker 或测试环境里。它只做三个动作：保存 state、提供 setState、维护订阅者列表。关键设计点：

1. **`setState` 支持两种形式**——直接传对象，或者传 `(prev) => next` 函数，跟 React 的 `setState` 完全一致。
2. **用 `Object.is` 做引用比较**，避免相同状态多次通知；所以 Zustand 推荐"返回新对象"的不可变更新。
3. **默认浅合并**：传对象进去会和旧 state 走 `Object.assign` 合并；`replace: true` 时直接替换整个 state。
4. **订阅者用 `Set`** 而不是数组，是为了取消订阅时 `O(1)` 删除，并且天然去重。
5. **首次执行工厂函数**：把用户写的 `(set, get) => state` 跑一次，把返回值作为初始状态，同时把 setState/getState 注入给 action 闭包。

**接入 React 的方式：**

接入层做了两步：

1. **把 store 包成一个 hook**：每次组件 render 时调用 `useSyncExternalStore`，参数是订阅函数和当前快照的 getter。React 会自己处理订阅、取消订阅、缓存快照、并发模式下避免 tearing 等所有边界。
2. **把 hook 和 store API 合并成一个对象**：让返回的 hook 同时拥有 `setState`/`getState`/`subscribe` 等方法，所以可以在 React 之外这样用：`useStore.getState()`、`useStore.subscribe(...)`，不必再绕一圈。

`useSyncExternalStore` 是 React 18 提供的官方"接外部 store"接口，保证：

1. 并发模式下不会出现 tearing（不同组件读到不同版本的 state）。
2. 自动处理订阅/取消订阅时机。
3. SSR 下能拿到一致的快照。

**选择器机制的关键：**

消费 store 时务必传 selector，并理解默认比较的边界：

1. **默认按 `Object.is` 比较**，selector 返回基本类型较稳妥。
2. **selector 返回对象时必须配 `shallow`**：因为每次 render 的对象都是新引用，默认比较永远判定为"变了"，组件就会无限重渲。`shallow` 比较的是对象内一层字段是否相等，避免这种假阳性。
3. **更稳的写法是用多个 selector 各取一个字段**，那样就不需要 `shallow`。
4. **不传 selector 等于订阅整个 state**——任何变化都会触发重渲染，这正是不推荐的用法。

### Zustand 中间件机制是怎么工作的？

Zustand 中间件本质是"包装 set 函数"的高阶函数，签名：

```ts
type Middleware = (config: StateCreator) => StateCreator
```

读这条签名：中间件接收一个 `StateCreator`（即用户写的 `(set, get) => state`），返回另一个 `StateCreator`。它通常会在内部调用原 config，但把 `set` 替换成自己包装过的版本，这样所有用户调用 `set(...)` 时都会先经过中间件这一层。

**`devtools` 中间件的做法：**

返回一个新的 StateCreator，在执行用户配置时把原始 `set` 替换成"先调原 set 更新状态，再把最新 state 上报给 Redux DevTools"的增强版；`get` 和 `api` 透传不变。这样每次状态变更都能被时间旅行调试器抓到，而用户代码无需额外改动。

**`immer` 中间件的做法：**

把 `set` 包装一层，让传入的"看似可变" recipe 先通过 Immer 的 `produce` 处理：Immer 内部用 Proxy 跟踪开发者对草稿的修改，最终产出新的不可变对象。包装后用户可以写 `set(state => { state.list.push(item) })` 这种直观写法，但底层 state 仍然是不可变更新，引用比较依然有效。

中间件可以层层嵌套（洋葱模型），比如 `devtools(persist(immer(config)))` 表示：用户调用 `set` → immer 把"可变"变"不可变" → persist 把新状态写入 storage → devtools 上报到面板。每一层都只关心自己那一段逻辑，互相解耦。

**常用中间件：**

1. `persist`：把 state 持久化到 localStorage / sessionStorage / AsyncStorage。
2. `devtools`：接入 Redux DevTools 调试。
3. `immer`：用 Immer 实现"可变写法、不可变结果"。
4. `subscribeWithSelector`：扩展 subscribe，可以选择性订阅。
5. `combine`：合并多个 state slice。

### 大型 Zustand 项目怎么组织？slices 模式是什么？

随着 store 变大，把所有 state 写在一个 create 里会变成"巨型函数"。slices 模式是社区主流方案：每个业务模块写成一个工厂函数，工厂函数接收 `(set, get, api)`，返回该模块的状态和方法；最后在统一的 `create` 调用里把所有工厂函数执行一遍，把结果展开合并。

**核心设计要点：**

1. **`(set, get, api)` 透传**：根 store 的工厂函数把这三个参数原封不动传给每个 slice，让它们共享同一个 store 实例。
2. **共享同一个 set**：所有 slice 内部的 `set` 是同一个函数，因此一个 slice 也可以读写另一个 slice 的字段，跨模块协作天然支持。
3. **对象展开合并**：把每个 slice 的返回值通过 `...` 展开到同一个对象里，等价于把所有 slice 的字段拍平到同一个 state 上。如果两个 slice 出现同名字段，后展开的会覆盖前面——所以命名时建议加业务前缀。
4. **TypeScript 友好**：可以给每个 slice 定义 `StateCreator<RootState, [], [], XxxSlice>` 类型，让 `set` / `get` 自动推导出整个 RootState 的形状。

**好处：**

1. 文件按业务拆分，便于维护。
2. 不同 slice 可以互相调用 `get()` 协作。
3. 类型推导可以用 `StateCreator` 精确化。

**要避免的反模式**：每个 slice 单独 `create` 一个 store，会失去跨 slice 协作能力，也增加理解成本。

#### 实践一：推荐的目录结构

```
src/
├── store/
│   ├── index.ts              # 根 store，组合所有 slice
│   ├── types.ts              # RootState 类型定义
│   ├── middleware.ts         # 自定义中间件
│   └── slices/
│       ├── auth.slice.ts     # 用户认证
│       ├── cart.slice.ts     # 购物车
│       ├── ui.slice.ts       # UI 状态（侧边栏、Modal）
│       └── settings.slice.ts # 用户偏好
├── features/
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   └── hooks.ts          # 该 feature 独有的 selector hook
│   └── cart/
│       ├── CartList.tsx
│       └── hooks.ts
```

按业务拆 slice、按 feature 组织 UI——store 的物理边界和业务边界对齐，新人上手能立刻理解"购物车逻辑在 cart.slice.ts、购物车 UI 在 features/cart/"。

#### 实践二：单个 slice 的写法

每个 slice 是一个工厂函数，配上独立的类型定义和 action 命名前缀：

```ts
// store/slices/auth.slice.ts
import { StateCreator } from 'zustand'
import type { RootState } from '../types'

export interface AuthSlice {
  user: User | null
  token: string | null
  isAuthenticated: boolean
  authLogin: (email: string, password: string) => Promise<void>
  authLogout: () => void
  authRefreshToken: () => Promise<void>
}

export const createAuthSlice: StateCreator<
  RootState,
  [['zustand/immer', never]],   // 中间件元组（用 immer 时声明）
  [],
  AuthSlice
> = (set, get) => ({
  user: null,
  token: null,
  isAuthenticated: false,

  authLogin: async (email, password) => {
    const { user, token } = await api.login(email, password)
    set((state) => {
      state.user = user
      state.token = token
      state.isAuthenticated = true
    })
  },

  authLogout: () => {
    set((state) => {
      state.user = null
      state.token = null
      state.isAuthenticated = false
    })
    // 清理购物车（跨 slice 调用）
    get().cartClear()
  },

  authRefreshToken: async () => {
    const token = get().token
    if (!token) return
    const fresh = await api.refresh(token)
    set((state) => { state.token = fresh })
  },
})
```

要点：

1. **action 加业务前缀**（`authLogin` 而不是 `login`）：避免合并 slice 时同名冲突。
2. **`set` 用 immer 风格**：配合 immer 中间件后可以直接 `state.user = user`，不用展开。
3. **跨 slice 协作通过 `get()`**：`authLogout` 调用 `get().cartClear()`，自然完成"登出后清空购物车"的联动。

#### 实践三：根 store 组合多个 slice

```ts
// store/index.ts
import { create } from 'zustand'
import { devtools, persist, immer } from 'zustand/middleware'

import { createAuthSlice, AuthSlice } from './slices/auth.slice'
import { createCartSlice, CartSlice } from './slices/cart.slice'
import { createUiSlice, UiSlice } from './slices/ui.slice'
import { createSettingsSlice, SettingsSlice } from './slices/settings.slice'

export type RootState = AuthSlice & CartSlice & UiSlice & SettingsSlice

export const useStore = create<RootState>()(
  devtools(
    persist(
      immer((...a) => ({
        ...createAuthSlice(...a),
        ...createCartSlice(...a),
        ...createUiSlice(...a),
        ...createSettingsSlice(...a),
      })),
      {
        name: 'app-storage',
        // 只持久化部分 slice：UI 状态不持久化
        partialize: (state) => ({
          token: state.token,
          settings: state.settings,
          cart: state.cart,
        }),
      },
    ),
    { name: 'AppStore' },
  ),
)
```

要点：

1. **`RootState` 是所有 slice 类型的交集**：TypeScript 会自动校验是否有字段冲突，命名规范化的好处在编译期就能享受到。
2. **中间件按需嵌套**：`devtools` 包最外层方便调试，`persist` 处理持久化，`immer` 让 slice 内部能写"看似可变"的代码。
3. **`partialize` 选择性持久化**：UI 状态（侧边栏开合、Modal 展示）不需要刷新后保留，只持久化业务关键状态——避免 localStorage 越塞越大。

#### 实践四：组件中的高效消费

slice 模式下，**selector 写法决定性能上限**：

```tsx
// ❌ 反模式：订阅整个 state，任何字段变都重渲
function CartBadge() {
  const state = useStore()
  return <span>{state.cart.items.length}</span>
}

// ❌ 反模式：返回新对象但没配 shallow，无限重渲
function UserMenu() {
  const { user, authLogout } = useStore((s) => ({
    user: s.user,
    authLogout: s.authLogout,
  }))
  return <button onClick={authLogout}>{user?.name}</button>
}

// ✅ 正确：每个字段单独订阅
function UserMenu() {
  const user = useStore((s) => s.user)
  const authLogout = useStore((s) => s.authLogout)
  return <button onClick={authLogout}>{user?.name}</button>
}

// ✅ 或者用 useShallow 处理对象返回
import { useShallow } from 'zustand/react/shallow'

function UserMenu() {
  const { user, authLogout } = useStore(
    useShallow((s) => ({ user: s.user, authLogout: s.authLogout })),
  )
  return <button onClick={authLogout}>{user?.name}</button>
}
```

#### 实践五：把 selector 抽成自定义 hook

每个 feature 写一组语义化 hook，组件层不直接依赖 store 形状，重构时改动最小：

```ts
// features/cart/hooks.ts
export const useCartItems = () => useStore((s) => s.cart.items)
export const useCartTotal = () =>
  useStore((s) =>
    s.cart.items.reduce((sum, x) => sum + x.price * x.quantity, 0),
  )
export const useCartActions = () =>
  useStore(
    useShallow((s) => ({
      cartAdd: s.cartAdd,
      cartRemove: s.cartRemove,
      cartClear: s.cartClear,
    })),
  )

// 组件里使用
function CartList() {
  const items = useCartItems()
  const { cartRemove } = useCartActions()
  // ...
}
```

#### 实践六：在非 React 代码中使用

slice 模式天然支持脱离 React 调用：API 拦截器、路由守卫、Service Worker、Web Worker 都能直接操作 store：

```ts
// api/interceptor.ts
import { useStore } from '@/store'

axios.interceptors.request.use((config) => {
  const token = useStore.getState().token
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

axios.interceptors.response.use(
  (res) => res,
  async (err) => {
    if (err.response?.status === 401) {
      // 直接调 action，不需要 React 上下文
      await useStore.getState().authRefreshToken()
      return axios(err.config)
    }
    throw err
  },
)
```

这是 Zustand 相比 Redux 的一大优势：store 是模块级单例，任何地方 `import` 都能拿到最新状态，不需要 dispatch/subscribe 那一套样板。

#### 实践七：测试时重置 store

slice 模式下测试容易，把初始状态抽出来即可：

```ts
// store/index.ts
const initialState = {
  user: null,
  cart: { items: [] },
  // ...
}

export const useStore = create<RootState>()(/* ... */)

// 测试工具
export const resetStore = () => useStore.setState(initialState, true)

// 单测
beforeEach(() => resetStore())
```

`setState(state, true)` 的第二个参数是 replace 标记，用整个新 state 替换当前 state，比浅合并更彻底——这是单测中"每个 case 拿到干净 store"的标准做法。

#### 反模式速查

1. **每个业务一个独立 store**：跨业务联动只能靠 effect 监听 + 手动同步，回到 Context 时代的"状态散落各处"。
2. **slice 内调用 `useStore`**：slice 工厂函数不能用 hook，只能用 `set`/`get`。
3. **action 命名不加前缀**：合并多个 slice 时极易撞名，且 DevTools 里看到 `add` / `remove` 这类通用名字根本不知道来自哪个模块。
4. **把派生值放进 state**：派生值（如购物车总价）应该用 selector 实时计算，写进 state 容易和源数据脱节。
5. **selector 里返回新对象不配 shallow**：每次 render 都判不等，组件无限重渲。

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

Redux store 必须通过 4 个固定 API 访问；MobX store 就是个普通对象，怎么定义就怎么访问；Zustand store 能当 hook 用，也能当普通对象用。

**2. 数量与作用域**

这里也能看出三者的项目结构差异：

1. Redux 默认是单一 store，并且通常要通过 `<Provider>` 挂到组件树上
2. MobX 更鼓励按业务拆多个 store，不少场景直接 `import store` 就能用
3. Zustand 单 store、多 store 都行，而且默认不需要 Provider

**3. 内部状态的存储形式**

这里也很好区分：

1. Redux 维护的是一棵严格不可变的对象树，更新时一定返回新引用
2. MobX 是可变的响应式对象，字段变化会自动通知观察者
3. Zustand 看起来也像普通对象，但每次 `setState` 还是会生成新对象

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

Redux 的"store 内核纯净 + 中间件扩展"是历史包袱也是好处：清晰但啰嗦。MobX/Zustand 的 store 自身就能装异步逻辑，但相应失去了"action 是可序列化数据"的好处。

调试能力方面，Redux 依然最强，因为它天生就是事件流模型，每次变化都有明确 action 记录；MobX 更偏响应式对象图，能看依赖关系，但不擅长时间旅行；Zustand 介于两者之间，足够轻，调试能力主要靠中间件补。

### Redux、MobX、Zustand 全方位对比？怎么选型？

这三个库的区别，最后还是落到状态模型不同：

1. Redux 是事件流，状态变化要通过 action 和 reducer。
2. MobX 是响应式对象，字段变化会自动通知依赖它的地方。
3. Zustand 是轻量 store，思路最直接，接近“我有一份状态，谁需要谁订阅”。

实际选型不用展开成大表格，抓住几个判断即可：

1. 团队强调可追踪、可回放、流程统一，用 Redux。
2. 业务对象关系复杂、派生状态多、希望响应式体验更好用，用 MobX。
3. 项目偏轻量，只想要一个简单的全局状态容器，用 Zustand。

较简单粗暴讲，Redux 偏纪律，MobX 偏响应式，Zustand 偏轻量。

### 简单实现一个 mini-Redux？

mini-Redux 真压到最核心，实际就这三块：

1. 保存一份 state
2. 提供 `getState`
3. 提供 `dispatch` 和 `subscribe`

再往上加，就是：

1. `combineReducers` 把多个 reducer 合起来
2. `compose` 把函数串起来
3. `applyMiddleware` 把中间件挂到 dispatch 外面

所以 Redux 内核并不复杂，复杂的是围绕它长出来的工程化生态。

### 简单实现一个 mini-MobX？

mini-MobX 抓住两个动作即可：

1. 读取时收集依赖
2. 修改时通知依赖重跑

常见的做法就是 `Proxy + 全局 activeReaction`。字段被访问时记录是谁在读，字段变化时把这些观察者再执行一遍。

MobX 的特点，就是更新粒度很细：没读过的字段变化，不会触发不相关的 reaction。

### 简单实现一个 mini-Zustand？

mini-Zustand 的骨架也很薄：

1. 闭包里放一份 state
2. 用 `Set` 存订阅者
3. 提供 `setState`、`getState`、`subscribe`
4. 再用 `useSyncExternalStore` 接到 React 上

所以 Zustand 的轻量不是宣传话术，它确实只保留了一个外部 store 最核心的那层能力。

### 说说你对 React Router 的理解？常用的 Router 组件有哪些？

React Router 是 React 生态中的路由方案，用来管理前端页面切换和 URL 映射关系。

常见 Router 组件有：

1. `BrowserRouter`
2. `HashRouter`
3. `MemoryRouter`

它的作用是：让 URL 和组件树建立对应关系。

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

优化的前提，是先找到实际的性能瓶颈，而不是盲目做 memo。

### 说说 React diff 的原理是什么？

React 的 diff 目标是高效比较新旧虚拟节点，找出最小更新范围。

React 的 diff 可以抓住三条规则：

1. 只在同层比较，不做跨层复用
2. 类型变了就直接换整棵子树
3. 列表节点靠 `key` 判断是不是同一个

真正到列表场景时，React 的策略也不复杂：先按顺序比一轮，遇到对不上号的，再把剩余旧节点放进 Map 里按 `key` 查。它放弃了理论上的最优解，但换来了足够稳定的复杂度和实现成本。

### 主流框架的 diff 算法对比？

虚拟 DOM diff 不是 React 独有。Vue、Preact、Svelte 这些框架也都在做同样的工作，只是路线不一样。

可以把差别简单记成这样：

1. React 更偏稳定、通用，配合 Fiber 做调度。
2. Vue 2 更偏双端比较。
3. Vue 3 在编译期和运行时都做了更多静态优化。
4. Svelte、Solid 这类方案把工作更多前移到编译期，运行时更轻。

所以它们的差别不只是“谁更快”，而是优化点放在哪一层：有的侧重运行时调度，有的侧重编译期消解。

**10. 各算法的实际性能对比（仅供参考）**

按 [js-framework-benchmark](https://krausest.github.io/js-framework-benchmark/) 中"创建 1000 行 + 部分更新 + 完全替换"的综合分数：

| 框架 | 相对 vanilla 慢多少倍（综合） |
| --- | --- |
| Vanilla JS | 1.00 |
| SolidJS | 1.10 |
| Svelte | 1.20 |
| Inferno | 1.35 |
| Preact | 1.55 |
| Vue 3 | 1.50 |
| React 19 | 1.95 |
| Vue 2 | 2.00 |

数据会随版本变化，但**相对位次基本稳定**：编译时方案（Svelte / Solid）> 优化过的运行时（Vue 3 / Inferno）> 通用框架（React / Vue 2）。

**11. 各种优化手段怎么取舍**

1. **React 的 diff**：算法保守、配合 Fiber 可中断，性能换工程化、调度能力、跨平台。
2. **Vue 2 双端比较**：在常见列表操作上比 React 快，但同步、不可中断。
3. **Vue 3 LIS + Block**：把"哪些节点真的会变"压到编译期，运行时 diff 量级骤降。
4. **Svelte / Solid**：根本不做 diff，编译期生成精确更新代码——上限最高，但动态性受限。
5. **Preact / Inferno**：极致轻量化或极致性能，是 React 在特定场景的替代。

**对 React 团队的影响**：React 19 的 React Compiler 正是在向 Vue 3 / Solid 学习——把"哪些值真的会变"放到编译期分析，运行时少做无用功。可以预见未来 React 也会向"编译期优化 + 运行时调度"的方向继续演进。

### 说说对 Fiber 架构的理解？解决了什么问题？

Fiber 可以先简单记成这样：React 不再要求一次性把整棵树同步跑完，而是把更新拆成不少小任务，自己决定什么时候继续、什么时候暂停、什么时候让更急的任务先做。

它解决的是早期 React 的一个老问题：树一深、更新一重，就会长时间霸占主线程，输入和动画都跟着卡。Fiber 把这个过程拆开以后，React 才有了实际的调度空间。

往下说，Fiber 带来的变化主要就是三点：

1. 更新可以中断
2. 中断后可以恢复
3. 不同更新可以排优先级

**2. Fiber 节点：链表化的虚拟 DOM**

每个 Fiber 节点大致如下（精简版）：

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

重点就是 `return` / `child` / `sibling` 这三个指针。它们把原本的树结构改造成一套可回退、可继续的遍历链路，React 才能在处理过程中随时停下来，把主线程让给更急的事。

**3. 两棵树的双缓冲（current / workInProgress）**

React 在内存里会同时维护两棵 Fiber 树：

1. `current`：当前显示在屏幕上的树。
2. `workInProgress`（WIP）：本次更新正在构建的树。

两棵树通过 `alternate` 指针互相关联。正在构建的新树完成后，会接替旧树成为新的 `current`。这样做的好处是：中途就算被打断，也不会影响当前已经显示在屏幕上的结果。

可以把它理解成：所有新一轮计算都先在草稿树里做，确认没问题了再一次性交换到前台。

**4. 两阶段：Render 阶段（可中断）+ Commit 阶段（不可中断）**

这是 Fiber 架构最核心的拆分：

| 阶段 | 名称 | 是否可中断 | 做什么 |
| --- | --- | --- | --- |
| 1 | Render（也叫 Reconcile） | ✅ 可中断、可丢弃、可重做 | 构建 WIP 树，对每个 Fiber 调用 `beginWork` / `completeWork`，diff 出要做的副作用，挂在 `flags` 上 |
第二阶段是 Commit。这个阶段不可中断，会把副作用一次性应用到真实 DOM，并触发生命周期和 effect。

Render 阶段必须尽量保持无副作用，原因简单：它可能被打断、重做、重复执行。如果在这里直接发请求、改 DOM、写订阅，问题就会很难收拾。

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

用位运算批量合并、判断包含、抽取最高优——能表达单个优先级，也能表达"一组优先级一起处理"。`useTransition` 标记的更新会落到 Transition lane，可以被普通输入打断；`useDeferredValue` 类似。

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
4. **Hooks 实现的基础**：函数组件的 hooks 状态就挂在对应 Fiber 节点的 `memoizedState` 上（一条单链表），每次 render 按调用顺序遍历——所以 hooks 不能写在条件分支里。

**10. 浓缩到一段总结**

Fiber 不是某个新算法，而是 **一种把"组件树渲染"改造成"可被调度的工作单元流"的架构**——它通过链表结构让递归变成可中断的循环，通过双缓冲让中断可丢弃，通过双阶段让副作用集中可控，通过 Lane 模型支持优先级并发，最终让 React 从"同步递归框架"演化成了"并发 UI 运行时"。

### 说说 React 性能优化的手段有哪些？

可以从几个层面回答：

1. 代码层：组件拆分、`memo`、`useMemo`、`useCallback`
2. 数据层：不可变更新、状态最小化
3. 渲染层：虚拟列表、按需渲染、懒加载
4. 网络层：接口缓存、资源压缩、CDN
5. 构建层：代码分割、路由懒加载

讲性能优化时，按“渲染优化 + 资源优化 + 交互优化”去组织会比较清楚。

## 六、工程落地与服务端渲染

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

价值：

1. 提升首屏加载体验
2. 改善 SEO

基本流程：

1. 服务端根据路由和数据生成 HTML。
2. 浏览器收到后先展示首屏内容。
3. 客户端再接管并完成 hydration。

实际方案常见有 Next.js，或者基于 React 官方服务端渲染 API 自建。

### 说说你在使用 React 过程中遇到的常见问题？如何解决？

这类题考的是实战经验，不要泛泛而谈。下面用"问题现象 + 原因 + 解决办法"的结构，给出几个高频真实场景。

#### 场景一：搜索框输入卡顿

**问题现象**：搜索框输入"react"五个字符，输入框反应明显滞后，每次输入感觉延迟 100~200ms。商品列表有 2000 多项，每次输入都要重渲整个列表。

**原因**：

1. 输入框的 `value` 由 `useState` 控制，setState 触发整棵子树 render。
2. 列表组件没做 memo，每次父组件 render 都跟着重新计算 + diff 2000 个 item。
3. 列表 item 内部还有时间格式化、价格计算等同步开销。

**解决办法**：

1. **拆分组件 + memo**：把列表抽成独立组件用 `React.memo` 包裹，把昂贵计算用 `useMemo` 缓存。
2. **`useDeferredValue` 降优先级**：搜索关键词存两份——一份立即更新（绑定输入框）、一份用 `useDeferredValue` 延迟（传给列表）。React 18 后用户输入会优先响应，列表渲染会被打断重排。
3. **虚拟列表**：超过 1000 项的场景上 `react-window` / `react-virtuoso`，只渲染可视区域的 20~30 项。
4. **服务端搜索 + 防抖**：本地过滤如果数据量再大，改成防抖 300ms 后请求接口。

```jsx
function Search() {
  const [query, setQuery] = useState('')
  const deferredQuery = useDeferredValue(query)
  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <List query={deferredQuery} />  {/* 用延迟值，渲染可被打断 */}
    </>
  )
}
```

#### 场景二：列表删除后，中间项的 input 内容错位

**问题现象**：列表渲染了 5 个用户卡片，每个卡片有个备注输入框。用户在第 3 个卡片输入"VIP"，然后点击第 1 个卡片的删除按钮，结果"VIP"出现在了第 2 个卡片（原来的第 3 个）。

**原因**：列表用了数组下标作为 key：`<Card key={index} />`。删除第 1 项后，原来下标 1 的节点变成下标 0，原来下标 2 变成下标 1……React diff 看到 key 都没变（0/1/2/3），就把"换了内容的节点"当作"没变的节点"复用，DOM 被复用但内容、组件 state（包括 input 的 value）全部对应错位置。

**解决办法**：

1. **用业务唯一 id 作 key**：`<Card key={user.id} />`。React diff 能识别"id=3 的节点从位置 2 移到了位置 1"——DOM 整体移动，内部 state 跟着走，不会错位。
2. **永远不用下标作 key**，除非满足三个条件之一：列表纯展示永不变动、item 无 state、item 无受控输入。
3. **如果数据真的没有 id**：用 `crypto.randomUUID()` 在添加时生成稳定 id，而不是渲染时算下标。

#### 场景三：`useEffect` 里 setInterval 计数器永远停在 1

**问题现象**：写了一个秒数自增的计数器，UI 显示 0 → 1 之后就不动了。

**原因**：经典的"闭包陷阱（Stale Closure）"。

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1)  // count 永远是 0
  }, 1000)
  return () => clearInterval(id)
}, [])  // ❌ deps 写空数组,effect 只跑一次
```

`useEffect` deps 是空数组，effect 只在 mount 时执行一次，setInterval 回调闭包捕获的是初始 render 的 `count = 0`。每秒都是 `setCount(0 + 1)`，UI 一直显示 1。

**解决办法**：

1. **函数式更新**（最推荐）：`setCount(c => c + 1)`，c 来自 React 最新状态队列，跟闭包无关，deps 可以保持空数组。
2. **正确写依赖**：`useEffect(..., [count])`，但每次 count 变化都会销毁重建定时器，浪费。
3. **用 ref 保存最新值**：在长寿命订阅（WebSocket、全局事件）里用 ref 兜底，避免 effect 反复重启。
4. **开 ESLint `react-hooks/exhaustive-deps`**：这条规则会自动警告漏写的依赖，是闭包陷阱的第一道防线。

#### 场景四：快速切换 Tab，旧请求覆盖新请求

**问题现象**：Tab1 加载耗时 2 秒，Tab2 加载耗时 100ms。用户点 Tab1 → 立刻切到 Tab2，100ms 后 Tab2 数据先显示，但 2 秒后 Tab1 的数据"覆盖"了 Tab2 的内容，看到的是 Tab1 的数据。

**原因**：**异步竞态（race condition）**。两次请求并发，先发出的反而后到达，后到达的 setState 覆盖了先到达的结果。

**解决办法**：

1. **取消标志位**：effect 返回 cleanup，旧请求返回时直接丢弃。

```jsx
useEffect(() => {
  let cancelled = false
  fetch(`/api/${tabId}`).then(data => {
    if (!cancelled) setData(data)
  })
  return () => { cancelled = true }
}, [tabId])
```

2. **`AbortController`**：更彻底——直接中断请求，不仅丢弃响应，连网络请求都终止。

```jsx
useEffect(() => {
  const ctrl = new AbortController()
  fetch(`/api/${tabId}`, { signal: ctrl.signal })
    .then(data => setData(data))
    .catch(err => { if (err.name !== 'AbortError') throw err })
  return () => ctrl.abort()
}, [tabId])
```

3. **用 React Query / SWR**：它们内置请求去重、缓存、自动取消，业务代码完全不用关心竞态。

#### 场景五：组件卸载后还在 setState，控制台爆警告

**问题现象**：从详情页点返回后，控制台出现 "Can't perform a React state update on an unmounted component" 警告。

**原因**：组件已经从 DOM 卸载，但 setTimeout / fetch / WebSocket 回调还在执行，回调里调了 `setState`——React 检测到目标组件已卸载，发出警告（React 17 还会警告，18 后悄悄忽略，但内存泄漏依然存在）。

**解决办法**：

1. **cleanup 清理副作用**：每个有"未来回调"的 effect 都要返回 cleanup。

```jsx
useEffect(() => {
  const id = setTimeout(() => setData('done'), 5000)
  return () => clearTimeout(id)
}, [])
```

2. **mounted 标志位**：用于 Promise 链等不能取消的场景。

```jsx
useEffect(() => {
  let mounted = true
  api.get().then(d => { if (mounted) setData(d) })
  return () => { mounted = false }
}, [])
```

3. **订阅型副作用必须 unsubscribe**：WebSocket、resize 监听、全局事件总线都要在 cleanup 里取消订阅，否则会泄漏。

#### 场景六：Context 一变，几十个无关组件全部重渲

**问题现象**：用户登录态用 Context 管理，登录后 setUser 触发了订阅 UserContext 的整个组件树重渲——包括只用了主题色、跟用户信息无关的 100 多个组件。

**原因**：

1. Context value 引用变化时，**所有消费该 Context 的组件无差别重渲**，即使它们只用了 value 中的一小部分。
2. 用户信息 + 主题 + 路由信息塞在同一个 Context 里，任何一个字段变都会牵一发动全身。
3. value 写成 `{ user, theme, ...}` 这种对象，每次 Provider render 都是新引用——浅比较失效，memo 子组件也救不了。

**解决办法**：

1. **拆分 Context**：按变化频率分成 `UserContext`、`ThemeContext`、`AuthContext` 等，只让真正需要某字段的组件订阅对应 Context。
2. **用 Zustand / Jotai 替代高频写入的 Context**：它们内置 selector + 浅比较，组件只订阅切片。Context 留给"低频写入 + 多处读取"（主题、i18n、路由）。
3. **value 用 useMemo 稳定引用**：`<Provider value={useMemo(() => ({ user, login }), [user])}>`。
4. **服务端状态用 React Query**：Context 不要管接口数据，React Query 自带缓存 + 重渲控制。

#### 场景七：`useState` 初始值用昂贵计算，导致每次 render 都跑一遍

**问题现象**：页面打开时卡顿明显，Profiler 显示 `Page` 组件首次 render 耗时 800ms。

**原因**：`useState(parseGiantJson(rawData))` 这种写法，每次 render 都会算 `parseGiantJson`，虽然 React 只采用第一次的结果作为初始值，但**计算本身每次都在跑**——这是常见误区。

**解决办法**：

1. **传函数当初始值**（lazy initial state）：

```jsx
// ❌ 每次 render 都执行
const [state, setState] = useState(parseGiantJson(rawData))

// ✅ 只在首次 render 执行
const [state, setState] = useState(() => parseGiantJson(rawData))
```

2. **同理适用 `useReducer`**：第三个参数 `init` 是惰性初始化函数。
3. **派生值用 `useMemo`**：如果是依赖 props/state 计算出来的，用 `useMemo` 缓存。

#### 场景八：表单受控输入打字越来越卡

**问题现象**：一个含 50 个字段的复杂表单，每输一个字符整个表单组件都重渲一次，字段多的时候输入明显延迟。

**原因**：用 `useState` 把整个表单数据放在父组件，每次 onChange 都 `setState({...form, [name]: val})`，触发整个表单 + 所有字段组件重渲。50 个字段时性能塌方。

**解决办法**：

1. **`react-hook-form`**：非受控 + 订阅模式，只让"真正变化的字段"重渲，其他字段引用 ref 不重渲。50 字段表单输入流畅度和单字段一样。
2. **字段隔离 + 局部 state**：如果不引库，把每个字段拆成独立组件，字段内部自己 `useState`，只在 submit 时收集。
3. **debounce 校验**：实时校验改成 debounce 300ms 触发，避免每个字符都触发一遍校验逻辑。

#### 场景九：点击按钮后 ref 的 DOM 还是旧的

**问题现象**：点击按钮 setState 切换显示一个 input，然后立即让它聚焦——但 `ref.current.focus()` 没生效，要再点一次才行。

**原因**：setState 不是同步的，调用后 DOM 还没更新，此时 `ref.current` 要么是 `null`（组件还没渲染），要么是旧节点。

**解决办法**：

1. **`useEffect` 在渲染后聚焦**：

```jsx
useEffect(() => {
  if (visible) inputRef.current?.focus()
}, [visible])
```

2. **`flushSync` 强制同步更新**：确实需要"setState 后立刻操作 DOM"时用，但要慎用——会破坏并发渲染优化。

```jsx
import { flushSync } from 'react-dom'
flushSync(() => setVisible(true))
inputRef.current.focus()
```

3. **`autoFocus` 属性**：简单场景直接用 `<input autoFocus />`。

#### 场景十：开发环境 effect 跑两次，导致请求发了两次

**问题现象**：本地开发时，组件 mount 一次，但 useEffect 里的 `console.log` 打了两次，接口也请求了两次；部署到生产环境后正常。

**原因**：React 18 的 `<StrictMode>` 在开发环境会**故意 mount → unmount → mount 一次组件**，目的是暴露不健壮的 effect 清理逻辑。生产环境不会有这个行为。

**解决办法**：

1. **不要关 StrictMode**：这是 React 帮你提前发现 bug 的机制，关掉等于鸵鸟。
2. **正确写 cleanup**：每个 effect 都要能被"重复执行"——副作用要可清理、可幂等。
3. **请求用 React Query / SWR**：它们自带请求去重，即使 effect 跑两次也只发一个网络请求。
4. **如果是 logger / 埋点**：用 ref 记录"是否已经上报过"，或者在路由守卫里上报而不是组件 mount 时。

---

**整理思路**：

不要把所有场景都列出来，选 2~3 个最典型的问题讲透，再补充几个其他常见类型。**单个问题展开讲细节（具体复现步骤、profiler 截图思路、最终解法对比）**，比泛泛列十个问题更有价值。

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

栈递归是 V8 的调用栈，**JS 层没法暂停一个正在执行的函数**。要做"渲染到一半让出主线程"，只能把递归改成迭代：把每个组件抽象成一个**可保存的工作单元**，自己维护遍历指针，需要让出时记录位置、下次从这个位置接着跑。

这就是 Fiber 架构的来历——它用**链表 + 自己实现的调度循环替换原生调用栈**，让"可中断、可恢复、可优先级调度"成为可能。

**4. 留下的历史包袱**

虽然 16 重写了底层，但 React 选择**对外 API 尽量兼容**——所以直到 17、18 还在持续清理：

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

16.3 是很关键的一个小版本，主要为 Fiber 异步渲染做铺垫。

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
3. **类组件理解成本重**：`this`、`bind`、绑定事件处理函数等问题。

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

   1. 不再复用 SyntheticEvent 对象，每次事件触发都创建一个新的 SyntheticEvent 实例,用完即丢,交给 GC 回收。

     2. 事件回调结束后不再 nullify 属性。事件对象的属性会一直保留,可以随时异步访问

5. **`componentDidMount` 调用时序更严格**：父子组件 mount 顺序、effect 执行顺序更可预测。

React 17 更像一个过渡版本，它的价值主要在于为后续多版本并存和升级路径铺路。

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

React 18 之后启动应用必须用新的 `createRoot` API，旧的 `ReactDOM.render` 仍可用但会输出"已废弃"警告，且不会启用任何并发能力。升级到 React 18 但只改包版本不改入口，等于没真正升级。

```js
// React 17
import ReactDOM from 'react-dom'
ReactDOM.render(<App />, document.getElementById('root'))

// React 18
import { createRoot } from 'react-dom/client'
createRoot(document.getElementById('root')).render(<App />)
```

注意 `createRoot` 来自 `react-dom/client`（不是 `react-dom`），返回一个 root 对象，后续可以多次调用 `root.render(...)` 更新，也可以 `root.unmount()` 卸载。这种"先创建 root 再 render"的模式，是为了让 React 内部能持有更多渲染元数据（优先级、并发模式、错误处理回调等）。

### 并发渲染（Concurrent Rendering）怎么开启？开启后效果是什么？

并发渲染是 React 18 最核心的能力，但它**不是默认全开的开关**——开启分两步：升级到并发模式的运行时，再用并发 API 显式标记哪些更新走低优先级。

#### 一、开启方式

**第一步：换 `createRoot` 入口**

这是激活并发模式的前提。只要应用通过 `createRoot` 启动，整棵树就具备了**并发模式的运行时能力**：自动批处理、Transitions、Suspense 数据获取、`useDeferredValue`、`useSyncExternalStore` 都才能正常工作。

```js
// ❌ 17 风格,只能跑同步渲染
ReactDOM.render(<App />, document.getElementById('root'))

// ✅ 18 风格,启用并发模式运行时
import { createRoot } from 'react-dom/client'
createRoot(document.getElementById('root')).render(<App />)
```

这里要分清楚一点：换了 `createRoot` 不等于"所有渲染都自动变并发"——大多数 `setState` 还是高优先级、同步处理。换 createRoot 只是**让并发能力可用**，是否真的让某次更新走并发，还要看代码里有没有用 `startTransition` 这类 API。

**第二步：显式标记低优先级更新**

通过下面这些 API 主动告诉 React"这里更新可以被打断"：

| API | 作用 | 常见场景 |
| --- | --- | --- |
| `startTransition(fn)` | fn 内的 setState 标记为 transition | 搜索结果列表、Tab 切换 |
| `useTransition()` | 返回 `[isPending, startTransition]`，多了一个加载态 | 同上，需要展示 spinner |
| `useDeferredValue(value)` | 延迟某个值的传递 | 输入框防抖、昂贵图表 |
| `<Suspense fallback={...}>` | 渲染中遇到未就绪资源会挂起 | 数据获取、代码分割 |
| `lazy(() => import())` | 配合 Suspense 实现组件级懒加载 | 路由分包 |

不用这些 API 的场景，更新依然走默认的同步优先级——**并发渲染是机会主义**，只有显式标记的更新才享受可中断、可调度。

**第三步：升级生态库**

老版状态库（旧版 react-redux、Zustand 1.x、Recoil 旧版）在并发模式下可能出现"tearing"——同一次更新中不同组件读到不同版本的 state。需要：

1. 升级到使用 `useSyncExternalStore` 的版本（react-redux ≥ 8、Zustand ≥ 4）。
2. 路由库升级到 React Router v6.4+ / TanStack Router。
3. 数据库 / 表单库（react-hook-form、Formik）也都需要 18 兼容版本。

#### 二、开启后的效果

并发模式不是"代码自动变快"，而是带来**五项以前做不到的能力**：

**效果 1：紧急更新优先响应，不再被低优先级渲染卡住**

经典的搜索框场景：

```jsx
function Search() {
  const [query, setQuery] = useState('')
  const [list, setList] = useState([])
  const [, startTransition] = useTransition()

  function onChange(e) {
    setQuery(e.target.value)              // 高优先级:输入框立即刷新
    startTransition(() => {
      setList(filterHugeList(e.target.value))  // 低优先级:列表渲染可被打断
    })
  }
  // ...
}
```

效果对比：

1. **未开并发**：输入"react"五个字符,每个字符都要等 2000 项列表重渲完才能响应下一次输入,输入框出现明显卡顿。
2. **开了并发**：输入框始终流畅响应,列表渲染会被新输入打断、丢弃旧渲染、用最新关键词重排。

底层原理:transition 走低优先级 lane,被高优更新打断后 React 直接丢弃 WIP fiber 树,用新值重新开始——这就是 Fiber 双缓冲架构的用武之地。

**效果 2：渲染中途让出主线程,避免长任务**

并发模式下,React 把渲染拆成 5ms 左右的小切片(time slicing),每个切片结束都会调用 `shouldYield()` 检查浏览器是否需要绘制、响应输入。如果是,就主动让出主线程,下一帧再继续。

| 场景 | 同步渲染 | 并发渲染 |
| --- | --- | --- |
| 列表 1 万项渲染 | 主线程独占 200ms,期间无法响应输入 | 拆成 40 个切片,期间用户输入、动画都流畅 |
| 重型表单初始化 | 主线程占用 150ms,FCP 滞后 | 滚动、点击等交互不阻塞 |

**效果 3：自动批处理覆盖所有上下文**

并发模式下,无论从哪里调用 setState 都会自动合并:

```js
// React 17:同步事件外的 setState 不批处理
setTimeout(() => {
  setCount(c => c + 1)   // 立即重渲一次
  setFlag(f => !f)        // 又立即重渲一次
}, 0)

// React 18 createRoot:任何位置都自动批处理
setTimeout(() => {
  setCount(c => c + 1)
  setFlag(f => !f)
  // 只重渲一次 ✓
}, 0)
```

这把以往 setTimeout、Promise.then、原生事件回调里的性能问题统一收口,业务代码不用再手动 batch。

**效果 4:Suspense 能配合数据获取**

18 之前 Suspense 只能配合 `React.lazy` 做代码分割。开了并发模式后,Suspense 可以拦截组件内部抛出的 Promise,等数据就绪再恢复渲染:

```jsx
<Suspense fallback={<Spinner />}>
  <UserProfile userId={id} />  {/* 内部用 use() 读 Promise,自动挂起 */}
</Suspense>
```

配合 React 19 的 `use()` API,可以做到"在组件里直接读异步值,等待期间自动展示 fallback"——完全不用写 `if (loading) return <Spinner />` 这种样板。

**效果 5:服务端流式 SSR + 选择性 hydration**

18 的 `renderToPipeableStream` 配合 Suspense,可以让 SSR:

1. **流式输出 HTML**:首屏关键内容先返回,慢的部分用 `<!--$-->` 占位,数据就绪再追加。
2. **选择性 hydration**:某个 Suspense 边界数据没回来时,客户端可以先 hydrate 已经返回的部分,不用等整页 HTML 全部就绪。
3. **优先 hydrate 用户正在交互的区域**:用户点了某个组件,React 会优先 hydrate 它,即使它在 DOM 中靠后。

这是 Next.js App Router、Remix 这类框架的底层基础。

#### 三、开启并发渲染需要警惕的副作用

并发模式不是"无副作用升级",有三个坑要心里有数:

1. **StrictMode 下 effect 跑两次**:18 的开发环境会模拟"mount → unmount → mount"以暴露副作用清理问题。生产环境正常。如果第三方库没适配,可能出现重复初始化(参考前面的"场景十")。
2. **render 阶段必须无副作用**:并发模式下 render 可能被打断、丢弃后重做。在 render 里发请求、改全局变量、写 ref(不在 useEffect 里),会导致不可预测的行为。`UNSAFE_componentWillMount` 等老生命周期被废弃就是这个原因。
3. **外部 store 必须用 useSyncExternalStore**:旧版 redux/zustand 在并发模式下可能出现 tearing——A 组件读到旧 state、B 组件读到新 state。React 18 的 `useSyncExternalStore` 通过快照机制解决,所以状态库必须升级。

#### 四、什么时候用 `startTransition`?

不是所有更新都该标记为 transition,简单判断:

| 更新类型 | 优先级 | 例子 |
| --- | --- | --- |
| **必须立即响应** | 高(默认) | 输入框打字、按钮点击、checkbox 切换 |
| **可以慢一点** | 低(transition) | 搜索结果、过滤后的列表、Tab 内容、路由切换的目标页 |
| **完全可延迟** | 最低 | 性能埋点、非关键日志 |

简化原则:**用户视线焦点必须立即响应,焦点之外的内容用 transition**。

#### 五、小结

并发渲染要做**两层激活**——`createRoot` 让运行时具备并发能力,`startTransition` / `useDeferredValue` / `Suspense` 让具体更新真正走低优先级。开启后带来五项收益:**紧急更新优先、长任务拆切片、自动批处理、Suspense 数据获取、流式 SSR**。代价是**render 必须无副作用 + 外部 store 必须用 useSyncExternalStore**——这两条照着做,并发模式才能稳定发挥威力。

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
5. **配合 Server Actions 一致**：在服务端 React 项目里，`updateName` 可以直接是一个 server action 函数，前端调用方式接近本地函数，由编译器自动转 RPC。

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

新旧对比看下来，变化主线就是：**把"可能产生副作用、可能多次执行"的生命周期换成"纯函数 / 在 DOM 已就绪后执行"的生命周期**。

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

简化理解：18 之前主要只在合成事件里做批处理，18 之后批处理范围明显扩大了。

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

不少人会混淆两者，但它们解决的是不同问题。

| 维度 | SSR | Server Components |
| --- | --- | --- |
| 执行时机 | 每次请求时执行 | 构建时或请求时 |
| 输出 | HTML 字符串 | RSC 序列化协议 |
| 是否 hydrate | 是 | 否（Server Component 永不在客户端运行） |
| 能否带交互 | 完整组件 | 只能渲染，不能用 useState、事件 |
| 包体积影响 | 计入客户端 bundle | 不计入客户端 bundle |
| 直接访问后端资源 | 可以但要小心 | 设计上鼓励 |

简要区分：SSR 是"服务端预渲染 HTML"，Server Components 是"组件本身在服务端运行，从不发到客户端"。两者可以组合使用，Next.js App Router 就是典型实现。

### React 16-19 整体演进的主线是什么？

这一路的演进主线就是：**从同步递归渲染，逐步演进到可中断、可调度、可流式、可服务端化的并发架构**。

具体里程碑：

1. **15 及以前**：Stack Reconciler，递归 + 同步，无法中断、无优先级——后续所有改造的起点。
2. **16**：底层换成 Fiber（链表结构 + 可中断），但还是同步消费。
3. **16.3**：为异步渲染清理生命周期。
4. **16.6 / 16.8**：上层 API 升级（memo、lazy、Hooks），开发体验跟上。
5. **17**：基础设施改造（事件、JSX、effect 时机），为并发铺路。
6. **18**：并发渲染正式可用，自动批处理、Transitions、新 SSR。
7. **19**：把"客户端 + 服务端"统一起来（RSC、Actions），开发模型回归"写组件就是写 UI"。

理解这条主线，就能解释为什么 React 不断废弃旧 API、为什么生命周期改革、为什么强调不可变更新、为什么推 Suspense——它们都在为同一个方向服务。

## 补充题

### `useEffect` 和 `useLayoutEffect` 有什么区别？

1. `useEffect`
   - 在浏览器完成绘制后再执行
   - 更适合请求、订阅、日志等不阻塞渲染的副作用

2. `useLayoutEffect`
   - 在 DOM 更新后、浏览器绘制前执行
   - 更适合立即读取布局或同步修正布局的场景

不能滥用 `useLayoutEffect` 的原因是：它会阻塞浏览器绘制，如果里面逻辑过重，就会直接拖慢首屏和交互流畅度。

### React 中为什么 `key` 重要？

`key` 的作用不是“消除警告”，而是帮助 React 在 diff 过程中识别同层级元素身份。

如果 `key` 不稳定，会导致：

1. 本该复用的节点被错误销毁重建
2. 输入框状态错位
3. 列表动画和局部状态异常

所以 `key` 真正表达的是“这是谁”，不是“它排在第几个”。

## 进阶高频题与生态实战

### React Compiler 实战体验是怎样的？真的能替代 memo / useMemo / useCallback 吗？

React Compiler 19 跟着 RC 发了，但生产环境真用上的项目还不多。它就做一项工作：**编译期自动 memoize**——你不写 `useMemo`、`useCallback`、`React.memo`，编译器帮你插。

效果是真实的。看一段：

```jsx
// 你写的代码
function ProductList({ items, query }) {
  const filtered = items.filter(x => x.name.includes(query))
  const onClick = (id) => analytics.track('click', id)
  return filtered.map(item =>
    <ProductCard key={item.id} item={item} onClick={onClick} />
  )
}

// 编译器产出（伪代码，实际是优化版 IR）
function ProductList({ items, query }) {
  const $ = useMemoCache(3)

  let filtered
  if ($[0] !== items || $[1] !== query) {
    filtered = items.filter(x => x.name.includes(query))
    $[0] = items; $[1] = query; $[2] = filtered
  } else {
    filtered = $[2]
  }

  const onClick = useEvent(...)  // 类似 useEffectEvent 的处理
  return ...
}
```

它做对了几件事：

- **判断哪些值是「外部输入」**（props、state、context）、哪些是「派生值」。派生值自动 memo。
- **稳定函数引用**——回调函数自动包成「编译期 useEvent」，引用稳定，传给子组件的 memo 不会失效。
- **JSX 元素也会被 memo**——`<Card />` 这种没引用变化时直接复用上次的 ReactElement 对象。

但要清楚它**不是通用方案**：

- **只在「Rules of React」严格遵守时生效**——render 函数必须纯、不能在 render 里改 ref、不能在 render 里订阅东西。编译器能在编译期检测一部分违规（用了 `eslint-plugin-react-compiler`），但有些动态行为无法静态识别，只能跳过对应函数。
- **不优化跨组件通信**——Context value 引用变化导致全量重渲、Redux selector 返回新对象导致重渲，这些跟「函数内部 memo」无关，编译器救不了。
- **生产环境上手的门槛是「lint 通过率」**——大项目里第一次跑 `eslint-plugin-react-compiler` 能扫出几百条违规，得花时间清理。

Meta 自己 instagram.com 已经全面启用，效果是「能减少约 30% 的不必要重渲染」。中小项目收益没那么夸张，主要是「开发者不用再纠结要不要包 useMemo」本身就值。

### Server Components 在 Next.js App Router 里怎么用才避免问题？

RSC 在 Next.js 14/15 是 App Router 的默认模式，但不少项目会逐渐把组件都标成 `'use client'`，重新退回纯客户端模式。要用好 RSC，需要先想清楚边界划在哪。

**Server Component 能做什么**：

- 直接 `await fetch()` / `await db.query()` 拿数据，不用 loader、不用 getServerSideProps
- 拿密钥、调内部 API（凭证不会泄露到 client）
- 引入超大依赖（markdown 渲染器、syntax highlighter），不计入 client bundle

**Server Component 不能做什么**：

- 不能用 `useState`、`useEffect`、事件回调
- 不能用 Context（但 props 还是能传）
- 不能 import 浏览器才有的库

**`'use client'` 划在哪？记住一条**：尽可能往叶子推。如果一个组件需要交互，只把那个交互的小块标 client，外面套层 server。

```jsx
// app/products/page.tsx —— server component
export default async function ProductsPage() {
  const products = await db.products.findMany()
  return (
    <div>
      <h1>商品列表</h1>
      <ProductGrid products={products} />
      {/* 只有筛选器需要交互，单独标 client */}
      <FilterBar />
    </div>
  )
}

// app/products/FilterBar.tsx
'use client'
import { useState } from 'react'
export function FilterBar() {
  const [keyword, setKeyword] = useState('')
  return <input value={keyword} onChange={e => setKeyword(e.target.value)} />
}
```

**Server → Client 传 props 的限制**：序列化能力受限。函数不能传（除了 server action）、Date 可以传、Map/Set 可以、Class 实例不行、Symbol 不行。这条规则被违反时 Next.js 会在 dev 模式直接报错。

**和 Server Actions 的协作**：表单提交、删除、更新这些写操作走 server action，避免单独写 API 路由：

```jsx
// app/products/actions.ts
'use server'
export async function deleteProduct(id: string) {
  await db.products.delete({ where: { id } })
  revalidatePath('/products')
}

// 在 client component 里直接 import 调用
<form action={deleteProduct.bind(null, product.id)}>
  <button>删除</button>
</form>
```

调用一个 server action 等同于一次 RPC——Next.js 自动生成 endpoint、做序列化、做 CSRF 防护。

实战里几个问题：

- **不要在 server component 里直接给 client component 传不可序列化的对象**（比如 ORM 的实体类），传纯数据。
- **环境变量分两套**——`NEXT_PUBLIC_*` 才会暴露到 client，其他只有 server 能读。
- **revalidatePath / revalidateTag 是 RSC 数据更新的核心机制**，理解它跟 TanStack Query 的 invalidate 是同一个思路。

### TanStack Query / SWR 在大型项目里和 Zustand 怎么分工？

这俩在职责上是井水不犯河水的：

- **TanStack Query / SWR 管「服务端状态」**——接口数据、缓存、重试、loading/error 状态、并发去重、stale-while-revalidate
- **Zustand / Jotai 管「客户端状态」**——UI 状态、表单中间态、客户端独有的偏好

不少人犯的错就是用 Redux/Zustand 存接口数据：

```js
// ❌ 反模式：接口数据塞进 Zustand
const useUserStore = create((set) => ({
  user: null,
  loading: false,
  error: null,
  fetchUser: async (id) => {
    set({ loading: true })
    try {
      const user = await api.getUser(id)
      set({ user, loading: false })
    } catch (e) {
      set({ error: e, loading: false })
    }
  },
}))
```

这样你要手写 loading、error、缓存、重试、并发、过期、refetch、乐观更新、跨组件订阅……每个都是问题。

用 React Query 之后这些全没了：

```js
// ✅ 服务端状态全交给 React Query
function UserProfile({ id }) {
  const { data: user, isPending, error } = useQuery({
    queryKey: ['user', id],
    queryFn: () => api.getUser(id),
  })
  if (isPending) return <Spinner />
  if (error) return <Error />
  return <div>{user.name}</div>
}
```

它给你内置的能力：

- **自动去重**——同一个 queryKey 同时多次调用只发一个请求
- **缓存共享**——不同组件用同样的 queryKey，共用一份数据
- **stale-while-revalidate**——窗口聚焦自动刷新、保持显示旧数据避免闪烁
- **乐观更新**——mutation 之前先改 UI，失败回滚
- **请求竞态保护**——切换 query 时旧请求自动取消（v5 默认行为）

Zustand 留给实际的「客户端独有状态」：

```js
// 全局 UI 状态：侧边栏开合、当前主题
const useUIStore = create((set) => ({
  sidebarOpen: false,
  theme: 'light',
  toggleSidebar: () => set(s => ({ sidebarOpen: !s.sidebarOpen })),
}))

// 跨页面共享的「不需要序列化到 URL」的客户端状态
const useFilterStore = create((set) => ({
  filters: { category: 'all', sortBy: 'date' },
  setFilter: (key, value) => set(s => ({ filters: { ...s.filters, [key]: value }})),
}))
```

判断标准简单粗暴：**这数据是不是来自接口？是 → React Query。是不是只活在当前会话？是 → Zustand。是不是要在 URL 里反映？是 → URL search params + React Router/Next.js**。

### React 状态管理选型决策树

React 状态库满地跑：Redux Toolkit、Zustand、Jotai、Valtio、MobX、XState……新人容易迷路。给个能拍板的决策树：

**1. 状态从哪里来？**

- 接口数据 → TanStack Query / SWR（不要塞进状态库）
- URL 里能反映的（筛选条件、当前页、详情 id） → URL search params
- 实际的客户端状态 → 往下看

**2. 这个状态有多少地方用？**

- 只在一个组件里 → `useState` / `useReducer`
- 父子几层之间 → props / Context
- 跨页面、跨组件树 → 状态库

**3. 状态库选哪个？看更新模式：**

| 场景 | 推荐 | 理由 |
|------|------|------|
| 简单全局 state、低频更新 | **Zustand** | API 最简洁，1KB |
| 复杂派生、原子化、reactive 风格 | **Jotai** | 像 Recoil 的精神继承，atom 组合自然 |
| 「直接改对象就触发更新」、习惯 Vue 心智 | **Valtio** | Proxy 响应式 |
| 大型团队、有 Redux 历史、需要 time-travel | **Redux Toolkit** | RTK Query 一并解决服务端状态 |
| 复杂状态机（多步表单、wizard、订单流转） | **XState** | 状态机能力其他库都没有 |
| 实时协作、CRDT、撤销重做 | **Zustand + Yjs** / **Immer middleware** | 不可变 + 历史快照 |

**4. 几个具体业务里对应选型：**

- 购物车：Zustand（state 简单、要持久化、跨页面读）
- 复杂表单（多步、依赖校验）：React Hook Form + Zustand 存中间态
- 实时协作白板：Zustand + Y.js（CRDT 同步）
- 后台权限菜单：Context（一次性写入、低频）
- 主题/i18n：Context
- 富文本编辑器（含撤销重做）：内部用 immer + 状态机思路

**关键的一条**：90% 的状态都不该上全局库。`useState` 配合「状态提升到共同父级」能解决大部分场景。等到真的「props 透传 4 层以上、3 个以上不相邻组件读写」才考虑全局。

### React 19 的 Form Actions / useFormStatus / useOptimistic 怎么用？

19 改造了表单相关能力，从「手写 onSubmit + useState 跟踪 loading」变成「声明式 action + 自动跟踪状态」。一个典型表单：

```jsx
'use client'
import { useActionState, useOptimistic } from 'react'

async function addComment(prevState, formData) {
  const text = formData.get('text')
  try {
    await api.postComment({ text })
    return { success: true }
  } catch (e) {
    return { error: e.message }
  }
}

function CommentForm({ comments }) {
  // useActionState 替代了「手写 useState 跟踪 form state」
  const [state, formAction, isPending] = useActionState(addComment, null)

  // useOptimistic 实现「点了提交立刻显示，失败再回滚」
  const [optimisticComments, addOptimistic] = useOptimistic(
    comments,
    (state, newComment) => [...state, { text: newComment, pending: true }]
  )

  return (
    <form action={async (formData) => {
      addOptimistic(formData.get('text'))
      await formAction(formData)
    }}>
      <input name="text" disabled={isPending} />
      <SubmitButton />
      {state?.error && <p>{state.error}</p>}
      <ul>
        {optimisticComments.map((c, i) => (
          <li key={i} style={{ opacity: c.pending ? 0.5 : 1 }}>{c.text}</li>
        ))}
      </ul>
    </form>
  )
}

function SubmitButton() {
  const { pending } = useFormStatus()   // 自动拿到外层 form 的提交状态
  return <button disabled={pending}>{pending ? '提交中' : '提交'}</button>
}
```

要点：

- **`useActionState`** 把 reducer 思想搬到表单上——action 函数签名是 `(prevState, formData) => newState`，state 就是「上次 action 的结果」。loading 由 React 自动跟踪。
- **`useFormStatus`** 是个「从子组件读父 form 状态」的钩子，省掉了 prop drilling。SubmitButton 不用接收 isPending，自己读即可。
- **`useOptimistic`** 是「乐观更新」的官方实现——给它一份当前 state 和一个 reducer，它返回一份「界面上已经更新但还在 pending」的乐观 state。action 完成后会自动 reconcile 到真实 state。
- **`<form action={fn}>` 不再是 HTML 的 action 属性**——React 把它劫持成了「submit 时调这个函数」的简写，跟原生表单提交（GET/POST 跳转）已经是两回事。

这套 API 跟 server actions 配合最自然：把 `addComment` 改成 `'use server'` 函数，整个流程不用写一行 API 路由。

跟老写法比省了什么？省去了：

- 手写 `useState` 跟踪 loading、error、success
- 手写 onSubmit + preventDefault
- 手写「禁用按钮直到请求完成」的状态联动
- 手写乐观更新和回滚逻辑

代价：心智得重新建一遍，react-hook-form 这种库的优势（校验、复杂表单）暂时还是更强。简单 CRUD 表单用 Actions 体验好，复杂表单还是 react-hook-form + Zod。

### `use()` 是 Hook 还是不是 Hook？为什么 19 里这么有存在感？

`use()` 严格说不是 Hook，是 React 19 里的一个新原语——但用起来像 Hook，也能突破 Hook 的规则限制。具体表现：

```jsx
import { use } from 'react'

function UserProfile({ promise }) {
  // 普通 hook 不能在条件里调用,use 可以
  if (!promise) return <Empty />

  const user = use(promise)   // 这里"暂停"组件,等 promise resolve
  return <div>{user.name}</div>
}

// 父组件
function App() {
  const userPromise = fetchUser()
  return (
    <Suspense fallback={<Spinner />}>
      <UserProfile promise={userPromise} />
    </Suspense>
  )
}
```

`use(promise)` 的意思是「告诉 React：我现在需要这个 promise 的值。没 resolve 就把我挂起，让最近的 Suspense 边界显示 fallback；resolve 了把值给我，组件继续渲染」。

这跟之前要写多组 `useState + useEffect` 获取数据的方式完全不同。配合 Server Components / Server Actions，调用链是：

- Server Component 里直接 `await fetch()`（在服务端 await）
- Client Component 里没法 await，但可以 `use(promiseFromServer)`

第二个能力是 `use(context)` 替代 `useContext`，但能用在条件分支里：

```jsx
function Theme({ enabled }) {
  if (!enabled) return null
  const theme = use(ThemeContext)   // ✅ useContext 写在这里会报错,use 可以
  return <div style={{ background: theme.bg }} />
}
```

`use()` 不是通用异步解决方案。它有个**关键限制**：传给它的 promise **必须稳定**。每次 render 都创建新 promise 就会陷入死循环——promise pending → 挂起 → 重渲 → 新 promise → pending → 又挂起...

```jsx
// ❌ 每次 render 都是新 promise
function UserProfile({ id }) {
  const user = use(fetch(`/api/user/${id}`).then(r => r.json()))
  return <div>{user.name}</div>
}

// ✅ promise 由父组件或 cache 提供
const userCache = new Map()
function getUserPromise(id) {
  if (!userCache.has(id)) {
    userCache.set(id, fetch(`/api/user/${id}`).then(r => r.json()))
  }
  return userCache.get(id)
}

function UserProfile({ id }) {
  const user = use(getUserPromise(id))
  return <div>{user.name}</div>
}
```

更工程化的做法是用 TanStack Query 的 `useSuspenseQuery`——它内部就是 cache + use 的封装，自带请求去重和缓存策略。

### React 19 的「ref 当 prop 传」真的让 forwardRef 失业了吗？

19 之前，函数组件想接收 ref 必须用 `forwardRef` 包一层：

```jsx
const Input = React.forwardRef(function Input(props, ref) {
  return <input ref={ref} {...props} />
})

// 用：
const inputRef = useRef()
<Input ref={inputRef} />
```

19 把这层包装直接干掉了——ref 可以像普通 prop 那样传：

```jsx
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />
}

// 用法一样
<Input ref={inputRef} />
```

老的 `forwardRef` API 没废除，仍能跑，但官方明确说「未来版本会移除」。新代码不要再用 forwardRef 包了。

但有一个**老代码迁移的坑**：用 `forwardRef` 时 ref 是第二个参数，去掉之后 ref 在 props 解构里。如果之前组件库的 ref 类型签名用了 `ForwardedRef<T>`，迁移时类型要改：

```tsx
// 旧
type Props = { value: string }
const Input = forwardRef<HTMLInputElement, Props>((props, ref) => ...)

// 新
type Props = { value: string; ref?: React.Ref<HTMLInputElement> }
function Input({ ref, ...props }: Props) { ... }
```

组件库（Radix UI、Headless UI）开始陆续迁移到新写法，但旧版本兼容 React 18 的还得用 forwardRef。

附带一个 19 改进：**ref 回调函数现在可以返回 cleanup 函数**，跟 useEffect 一样：

```jsx
<div ref={(node) => {
  if (node) observer.observe(node)
  return () => observer.unobserve(node)   // ✨ 新增
}} />
```

以前要在外面单独写 useEffect 处理 cleanup，现在 ref callback 自己能管。

### React 跟 Web Components 集成的实战要点

不少团队会问「我们已经用 Lit 写了一套 Web Components，React 项目能不能直接用？」答案是能用，但有几个注意点。

**渲染层面**：React 18 之前对 Web Components 的支持是「能 mount，但有 bug」。19 起 React 团队专门做了适配——现在能正确处理 custom element 的 props、events、children。

```jsx
// 用法跟普通 element 一样
function App() {
  return (
    <my-button variant="primary" onClick={handleClick}>
      点击
    </my-button>
  )
}
```

但有几个细节：

**props 传递的类型限制**：HTML 原生属性都是字符串，传非字符串 prop 时 React 19 之前会强转成字符串：

```jsx
// React 18 老行为
<my-list items={[1, 2, 3]} />
// 实际 DOM 上 items="1,2,3"(被 toString 了)

// React 19 新行为:对 custom element 自动检测
// 是字符串属性就 setAttribute,是对象就走 property assignment
```

复杂数据传递还是更适合用 ref + 命令式：

```jsx
const ref = useRef()
useEffect(() => {
  ref.current.items = [1, 2, 3]
}, [items])
return <my-list ref={ref} />
```

**事件监听**：Web Components 派发的是 `CustomEvent`，React 之前的 `on{EventName}` 语法只认识标准 DOM 事件。19 起 React 会自动把 onMyEvent 这种 camelCase 转成 `addEventListener('myevent', ...)`。

```jsx
<my-modal onClose={handleClose} />   // ✅ 19 自动绑定
```

**SSR 兼容**：Web Components 必须在浏览器才能执行（依赖 `customElements` API）。在 SSR 项目里直接用 `<my-button>` 会报 server-side 错误。处理方式：

- 用 `<ClientOnly>` 包起来
- 或者用 declarative shadow DOM（新标准，SSR 友好但兼容性还在追）

**项目里要不要混用？**

如果团队已经有 Web Components 资产（设计系统、跨技术栈共享组件），React 19 集成体验已经够好。但如果是新项目，没必要为了「未来跨技术栈」预先用 WC——React 组件库（Radix、shadcn/ui）的生态远比 Lit 丰富。

### React 19 的 Asset Loading（preload / preinit / prefetchDNS）怎么用？

React 19 把资源加载相关的能力都做成了内置 API。以前要在 HTML 里写多条 `<link rel="preload">`，现在可以在组件里声明，React 自动 hoist 到 `<head>`：

```jsx
import { preload, preinit, prefetchDNS, preconnect } from 'react-dom'

function Page() {
  // 关键图片预加载
  preload('/hero.webp', { as: 'image', fetchPriority: 'high' })

  // 字体预加载
  preload('/fonts/inter.woff2', {
    as: 'font',
    type: 'font/woff2',
    crossOrigin: 'anonymous',
  })

  // 关键脚本预初始化(下载 + 解析,但不执行)
  preinit('https://cdn.example.com/analytics.js', { as: 'script' })

  // DNS 预解析
  prefetchDNS('https://api.example.com')

  // 提前建立 TCP/TLS 连接
  preconnect('https://cdn.example.com')

  return <Hero />
}
```

跟手写 `<link>` 比这几个 API 的好处：

- **去重**：同一个 url 多次调用只生成一个 link 标签
- **跨组件**：子组件里声明的 preload 会被自动 hoist 到 head
- **服务端友好**：SSR 时直接渲染到 HTML 的 head 里，浏览器拿到 HTML 立刻能下载

实际用法是「在路由级 component 顶部声明本路由需要的关键资源」：

```jsx
function ProductPage({ id }) {
  // 这个页面的核心图片立刻 preload
  preload(`https://cdn.example.com/product/${id}.webp`, { as: 'image', fetchPriority: 'high' })
  // 用户大概率会点的下一步,prefetch DNS
  prefetchDNS('https://checkout.example.com')

  return <ProductDetail id={id} />
}
```

LCP 优化效果明显——主图加载从「等 JS 跑完 → 等图片下载」变成「HTML 到达 → 立刻并行下载图片」。

### Concurrent 模式下 render 必须纯，具体有哪些问题？

「render 阶段必须无副作用」是 React 并发渲染里的重要约束。Concurrent 模式下 render 可能被打断、丢弃、重做，如果在 render 阶段产生副作用，就容易出现问题。看几个常见场景：

**问题一：render 里改 ref**

```jsx
function Bad() {
  const counterRef = useRef(0)
  counterRef.current++   // ❌ render 里改 ref
  return <div>{counterRef.current}</div>
}
```

在 StrictMode 下 React 会刻意 render 两次（暴露副作用），counterRef 直接变 2。Concurrent 模式下如果这次 render 被丢弃重做，counter 还会再加一次。改法：把 ref 修改放进 effect。

**问题二：render 里发请求**

```jsx
function Bad({ id }) {
  fetch(`/api/user/${id}`).then(setUser)   // ❌ 每次 render 都发
  return <div>...</div>
}
```

不只是浪费请求，还可能出现「请求顺序错乱后用旧响应覆盖新响应」。改法：用 useEffect 或者 TanStack Query 这种声明式数据库。

**问题三：render 里给全局变量赋值**

```jsx
let pageTitle = ''   // 模块级变量

function Bad({ title }) {
  pageTitle = title   // ❌
  return <h1>{title}</h1>
}
```

如果两个组件同时被 render（Concurrent 模式有可能并发处理两棵 fiber 树），pageTitle 会乱。改法：把跨组件状态放 Context 或者全局 store。

**问题四：render 里读 / 改 DOM**

```jsx
function Bad() {
  const width = document.getElementById('foo').offsetWidth   // ❌
  return <div style={{ width }}>...</div>
}
```

render 时 DOM 还没更新（或者根本没 mount），读到的可能是旧值。改法：用 useLayoutEffect 在 mount 后读，或者用 useSyncExternalStore 接外部 store。

**问题五：在条件分支或循环里用 hook**

```jsx
function Bad({ enabled }) {
  if (enabled) {
    const [state, setState] = useState(0)   // ❌
  }
}
```

不算 render 副作用，但属于「render 纯函数」的违规——hook 依赖固定调用顺序。

判断标准简单：**把组件 render 调用 100 次，结果应该完全一样、不影响外部任何东西**。做不到就是有副作用。

React 19 的 `eslint-plugin-react-compiler` 会扫描这些违规，编译器遇到不纯的组件会跳过 memoization。所以「render 纯」这条规则未来会越来越被强制。

### React + IndexedDB / Dexie 在大型应用里怎么集成？

复杂前端应用经常需要本地持久化大量数据：离线编辑、PWA 缓存、富文本草稿、IM 消息历史等。localStorage 5MB 上限根本不够，必须上 IndexedDB。

裸 IndexedDB API 使用成本高（事务模型、回调地狱、版本管理）。Dexie 是事实标准的封装：

```js
import Dexie from 'dexie'

class AppDB extends Dexie {
  constructor() {
    super('AppDB')
    this.version(1).stores({
      messages: '++id, conversationId, sentAt',
      drafts: '++id, &localKey',     // & 表示唯一索引
    })
  }
}

export const db = new AppDB()
```

跟 React 配合的关键是 `dexie-react-hooks`：

```jsx
import { useLiveQuery } from 'dexie-react-hooks'

function MessageList({ conversationId }) {
  const messages = useLiveQuery(
    () => db.messages
      .where('conversationId').equals(conversationId)
      .reverse()
      .limit(50)
      .toArray(),
    [conversationId]
  )

  if (!messages) return <Spinner />
  return messages.map(m => <Message key={m.id} {...m} />)
}
```

`useLiveQuery` 是「响应式查询」——数据库变化时自动重新查询并触发组件重渲。底层用 IndexedDB 的 onversionchange 跟一个内部的发布订阅。

实战中常用这几种模式：

**模式 1：服务端 → IndexedDB → UI**

```js
// 拉数据后写 IndexedDB,UI 直接读 IndexedDB
async function syncMessages(conversationId) {
  const remote = await api.getMessages(conversationId)
  await db.messages.bulkPut(remote)
  // UI 自动通过 useLiveQuery 看到新数据
}
```

**模式 2：乐观写入 + 后台同步**

```js
async function sendMessage(text, conversationId) {
  // 立刻写本地,UI 立即显示
  const localId = await db.messages.add({
    text, conversationId,
    sentAt: Date.now(),
    pending: true,
  })

  try {
    const serverMsg = await api.sendMessage(text)
    // 同步成功,更新本地状态
    await db.messages.update(localId, { ...serverMsg, pending: false })
  } catch {
    // 失败标记为待重试
    await db.messages.update(localId, { pending: false, failed: true })
  }
}
```

**模式 3：PWA 离线**

配合 Service Worker：在线时所有请求都同步进 IndexedDB，离线时 SW 拦截请求从 IndexedDB 读。这是 Notion、Linear 这类「重客户端 + 离线优先」应用的标准架构。

**注意**：

- IndexedDB 容量受浏览器策略影响（Chrome 通常 ~60% 磁盘、Safari 1GB），定期清理过期数据
- 跨 origin 不共享，PWA 和官网各自一份
- 跟 React Query 配合：用 React Query 做服务端状态，IndexedDB 做持久化层，两者通过 `queryClient.setQueryData` 互通

### React 跟 Astro / Qwik 这种新生代框架怎么对比？

24-25 年「React 是不是该被替代」的讨论越来越多。Astro、Qwik、Solid 这些新生代框架确实在某些维度上对 React 是降维打击。但 React 仍占绝对市场——理解差异在哪、什么场景该考虑切，比单纯讨论「谁更好」有意义。

**Astro**：内容优先的框架，主打「Islands Architecture」（岛屿架构）。

- 页面默认是纯静态 HTML，零 JS
- 需要交互的部分用 React / Vue / Svelte 等任意框架写，按需 hydrate（「岛屿」）
- 单页能混用多种框架——比如导航用 React、评论框用 Vue

```astro
---
// 服务端代码,跑一次后扔了
import { fetchPosts } from '../lib/api'
const posts = await fetchPosts()
---

<html>
  <body>
    {posts.map(p => <article>{p.title}</article>)}

    <!-- 这一段是 React 组件,只有它会 hydrate -->
    <CommentSection client:visible />
  </body>
</html>
```

适合：博客、文档站、营销页、电商列表页——内容为主、交互区域小、SEO 重要。

不适合：业务后台、复杂 SPA、实时协作类应用。

**Qwik**：主打「Resumability」（可恢复性）——彻底解决「JS hydration 浪费」问题。

传统 SSR 框架（Next.js / Nuxt）的痛点：服务端把组件渲染成 HTML 发给浏览器，浏览器还要再把整个 JS bundle 下载、解析、执行一遍才能交互（hydration）。大型 App 的 JS 包好几 MB，hydration 慢得很。

Qwik 的思路：组件状态序列化进 HTML，浏览器只在用户真正交互时才加载对应组件的 JS。一个 50 个组件的页面，用户只点了一个按钮，只下载那个按钮的 JS（KB 级），不是整个 bundle（MB 级）。

理论上首屏交互速度比任何现有方案都快。

**Solid**：前面提过，「无 VDOM 细粒度响应式」的方向。

**怎么选**：

| 场景 | 推荐 |
|------|------|
| 内容站 / 博客 / 文档 | Astro |
| 大型 SPA / 业务后台 | React + Next.js |
| 极致首屏性能（电商） | Qwik |
| 性能敏感、团队接受新技术 | Solid |
| 跟 React 生态深度集成 | React 仍然是较稳的 |

实际跑下来：

- Astro 24 年起在内容型项目里增长很快——Stripe、Cloudflare、IBM 文档站都迁过去了
- Qwik 仍在「早期采用者」阶段，生态没起来，生产案例不多
- Solid 在游戏 UI、动画密集场景有优势，普通业务用得少
- React 仍占 80%+ 市场，新框架更多是「补位」而非「替代」

这个话题需要讲清楚每个框架的核心创新点（Astro = Islands、Qwik = Resumability、Solid = 无 VDOM 响应式）以及各自适合的场景。「React 是不是过时了」这类问题不适合简单二选一。

### React 自定义 Hook 的设计原则

自定义 Hook 的质量差异很大，设计时可以遵循下面几条实践原则。

**原则 1：单一职责**

一个 Hook 只承担一个职责。看到 useUser 里既请求接口、又管理本地状态、还做权限校验，就该拆。

```js
// ❌ 一个 hook 干 5 件事
function useUser() {
  const [user, setUser] = useState()
  const [loading, setLoading] = useState()
  const [permissions, setPermissions] = useState()
  // fetch user、check permission、save to storage、refresh on focus...
}

// ✅ 拆成职责清晰的小 hook
function useUser() { /* 只管获取 user */ }
function useUserPermissions(userId) { /* 派生权限 */ }
function usePersistedUser() { /* 持久化逻辑 */ }
```

**原则 2:返回值用对象,不用元组**

```js
// ❌ 调用方记不住顺序
const [data, loading, error, refetch, mutate] = useQuery()

// ✅ 调用方按需解构
const { data, loading, error, refetch } = useQuery()
```

React 内置的 `useState` 用元组是因为它通常需要重命名（`const [count, setCount]`、`const [name, setName]`），自定义 Hook 一般不需要重命名，对象更自然。

**原则 3：尊重 Rules of Hooks**

- 不在条件 / 循环里调用 Hook
- 自定义 Hook 名字以 `use` 开头
- Hook 之间的依赖关系要清晰

ESLint 的 `eslint-plugin-react-hooks` 必装。

**原则 4：依赖项稳定**

Hook 内部返回的函数 / 对象要稳定引用，否则消费方的 `useEffect` 会反复触发：

```js
// ❌ 每次返回新函数,消费方 useEffect deps 变了一直跑
function useDebouncedValue(value, delay) {
  const debounced = debounce((v) => setState(v), delay)   // 每次 render 新建
  // ...
}

// ✅ 用 useCallback / useMemo 稳定引用
function useDebouncedValue(value, delay) {
  const debounced = useMemo(() => debounce(setState, delay), [delay])
  // ...
}
```

**原则 5：副作用要可清理**

```js
function useWindowSize() {
  const [size, setSize] = useState({ w: 0, h: 0 })

  useEffect(() => {
    const handler = () => setSize({ w: innerWidth, h: innerHeight })
    handler()   // 初始化
    window.addEventListener('resize', handler)
    return () => window.removeEventListener('resize', handler)   // ✅ 清理
  }, [])

  return size
}
```

订阅、定时器、WebSocket、事件监听都必须配对清理，否则就是内存泄漏。

**原则 6：用现成的别自己写**

`react-use`、`ahooks`、`@vueuse/core`（Vue）这些库提供了几百个常用 hook（防抖、节流、复制到剪贴板、本地存储、网络状态、媒体查询、IntersectionObserver、ResizeObserver……）。能直接用就别自己写。

**好 Hook 的标志**：

- 在 README 里能简短说清「这个 hook 做什么」
- 没有意外的副作用——只做名字暗示的事
- 测试用例好写（Hook 独立可测）
- 接 SSR 时不会爆（不直接读 window / document）

业务里见到「越来越复杂、依赖项一长串、改一个地方好几个地方坏」的 Hook，主要是没遵守这几条。

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

1. **`getDerivedStateFromError`**：在渲染阶段调用，返回的对象会被 merge 到 state。它是纯函数（不能在里面发副作用），唯一的目的是把"出错了"这个事实落到 state 上，这样可以下次 render 走降级分支。
2. **`componentDidCatch`**：在 commit 阶段调用，可以执行副作用（上报、打日志），第二个参数 `info.componentStack` 包含组件调用栈，对线上排错重要。

能捕获：渲染阶段、生命周期、子组件树里的同步错误。
不能捕获：事件回调里的错误、`setTimeout`/Promise 等异步错误、SSR 错误、Error Boundary 自身错误——这些需要 `window.onerror`、`unhandledrejection`、try/catch 等其他手段兜底。
