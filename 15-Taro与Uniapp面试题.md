[toc]



## 一、Taro 基础认知

### 说说你对 Taro 的理解？优缺点是什么？

Taro 是由京东凹凸实验室开源的多端统一开发框架，支持使用 React/Vue/Nerv 等框架来开发微信/京东/百度/支付宝/字节跳动/QQ 小程序/H5/React Native 等应用。

它的特点：

1. 遵循 React/Vue 语法规范，使用 JSX/Vue 模板编写代码。
2. 支持多端编译，一套代码可以生成多个平台的应用。
3. 提供了丰富的组件库和 API 适配层。
4. 支持 TypeScript，提供完善的类型定义。

优点：

1. 技术栈统一，React/Vue 开发者上手快。
2. 代码复用率高，降低多端开发成本。
3. 社区活跃，生态丰富，有京东团队维护。
4. 支持多种框架，灵活度高。

缺点：

1. 编译后的代码体积相对较大。
2. 部分平台特性需要条件编译处理。
3. 调试链路相对复杂，需要理解编译机制。
4. 性能不如原生小程序开发。

### Taro 的实现原理是什么？

Taro 的核心原理是**编译时适配 + 运行时适配**的组合方案。

**编译时适配：**

1. **语法转换**：将 React/Vue 语法转换为各平台支持的语法。
2. **组件映射**：将 Taro 组件映射为目标平台的原生组件。
3. **API 适配**：将 Taro API 转换为目标平台的 API 调用。

**运行时适配：**

1. **Taro 1/2 版本**：采用重编译、轻运行时的方案，编译时做大量转换。
2. **Taro 3 版本**：采用重运行时方案，实现了一套基于 Web 标准的运行时框架。

**Taro 3 的核心架构：**

```
React/Vue 代码
    ↓
Taro Runtime (模拟 DOM/BOM)
    ↓
小程序的 setData
    ↓
小程序视图层渲染
```

一句话总结：Taro 通过编译时转换和运行时适配，让开发者可以使用 React/Vue 语法开发多端应用。

### Taro 1/2 和 Taro 3 有什么区别？

**Taro 1/2 的方案：**

1. **重编译、轻运行时**：在编译时做大量的语法转换和优化。
2. **模板化**：将 JSX 编译为小程序的模板语法。
3. **限制较多**：对 React 的写法有较多限制，不支持动态组件等特性。

**Taro 3 的方案：**

1. **重运行时、轻编译**：实现了一套完整的 DOM/BOM API 模拟层。
2. **更接近 Web 开发**：支持几乎所有的 React/Vue 特性。
3. **灵活性更高**：支持动态组件、Hooks、任意 JSX 写法等。
4. **性能优化**：通过虚拟 DOM diff 和批量 setData 优化性能。

**核心差异：**

```javascript
// Taro 1/2：编译时限制
class Index extends Component {
  render() {
    // ❌ 不支持：动态组件
    const Comp = this.state.type === 'A' ? CompA : CompB
    return <Comp />
  }
}

// Taro 3：运行时支持
function Index() {
  const [type, setType] = useState('A')
  // ✅ 支持：动态组件、Hooks
  const Comp = type === 'A' ? CompA : CompB
  return <Comp />
}
```

### Taro 支持哪些平台？如何配置多端编译？

**支持的平台：**

1. 微信小程序
2. 支付宝小程序
3. 百度小程序
4. 字节跳动小程序
5. QQ 小程序
6. 京东小程序
7. H5
8. React Native
9. 快应用
10. 鸿蒙

**配置多端编译：**

```javascript
// config/index.js
const config = {
  projectName: 'myApp',
  // 编译配置
  mini: {
    // 小程序配置
  },
  h5: {
    // H5 配置
  }
}

// package.json 配置编译命令
{
  "scripts": {
    "build:weapp": "taro build --type weapp",
    "build:alipay": "taro build --type alipay",
    "build:h5": "taro build --type h5",
    "build:rn": "taro build --type rn"
  }
}
```

### Taro 的生命周期有哪些？

Taro 的生命周期分为**应用生命周期**和**页面生命周期**。

**应用生命周期（App.jsx）：**

```javascript
import { Component } from 'react'

class App extends Component {
  // 程序启动时触发，全局只触发一次
  onLaunch(options) {
    console.log('App Launch', options)
  }
  
  // 程序显示时触发
  onShow(options) {
    console.log('App Show', options)
  }
  
  // 程序隐藏时触发
  onHide() {
    console.log('App Hide')
  }
  
  // 程序错误时触发
  onError(error) {
    console.log('App Error', error)
  }
  
  render() {
    return this.props.children
  }
}

export default App
```

**页面生命周期：**

```javascript
import { Component } from 'react'
import { View } from '@tarojs/components'

class Index extends Component {
  // 页面加载时触发，只触发一次
  onLoad(options) {
    console.log('Page Load', options)
  }
  
  // 页面显示时触发，每次显示都会触发
  onShow() {
    console.log('Page Show')
  }
  
  // 页面初次渲染完成时触发，只触发一次
  onReady() {
    console.log('Page Ready')
  }
  
  // 页面隐藏时触发
  onHide() {
    console.log('Page Hide')
  }
  
  // 页面卸载时触发
  onUnload() {
    console.log('Page Unload')
  }
  
  // 页面下拉刷新
  onPullDownRefresh() {
    console.log('Pull Down Refresh')
  }
  
  // 页面上拉触底
  onReachBottom() {
    console.log('Reach Bottom')
  }
  
  // 页面滚动
  onPageScroll(e) {
    console.log('Page Scroll', e.scrollTop)
  }
  
  // 用户点击右上角分享
  onShareAppMessage() {
    return {
      title: '分享标题',
      path: '/pages/index/index'
    }
  }
  
  render() {
    return <View>页面内容</View>
  }
}

export default Index
```

### Taro 的编译过程是怎样的？涉及哪些核心工具？

Taro 的编译过程本质上是一个 **源码到源码（Source to Source）** 的转换器，借助 Babel 工具链完成。

**编译三大核心阶段：**

```
源代码 (React/Vue)
    ↓
1. Parse（解析）：通过 @babel/parser 将代码解析成 AST（抽象语法树）
    ↓
2. Transform（转换）：通过 @babel/traverse 遍历 AST，应用各种转换规则
    ↓
3. Generate（生成）：通过 @babel/generator 将新 AST 生成目标代码
```

**核心 Babel 工具链：**

| 工具 | 作用 |
|------|------|
| `@babel/parser`（旧名 babylon） | 将源代码解析为 AST，支持 JSX 语法 |
| `@babel/traverse` | 遍历 AST 节点，提供 Path 路径对象操作节点 |
| `@babel/types` | 用于构造、验证、变换 AST 节点的工具库 |
| `@babel/template` | 通过模板字符串生成代码片段 |
| `@babel/generator` | 将 AST 生成目标代码字符串 |

**JSX 转换为小程序模板的核心流程（以 wx:if 为例）：**

```javascript
// 源代码（JSX）
{isShow && <View>Hello</View>}

// 解析后的 AST 节点
LogicalExpression {
  operator: '&&',
  left: Identifier(isShow),
  right: JSXElement(View)
}

// 转换规则：将 LogicalExpression 转为 wx:if 指令
// 生成目标代码（WXML）
<view wx:if="{{isShow}}">Hello</view>
```

**Taro 1/2 和 Taro 3 在编译策略上的差异：**

1. **Taro 1/2（编译型）**：在编译阶段完成大量 AST 转换，将 JSX 转为字符串模板。问题：写法限制多，不支持动态 JSX、展开运算符等。
2. **Taro 3（运行时型）**：编译时不操作开发者代码的 AST，只对入口文件、配置等做处理，因此**编译速度大幅提高**。运行时通过模拟 BOM/DOM 让 React/Vue 直接运行。

### Taro 中使用 Hooks 有哪些注意事项？

Taro 中的 Hooks 分为两类：**React 框架自带的 Hooks**（如 `useState`、`useEffect`、`useCallback` 等）从 `react` 引入，**Taro 专有 Hooks**（如 `useRouter`、`useDidShow`、`useReachBottom` 等页面生命周期相关 Hook）从 `@tarojs/taro` 引入。二者不能混用引入来源，否则会报错或失效。

```javascript
// React 框架自带的 Hooks - 从 react 引入
import { useState, useEffect, useCallback } from 'react'

// Taro 专有 Hooks - 从 @tarojs/taro 引入
import { 
  useRouter,        // 等同于 getCurrentInstance().router
  useReady,         // 等同于 onReady
  useDidShow,       // 等同于 onShow
  useDidHide,       // 等同于 onHide
  usePullDownRefresh,
  useReachBottom,
  usePageScroll,
  useResize,
  useShareAppMessage,
  useTabItemTap,
  useShareTimeline
} from '@tarojs/taro'
```

#### **踩坑点 1：页面级 Hook 不能在子组件中使用**

`useReachBottom`、`usePullDownRefresh`、`usePageScroll` 等 Taro 专有 Hooks 本质上绑定的是小程序页面的生命周期，只在**页面根组件**中注册才会生效。如果放在子组件中，由于子组件并不对应小程序的 Page 实例，Hook 的回调永远不会被触发。

```javascript
// ❌ 错误：子组件中使用 useReachBottom，不会触发
function ChildComponent() {
  useReachBottom(() => {
    console.log('不会触发')
  })
  return <View>子组件</View>
}

// ✅ 正确：必须在页面组件根层级使用
function PageIndex() {
  useReachBottom(() => {
    console.log('触底加载')
  })
  return <ChildComponent />
}
```

#### **踩坑点 2：闭包陷阱 - 推荐函数式更新**

与 React 中 `useEffect` 的闭包问题类似，Taro 的页面级 Hooks（如 `useReachBottom`）的回调函数也会捕获创建时的 state 值。如果直接引用 state 变量，拿到的永远是注册那一刻的旧值，导致更新不生效。解决办法是使用 `setState` 的**函数式更新**形式，让 React 自动传入最新 state。

```javascript
// ❌ 错误：捕获旧 state，page 一直是 0
const [page, setPage] = useState(0)
useReachBottom(() => {
  setPage(page + 1) // 闭包捕获了初始的 page=0
})

// ✅ 正确：使用函数式更新避免闭包问题
useReachBottom(() => {
  setPage((prev) => prev + 1)
})
```

#### **踩坑点 3：DOM 操作必须在 useReady 中**

在小程序中，页面的初次渲染完成由 `onReady` 生命周期标识。Taro 对应提供了 `useReady` Hook，在此时才能安全地通过 `createSelectorQuery` 获取节点信息或执行 DOM 相关操作。而在 `useEffect` 中执行时，小程序视图层可能还未完成渲染，查询结果会返回 `null`。

```javascript
// ❌ 错误：useEffect 中无法获取小程序节点
useEffect(() => {
  const query = Taro.createSelectorQuery()
  query.select('.container').boundingClientRect().exec(res => {
    console.log(res) // null
  })
}, [])

// ✅ 正确：useReady 对应 onReady，可获取节点
useReady(() => {
  const query = Taro.createSelectorQuery()
  query.select('.container').boundingClientRect().exec(res => {
    console.log(res) // 正常获取
  })
})
```

#### **踩坑点 4：useShareAppMessage 需要配置启用**

从 Taro 3.0.3 起，出于性能考虑，分享功能默认关闭。必须在页面的配置文件（`*.config.ts`）中显式将 `enableShareAppMessage` 设为 `true`，`useShareAppMessage` 的回调才会被注册。同理 `useShareTimeline` 也需要 `enableShareTimeline: true`。遗漏配置会导致分享按钮不显示或分享行为不触发。

```javascript
// 从 Taro 3.0.3 开始，必须在页面配置中显式启用
// pages/index/index.config.js
export default {
  enableShareAppMessage: true,
  enableShareTimeline: true
}

// 然后才能在组件中使用
useShareAppMessage(() => ({
  title: '分享标题',
  path: '/pages/index/index'
}))
```

### Taro 3 的 setData 优化机制是怎样的？

Taro 3 是重运行时框架，每次更新都需要把数据通过 `setData` 从逻辑层（JS 线程）传到视图层（渲染线程），这是一个跨线程通信过程，传输的数据量直接影响性能，因此 setData 的优化至关重要。Taro 3 在框架层面内置了多种优化策略来减少 setData 的调用次数和数据体积。

**1. Batch 批量更新机制**

小程序的 `setData` 是一次昂贵的跨线程调用。如果在同一次事件循环中多次调用 `setState`，Taro 会将这些更新收集并合并，最终只触发一次 `setData`，从而避免频繁的线程通信开销。这与 React 的批量更新思路一致。

```javascript
// 这三次更新会被合并为一次 setData
this.setState({ a: 1 })
this.setState({ b: 2 })
this.setState({ c: 3 })

// 实际只触发一次：setData({ a: 1, b: 2, c: 3 })
```

**2. 自动 Diff 优化**

即使经过批量合并，如果每次都把整个页面数据全量传给视图层，数据量仍然很大。Taro 在调用小程序原生 `setData` 之前，会将当前虚拟 DOM 数据与上一次的数据做浅比较 diff，只提取发生变化的字段进行增量更新，开发者无需手动拆分 setData。例如页面中有 100 个字段但只改了 1 个，最终只会 `setData({ thatOneField: newValue })`。

**3. baseLevel 配置（解决模板递归问题）**

微信等小程序的 WXML 模板不支持递归调用自身，因此 Taro 采用"循环展开"的方式：将模板重复写 N 层（`baseLevel` 次）来支持嵌套渲染。当 DOM 嵌套层级超过 `baseLevel` 后，Taro 会将超出的部分放在一个原生自定义组件中渲染，这个自定义组件内部又拥有新的 `baseLevel` 层模板空间。降低 `baseLevel` 可以减少模板体积（编译产物更小），但会增加自定义组件的数量。

```javascript
// config/index.js
module.exports = {
  mini: {
    baseLevel: 16  // 默认 16，可调整为 8 或 4 提升性能
  }
}
```

**baseLevel 调整的副作用：**
- flex 布局跨原生自定义组件会失效（最大问题）——因为自定义组件形成了新的渲染边界，flex 容器无法穿透组件边界影响子元素的布局
- SelectorQuery.select 跨自定义组件需要使用 `>>>` 选择器：`.parent >>> .child`——小程序的节点查询同样受到组件边界限制
- 是全局配置，影响所有页面——无法按页面粒度调整

**4. CustomWrapper 局部更新优化（推荐）**

`baseLevel` 是全局配置，调低后所有页面都会受影响，不够灵活。`CustomWrapper` 是 Taro 提供的更精细的优化方案：它会在小程序端创建一个原生自定义组件作为渲染边界，被其包裹的模块更新时只会触发该组件内部的 `setData`，不会把外层或兄弟节点的数据一起传入，从而实现局部更新、减少传输量。典型场景是页面中只有某个模块频繁更新（如倒计时、轮播图），其他部分相对静态。

```javascript
import { View, Text, CustomWrapper } from '@tarojs/components'

function Page() {
  return (
    <View>
      <Text>不需要频繁更新的部分</Text>
      
      {/* 使用 CustomWrapper 包裹频繁更新的模块 */}
      <CustomWrapper>
        <GoodsList />  {/* 商品列表只更新自己，不影响外部 */}
      </CustomWrapper>
      
      <CustomWrapper>
        <Comments />   {/* 评论模块独立更新 */}
      </CustomWrapper>
    </View>
  )
}
```

**注意事项：**
- 不要过度使用 CustomWrapper，每个 CustomWrapper 都会创建一个原生自定义组件实例，带来额外的内存和通信开销
- 一般包裹遇到性能问题的复杂模块即可，按需使用
- 注意自定义组件的两个限制（flex 布局跨界、选择器跨界），与 baseLevel 降低后的副作用一致

**5. 写法陷阱：保持对象引用**

Taro 的自动 Diff 依赖浅比较来判断数据是否变化。如果在 JSX 中直接内联写对象或数组字面量，每次渲染都会产生新的引用，导致 Diff 结果为"数据已变化"，从而触发不必要的 setData。解决办法是通过 `useState` 或 `useMemo` 保持引用稳定，让浅比较判断出"未变化"并跳过更新。

```javascript
// ❌ 错误：每次渲染都生成新对象，引用变化触发 setData
function MapComponent() {
  return <Map markers={[{ id: 1, lat: 30 }]} />  // markers 每次都是新对象
}

// ✅ 正确：使用 useState 或 useMemo 保持引用
function MapComponent() {
  const [markers] = useState([{ id: 1, lat: 30 }])  // 引用稳定
  return <Map markers={markers} />
}
```

**6. 避免传递非标准属性**

Taro 会将组件的 props 通过 `setData` 传递到视图层。如果给小程序原生组件传了它不认识的自定义属性（如 `customProp`），这些属性依然会被序列化到 setData 的数据中，造成传输冗余。小程序约定使用 `data-*` 属性来传递自定义数据，这类属性有专门的处理通道，不会增加 setData 的数据负担。

```javascript
// ❌ 错误：自定义属性会被一并 setData，造成数据冗余
<Text customProp="value" mySize="20">文本</Text>

// ✅ 正确：标准属性以外的数据放到 data-* 中
<Text data-custom="value" data-size="20">文本</Text>
```

### Taro 的 Prerender 预渲染是什么？如何使用？

Prerender（预渲染）是 Taro CLI 提供的提升小程序首屏渲染速度的技术，原理类似服务端渲染（SSR）。

**解决的问题：**

Taro 3 在初始化时，会把框架的虚拟 DOM 树通过 setData 传递到视图层，初始数据较大时会产生白屏。Prerender 在编译期间将首屏静态 wxml 直接产出，先于业务逻辑渲染。

**配置方式：**

```javascript
// config/index.js
const config = {
  mini: {
    prerender: {
      // 匹配规则
      match: 'pages/shop/**',                    // 所有 pages/shop 下的页面参与预渲染
      include: ['pages/any/way/index'],          // 显式包含
      exclude: ['pages/shop/index/index'],       // 显式排除
      
      // 高级配置
      console: true,                              // 显示日志
      mock: { isLogin: true },                    // mock 一些状态
      transformData: (data, config) => data,     // 转换数据
      transformXML: (data, config, xml) => xml   // 转换 XML
    }
  }
}
```

**注意事项：**

1. **包体积增加**：以空间换时间，预渲染越多，包越大
2. **不会执行生命周期**：componentDidMount/mounted 不会执行，需要的话提前到 getDerivedStateFromProps/created
3. **Hydrate 之前不响应交互**：用户操作要等到真实 DOM 挂载后才能响应
4. **可访问 PRERENDER 全局变量**：值为 true 时表示当前在预渲染阶段

```javascript
// 区分预渲染时期的逻辑
if (typeof PRERENDER !== 'undefined' && PRERENDER) {
  // 预渲染期间执行的逻辑
  return <View>初始内容</View>
} else {
  // 正常运行时
  return <DynamicContent />
}
```

5. **跳过特定组件预渲染**：通过 `disablePrerender` 属性

```javascript
<MyComponent disablePrerender={true} />
```

### Taro 中 Vue3 开发有哪些写法限制？

Taro 支持 Vue3 开发，但因为遵循小程序规范，有一些与 Web 端不同的写法限制。

**1. 组件名和属性遵循 kebab-case**

```vue
<!-- ❌ 错误：使用 PascalCase 或 camelCase -->
<View></View>
<Button openType="share"></Button>

<!-- ✅ 正确：使用 kebab-case -->
<view></view>
<button open-type="share"></button>
```

**2. Boolean 属性需要显式绑定**

```vue
<!-- ❌ 错误：简写形式不支持 -->
<button disabled></button>

<!-- ✅ 正确：显式绑定 true -->
<button :disabled="true"></button>
```

**3. 不支持 `<style scoped>`**

```vue
<!-- ❌ 错误：小程序不支持 scoped 样式 -->
<style scoped>
.container { padding: 20rpx; }
</style>

<!-- ✅ 正确：使用 cssModules 或 BEM 命名 -->
<style module>
.container { padding: 20rpx; }
</style>
```

**4. transition 组件需要显式指定动画时长**

```vue
<!-- ❌ 问题：小程序无 getComputedStyle，transition 嗅探失败 -->
<transition name="fade">
  <view v-if="show">内容</view>
</transition>

<!-- ✅ 解决：style 显式指定 transitionDuration -->
<transition name="fade">
  <view v-if="show" style="transition-duration: 300ms;">内容</view>
</transition>
```

**5. transition-group 不可用**

由于小程序访问元素位置是异步 API，无法使用内置的 transition-group 组件，需要自行实现动画或使用第三方组件。

**6. Ref 获取的是虚拟 DOM 不是真实节点**

```vue
<template>
  <view ref="container">内容</view>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Taro from '@tarojs/taro'

const container = ref(null)

onMounted(async () => {
  // ❌ container.value 是 Taro 虚拟 DOM，没有尺寸信息
  console.log(container.value.clientHeight) // undefined
  
  // ✅ 正确：等待渲染完成，使用小程序 API
  await Taro.nextTick()
  const query = Taro.createSelectorQuery()
  query.select('.container').boundingClientRect().exec(res => {
    console.log(res[0].height) // 真实高度
  })
})
</script>
```

**7. v-html 需要额外配置**

小程序端使用 v-html 时需要做特殊处理，参考 Taro 文档《渲染 HTML》章节，否则只能渲染纯文本。

### Taro 的 VirtualList 虚拟列表如何使用？

针对长列表场景，Taro 内置了 VirtualList 组件，只渲染可视区域的元素，提升渲染性能。

```javascript
import VirtualList from '@tarojs/components/virtual-list'
import { View } from '@tarojs/components'

function Row({ index, data }) {
  return (
    <View style={{ height: '100px' }}>
      Row {index}: {data[index].name}
    </View>
  )
}

function LongList() {
  // 模拟 10000 条数据
  const data = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`
  }))
  
  return (
    <VirtualList
      height={500}        // 容器高度
      width="100%"        // 容器宽度
      itemData={data}     // 数据源
      itemCount={data.length}  // 数据总数
      itemSize={100}      // 每行高度
      item={Row}          // 行组件
    />
  )
}
```

**关键特性：**
- **按需渲染**：仅渲染可视区域 + 缓冲区的元素，DOM 节点数量稳定
- **支持横向滚动**：通过 `layout="horizontal"` 切换
- **可变高度**：通过 `itemSize` 传入函数支持不同高度

## 二、Uniapp 基础认知

### 说说你对 Uniapp 的理解？优缺点是什么？

Uniapp 是 DCloud 推出的使用 Vue.js 开发所有前端应用的框架，开发者编写一套代码，可发布到 iOS、Android、Web（响应式）、以及各种小程序（微信/支付宝/百度/头条/QQ/钉钉/淘宝）、快应用等多个平台。

它的特点：

1. 基于 Vue.js 语法，Vue 开发者零学习成本。
2. 内置丰富的组件和 API，开箱即用。
3. 条件编译机制完善，方便处理平台差异。
4. 拥有插件市场，生态丰富。

优点：

1. Vue 技术栈，学习成本低，上手快。
2. 官方维护，文档完善，社区活跃。
3. 插件市场丰富，有大量现成的组件和模板。
4. 性能优化做得好，接近原生小程序。
5. HBuilderX 编辑器集成度高，开发体验好。

缺点：

1. 主要支持 Vue，不支持 React。
2. 部分高级特性需要付费（如云打包、离线打包）。
3. 对 Vue 3 的支持相对较晚。
4. 某些平台特性需要条件编译处理。

### Uniapp 的实现原理是什么？

Uniapp 的核心原理是**编译时 + 运行时**的混合方案。

**编译时处理：**

1. **条件编译**：根据目标平台编译不同的代码。
2. **语法转换**：将 Vue 语法转换为各平台支持的语法。
3. **组件映射**：将 uni 组件映射为目标平台的原生组件。

**运行时处理：**

1. **统一 API**：提供统一的 API 层，运行时调用对应平台的原生 API。
2. **事件系统**：统一的事件处理机制。
3. **数据绑定**：基于 Vue 的响应式数据绑定。

**架构图：**

```
Vue 代码
    ↓
Uniapp 编译器
    ↓
├─ 小程序：WXML + WXSS + JS
├─ H5：标准 Web
└─ App：Weex/原生渲染
```

**不同平台的渲染方式：**

1. **小程序平台**：编译为小程序的 WXML、WXSS、JS。
2. **H5 平台**：编译为标准的 HTML、CSS、JS。
3. **App 平台**：使用 Weex 或原生渲染引擎。

### Uniapp 的条件编译是什么？如何使用？

条件编译是 Uniapp 处理多端差异的核心机制，可以在编译时根据目标平台包含或排除特定代码。

**基本语法：**

```javascript
// #ifdef 平台名
// 仅在指定平台编译
// #endif

// #ifndef 平台名
// 除了指定平台外都编译
// #endif
```

**在 JS 中使用：**

```javascript
export default {
  data() {
    return {
      title: ''
    }
  },
  onLoad() {
    // #ifdef MP-WEIXIN
    // 仅在微信小程序中编译
    this.title = '微信小程序'
    wx.showShareMenu()
    // #endif
    
    // #ifdef MP-ALIPAY
    // 仅在支付宝小程序中编译
    this.title = '支付宝小程序'
    my.showSharePanel()
    // #endif
    
    // #ifdef H5
    // 仅在 H5 中编译
    this.title = 'H5'
    document.title = 'H5 页面'
    // #endif
    
    // #ifdef APP-PLUS
    // 仅在 App 中编译
    this.title = 'App'
    plus.navigator.setStatusBarBackground('#000000')
    // #endif
    
    // #ifdef MP-WEIXIN || MP-ALIPAY
    // 微信或支付宝小程序中编译
    console.log('小程序平台')
    // #endif
  }
}
```

**在 CSS 中使用：**

```css
/* #ifdef MP-WEIXIN */
/* 仅在微信小程序中生效 */
.container {
  padding: 20rpx;
}
/* #endif */

/* #ifdef H5 */
/* 仅在 H5 中生效 */
.container {
  padding: 10px;
}
/* #endif */

/* #ifndef H5 */
/* 除了 H5 外都生效 */
.container {
  background: #f5f5f5;
}
/* #endif */
```

**在模板中使用：**

```vue
<template>
  <view class="container">
    <!-- #ifdef MP-WEIXIN -->
    <button open-type="share">微信分享</button>
    <!-- #endif -->
    
    <!-- #ifdef MP-ALIPAY -->
    <button @click="alipayShare">支付宝分享</button>
    <!-- #endif -->
    
    <!-- #ifdef H5 -->
    <button @click="h5Share">H5 分享</button>
    <!-- #endif -->
  </view>
</template>
```

**常用平台标识：**

| 平台 | 标识 |
|------|------|
| 微信小程序 | MP-WEIXIN |
| 支付宝小程序 | MP-ALIPAY |
| 百度小程序 | MP-BAIDU |
| 字节跳动小程序 | MP-TOUTIAO |
| QQ 小程序 | MP-QQ |
| H5 | H5 |
| App | APP-PLUS |
| 所有小程序 | MP |

### Uniapp 的生命周期有哪些？

Uniapp 的生命周期分为**应用生命周期**、**页面生命周期**和**组件生命周期**。

**应用生命周期（App.vue）：**

```javascript
export default {
  // 应用初始化完成时触发，全局只触发一次
  onLaunch(options) {
    console.log('App Launch', options)
  },
  
  // 应用显示时触发
  onShow(options) {
    console.log('App Show', options)
  },
  
  // 应用隐藏时触发
  onHide() {
    console.log('App Hide')
  },
  
  // 应用错误时触发
  onError(err) {
    console.log('App Error', err)
  }
}
```

**页面生命周期：**

```javascript
export default {
  // 页面加载时触发，参数为上个页面传递的数据
  onLoad(options) {
    console.log('Page Load', options)
  },
  
  // 页面显示时触发
  onShow() {
    console.log('Page Show')
  },
  
  // 页面初次渲染完成时触发
  onReady() {
    console.log('Page Ready')
  },
  
  // 页面隐藏时触发
  onHide() {
    console.log('Page Hide')
  },
  
  // 页面卸载时触发
  onUnload() {
    console.log('Page Unload')
  },
  
  // 页面下拉刷新
  onPullDownRefresh() {
    console.log('Pull Down Refresh')
    // 停止下拉刷新
    setTimeout(() => {
      uni.stopPullDownRefresh()
    }, 1000)
  },
  
  // 页面上拉触底
  onReachBottom() {
    console.log('Reach Bottom')
  },
  
  // 页面滚动
  onPageScroll(e) {
    console.log('Page Scroll', e.scrollTop)
  },
  
  // 用户点击右上角分享
  onShareAppMessage(res) {
    return {
      title: '分享标题',
      path: '/pages/index/index'
    }
  },
  
  // 页面尺寸变化时触发
  onResize(res) {
    console.log('Page Resize', res.size)
  },
  
  // Tab 点击时触发
  onTabItemTap(item) {
    console.log('Tab Item Tap', item)
  }
}
```

**组件生命周期（Vue 标准生命周期）：**

```javascript
export default {
  beforeCreate() {},
  created() {},
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {}
}
```

### Uniapp 的双线程架构是什么？

Uniapp 在非 H5 端运行时，采用**逻辑层 / 视图层分离**的双线程架构，这与小程序原生架构一致。

**架构图：**

```
┌─────────────────────────┐
│   逻辑层（Service）      │  独立 jscore 运行
│   - 业务逻辑、数据处理     │  无浏览器 API
│   - Vue 实例、状态管理    │
└─────────────┬───────────┘
              │ 通信桥（有损耗）
┌─────────────▼───────────┐
│   视图层（View）          │  WebView 渲染（H5/小程序/app-vue）
│   - 渲染页面 UI          │  原生渲染（app-nvue/Weex）
│   - 处理用户交互          │
└─────────────────────────┘
```

**关键特点：**

1. **逻辑层运行在独立的 jscore 中**：不依赖本机 webview，没有浏览器兼容问题，可以在 Android 4.4 上跑 ES6 代码
2. **逻辑层无浏览器 API**：无法访问 `window`、`document`、`navigator`、`localStorage` 等
3. **通信存在损耗**：滚动、跟手操作、setData 都需要跨线程通信，会产生延迟
4. **不同平台视图层不同**：H5/小程序/app-vue 是 webview 渲染；app-nvue 是基于 weex 改造的原生渲染

**通信折损的典型表现：**

```javascript
// ❌ 在 onPageScroll 中频繁更新数据，会引起严重卡顿
onPageScroll(e) {
  this.scrollTop = e.scrollTop  // 高频跨线程通信
  this.calculateHeader()
}

// ✅ 解决方案：使用 wxs/renderjs 在视图层处理
// 或使用节流降低通信频率
import { throttle } from 'lodash'
onPageScroll: throttle(function(e) {
  this.scrollTop = e.scrollTop
}, 100)
```

### 什么是 wxs / renderjs / bindingx？分别用于什么场景？

这三者都是为了**降低逻辑层和视图层通信折损**而设计的解决方案。

**1. WXS（视图层脚本）**

WXS 是运行在视图层的专属 JS，可以在视图层直接处理事件和数据，避免跨线程通信。

```html
<!-- 微信小程序 / app-vue -->
<wxs module="utils">
  function formatPrice(price) {
    return '￥' + (price / 100).toFixed(2)
  }
  module.exports = { formatPrice: formatPrice }
</wxs>

<view>{{utils.formatPrice(price)}}</view>
```

**WXS 的限制：**
- 只能用 ES5 语法
- 不能调用小程序 API
- 微信小程序限制较多

**2. RenderJS（WXS 增强版，仅 app-vue 和 H5）**

RenderJS 是 uni-app 提供的更强大的视图层脚本，支持完整 ES6+ 语法，可以操作 DOM 和使用 web 库（如 echarts）。

```vue
<template>
  <view>
    <view :prop="data" :change:prop="echart.update" id="chart"></view>
  </view>
</template>

<script>
// 逻辑层
export default {
  data() {
    return {
      data: { value: 100 }
    }
  }
}
</script>

<script module="echart" lang="renderjs">
// 视图层 - 可以直接访问 DOM
import * as echarts from 'echarts'

export default {
  mounted() {
    this.chart = echarts.init(document.getElementById('chart'))
  },
  methods: {
    update(newValue) {
      // 在视图层直接操作 DOM 和 echarts
      this.chart.setOption({
        series: [{ data: [newValue.value] }]
      })
    },
    // 通过 $ownerInstance 调用逻辑层方法
    callLogic() {
      this.$ownerInstance.callMethod('logicMethod')
    }
  }
}
</script>
```

**RenderJS 的优势：**
- 支持完整 ES6+ 和 npm 库
- 可直接访问 DOM、window
- 适用于高性能 canvas 动画、第三方图表库等
- App 端 + H5 端通用

**3. BindingX（仅 app-nvue）**

BindingX 是 weex 提供的原生层表达式机制，可以在 JS 中传递一个表达式给原生层，由原生层解析并直接操作视图层。

```javascript
// app-nvue 中使用 bindingx 实现跟手动画
import { bindingx } from '@nvue/bindingx'

export default {
  mounted() {
    bindingx.bind({
      eventType: 'pan',                          // 监听拖动手势
      anchor: this.$refs.handler.ref,           // 触发元素
      props: [{
        element: this.$refs.target.ref,        // 受影响元素
        property: 'transform.translateX',
        expression: 'x + 0'                    // 跟随手指 x 移动
      }]
    })
  }
}
```

**三者对比：**

| 方案 | 平台支持 | 主要场景 |
|------|---------|---------|
| WXS | 微信小程序、app-vue | 视图层数据格式化、简单事件处理 |
| RenderJS | app-vue、H5 | 高性能动画、canvas、第三方库（echarts） |
| BindingX | app-nvue | 原生渲染下的手势跟手动画 |

### nvue 和 vue 页面有什么区别？什么场景使用 nvue？

**nvue（native vue）** 是 uni-app 的一种原生渲染方案，基于 weex 改造而来。

**1. 渲染机制差异**

```
vue 页面：Vue 代码 → 编译 → WebView 渲染（H5/小程序/app-vue）
                            ↓
                        系统浏览器内核（性能受限于 WebView）

nvue 页面：Vue 代码 → 编译 → 原生组件树（仅 app-nvue）
                            ↓
                        Weex/原生渲染引擎（接近原生性能）
```

**2. nvue 的优势**

- **接近原生的渲染性能**：尤其在低端 Android 设备上差距明显
- **App 启动速度提升**：首页用 nvue + manifest 配置 fast 模式，启动可控制在 1 秒
- **更好的 map/video 体验**：原生组件无层级问题
- **包体积更小**：纯 nvue 项目可减少约 2M

**3. nvue 的限制**

```css
/* ❌ nvue 只支持 flex 布局，不支持其他布局方式 */
.container {
  display: block;          /* 不支持 */
  display: grid;           /* 不支持 */
  float: left;             /* 不支持 */
  position: fixed;         /* 部分支持 */
}

/* ✅ nvue 默认 flex，方向是 column（不是 row）*/
.container {
  flex-direction: column;
  align-items: center;
}
```

**其他限制：**
- 只有 `<text>` 标签可以设置字体大小、字体颜色
- 不支持百分比布局
- 只能使用 class 选择器（不支持 #id、>、~ 等选择器）
- 不支持媒体查询（动态横竖屏适配困难）
- canvas 性能差（推荐用 vue 页面 + renderjs）

**4. 什么场景使用 nvue？**

| 推荐使用 nvue | 不推荐使用 nvue |
|--------------|----------------|
| 深度使用 map 组件 | 需要 canvas 动画 |
| 深度使用 video/直播推流 | 需要复杂 CSS 布局 |
| 长列表（list 组件有资源回收） | 需要媒体查询 |
| 对启动速度要求高 | 需要复杂样式（如背景图） |
| 需要原生跟手动画 | 横竖屏动态切换 |

**5. nvue 与 vue 页面通讯**

```javascript
// nvue 中发送消息到 vue 页面
uni.postMessage({
  data: { type: 'update', value: 'hello' }  // 只能是 JSON
})

// vue 页面中接收 nvue 消息
onUniNViewMessage(e) {
  console.log('收到 nvue 消息:', e.data)
}
```

### Uniapp 中如何集成 Pinia？有哪些注意事项？

Pinia 是 Vue3 官方推荐的状态管理库，是 Vuex 的替代方案，对 TypeScript 支持更友好。

**1. 在 Uniapp 中集成 Pinia 的标准做法**

```javascript
// main.js（注意：不能像普通 Vue3 项目那样直接 use）
import App from './App'
import { createSSRApp } from 'vue'
import * as Pinia from 'pinia'

export function createApp() {
  const app = createSSRApp(App)
  
  app.use(Pinia.createPinia())
  
  return {
    app,
    Pinia  // 注意：必须将 Pinia 返回，这是 uniapp 与普通 Vue3 项目的关键区别
  }
}
```

**2. 定义 Store（Composition API 风格）**

```javascript
// stores/user.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
  // state
  const userInfo = ref(null)
  const token = ref('')
  
  // getters
  const isLogin = computed(() => !!token.value)
  
  // actions
  const login = async (username, password) => {
    const res = await api.login(username, password)
    userInfo.value = res.userInfo
    token.value = res.token
    
    // 持久化
    uni.setStorageSync('token', res.token)
  }
  
  const logout = () => {
    userInfo.value = null
    token.value = ''
    uni.removeStorageSync('token')
  }
  
  return {
    userInfo,
    token,
    isLogin,
    login,
    logout
  }
})
```

**3. 在组件中使用**

```vue
<template>
  <view>
    <view v-if="userStore.isLogin">欢迎，{{ userStore.userInfo.name }}</view>
    <button v-else @click="handleLogin">登录</button>
  </view>
</template>

<script setup>
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

const handleLogin = () => {
  userStore.login('admin', '123456')
}
</script>
```

**4. 持久化方案**

```javascript
// 使用 pinia-plugin-persist-uni 插件
// main.js
import { createPersistedState } from 'pinia-plugin-persistedstate'

const pinia = Pinia.createPinia()
pinia.use(createPersistedState({
  storage: {
    getItem: uni.getStorageSync,
    setItem: uni.setStorageSync
  }
}))

// store 中开启持久化
export const useUserStore = defineStore('user', {
  state: () => ({ userInfo: null }),
  persist: true  // 开启持久化
})
```

**5. 常见踩坑点**

**问题 1：版本不兼容**

```bash
# 错误：pinia@3.0.1 需要 vue@^2.7.0 或 ^3.5.11
# 但 uniapp 的 vue 版本为 3.4.21

# 解决：锁定 Pinia 至兼容版本
npm install pinia@2.1.7
```

**问题 2：vue-demi 报错**

```
Uncaught SyntaxError: The requested module does not provide an export named 'hasInjectionContext'
```

解决方案：升级 Pinia 或锁定 vue-demi 版本。

**问题 3：忘记返回 Pinia 实例**

```javascript
// ❌ 错误：忘记返回 Pinia，会导致状态丢失
export function createApp() {
  const app = createSSRApp(App)
  app.use(createPinia())
  return { app }  // 缺少 Pinia
}

// ✅ 正确
export function createApp() {
  const app = createSSRApp(App)
  const Pinia = createPinia()
  app.use(Pinia)
  return { app, Pinia }
}
```

### Uniapp 如何适配鸿蒙（HarmonyOS）？

Uniapp 自 HBuilderX 4.24+ 开始支持适配鸿蒙系统，分为传统 uni-app 和 uni-app x 两套方案。

**1. 环境要求**

| 工具 | 版本要求 |
|------|---------|
| HBuilderX | 4.24+（推荐 4.31+） |
| DevEco-Studio | 5.0.3.400+（推荐 5.0.3.800+） |
| 鸿蒙系统 | API 12 以上（uni-app x 需 API 14+） |

**2. 鸿蒙开发限制**

- **只支持 Vue3**，不支持 Vue2
- **不支持 plus**（plus 是 5+ App 的能力）
- **支持 nvue**
- 鸿蒙工程目录路径不能包含中文和特殊字符

**3. manifest.json 配置**

```json
{
  "uni-app-x": {
    "harmony": {
      "minAPIVersion": "12",
      "targetAPIVersion": "12",
      "abilities": [],
      "permissions": [
        "ohos.permission.KEEP_BACKGROUND_RUNNING",
        "ohos.permission.NOTIFICATION_CONTROLLER",
        "ohos.permission.VIBRATE",
        "ohos.permission.INTERNET",
        "ohos.permission.GET_NETWORK_INFO"
      ]
    }
  }
}
```

**4. 条件编译适配**

```javascript
// 鸿蒙特有逻辑
// #ifdef APP-HARMONY
import { router } from '@kit.ArkUI'
router.pushUrl({ url: 'pages/Detail', params: { id: 1 } })
// #endif

// #ifndef APP-HARMONY
uni.navigateTo({ url: '/pages/detail/index?id=1' })
// #endif
```

```css
/* 鸿蒙安全区域适配 */
.container {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
}

/* 折叠屏适配 */
@media screen and (min-width: 600px) {
  .container {
    max-width: 600px;
    margin: 0 auto;
  }
}
```

**5. 鸿蒙原生 API 调用示例**

```javascript
// 监听系统主题变化
import { configuration } from '@kit.AbilityKit'

const config = configuration.getConfiguration()
const isDark = config.colorMode === configuration.ColorMode.COLOR_MODE_DARK

if (isDark) {
  // 应用深色主题
}
```

**6. 已知问题和踩坑点**

- **付费插件暂不支持**：直接使用会报错，需购买 uni-app x 源码版
- **uts 插件中的 component**：参与 uni-app x 鸿蒙编译会报错，需复制到项目根目录
- **Web 引擎被移除**：鸿蒙 NEXT 上 uni-app 依赖的 Web 引擎已移除，性能不如原生 ArkTS
- **建议**：新项目优先 ArkTS，存量项目逐步重构关键模块

### 什么是 uni-app x？与传统 uni-app 有何区别？

uni-app x 是 DCloud 推出的下一代 uni-app，采用 **UTS + UVue** 的全新架构，实现真正的原生跨端开发。

**1. 核心组件**

| 组件 | 作用 |
|------|------|
| **UTS** | uni type script，TypeScript 超集，编译为各平台原生代码 |
| **UVue** | 基于 UTS 的、兼容 Vue 语法的、跨平台的原生渲染引擎 |
| **uni 组件和 API** | 跨平台一致的组件和 API |

**2. UTS 编译产物**

```
UTS 代码
    ↓ 编译
├─ Android：编译为 Kotlin 原生代码
├─ iOS：编译为 Swift 原生代码
├─ 鸿蒙 Next：编译为 ArkTS 原生代码
└─ Web/小程序：编译为 JavaScript
```

**3. 与传统 uni-app 的关键区别**

| 对比项 | 传统 uni-app | uni-app x |
|--------|------------|-----------|
| 开发语言 | JS/TS | UTS（编译为原生语言） |
| Android 性能 | JS 引擎跨语言通信 | 原生 Kotlin，无通信桥 |
| iOS 性能 | JSCore 通信 | 原生 Swift |
| 渲染引擎 | WebView/Weex | UVue 原生渲染 |
| 包体积 | 较大（带 JS 引擎） | 更小（无运行时） |
| 跨语言通信 | 有 | 无（编译为原生） |
| Vue 版本 | Vue2/Vue3 | 仅 Vue3 |
| 学习成本 | Vue + JS | Vue + UTS（接近 TS） |

**4. UTS 代码示例**

```typescript
// UTS 写法（看起来像 TypeScript，但编译为原生代码）
class UserService {
  private apiUrl: string = 'https://api.example.com'
  
  async getUserInfo(id: number): Promise<UserInfo> {
    const response = await uni.request({
      url: `${this.apiUrl}/user/${id}`,
      method: 'GET'
    })
    return response.data as UserInfo
  }
}

// 在 UVue 组件中使用
<template>
  <view>{{ user?.name }}</view>
</template>

<script setup lang="uts">
import { UserService } from '@/services/user.uts'

const user = ref<UserInfo | null>(null)
const service = new UserService()

onMounted(async () => {
  user.value = await service.getUserInfo(1)
})
</script>
```

**5. 性能优势**

uni-app x 在 Android 上的工程被整体编译为 Kotlin 代码，本质上是**换了 vue 写法的原生 kotlin 应用**，性能与原生 kotlin 一致。

```
传统 uni-app 在 Android：
  JS 代码 → JS 引擎 → 跨语言桥 → 原生 UI（性能受桥限制）

uni-app x 在 Android：
  UTS 代码 → 编译为 Kotlin → 直接运行（性能 = 原生）
```

**6. 适用场景**

- 对性能要求极高的应用（游戏、视频处理等）
- 需要深度调用原生能力的应用
- 鸿蒙 Next 适配（uni-app x 是官方推荐方案）
- 新项目优先选择 uni-app x

## 三、Taro vs Uniapp 对比

### Taro 和 Uniapp 有什么区别？如何选择？

**技术栈差异：**

| 对比项 | Taro | Uniapp |
|--------|------|--------|
| 支持框架 | React/Vue/Nerv | Vue（主要） |
| 语法风格 | JSX/Vue 模板 | Vue 模板 |
| 开发团队 | 京东凹凸实验室 | DCloud |
| 开源协议 | MIT | Apache 2.0 |

**架构差异：**

1. **Taro**：
   - Taro 3 采用重运行时方案
   - 实现了完整的 DOM/BOM API 模拟
   - 更接近 Web 标准开发

2. **Uniapp**：
   - 编译时 + 运行时混合方案
   - 更贴近小程序原生开发
   - 性能优化更激进

**生态对比：**

1. **Taro**：
   - 可以使用 React/Vue 生态的大部分库
   - 社区活跃，有京东背书
   - 适合 React 技术栈团队

2. **Uniapp**：
   - 官方插件市场丰富
   - 有完善的 UI 组件库（uView、uni-ui）
   - 适合 Vue 技术栈团队

**性能对比：**

1. **Taro**：
   - Taro 3 运行时开销相对较大
   - 适合复杂交互的应用
   - 编译后代码体积较大

2. **Uniapp**：
   - 编译后代码更接近原生
   - 性能接近原生小程序
   - 包体积相对较小

**选择建议：**

1. **选择 Taro 的场景**：
   - 团队使用 React 技术栈
   - 需要使用 React 生态的库
   - 对 JSX 语法有偏好
   - 需要更灵活的组件化方案

2. **选择 Uniapp 的场景**：
   - 团队使用 Vue 技术栈
   - 需要快速开发，使用现成模板
   - 对性能要求较高
   - 需要丰富的插件市场支持

**代码对比示例：**

```javascript
// Taro (React)
import { View, Text } from '@tarojs/components'
import { useState } from 'react'

function Index() {
  const [count, setCount] = useState(0)
  
  return (
    <View>
      <Text>{count}</Text>
      <Button onClick={() => setCount(count + 1)}>+1</Button>
    </View>
  )
}

// Uniapp (Vue)
<template>
  <view>
    <text>{{ count }}</text>
    <button @click="count++">+1</button>
  </view>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>
```

## 四、跨端开发实践

### 如何处理多端差异？

**1. 使用条件编译（推荐）**

Taro 条件编译：

```javascript
import Taro from '@tarojs/taro'

// 使用 process.env.TARO_ENV 判断平台
if (process.env.TARO_ENV === 'weapp') {
  // 微信小程序
  Taro.showShareMenu()
} else if (process.env.TARO_ENV === 'h5') {
  // H5
  document.title = 'H5 页面'
}
```

Uniapp 条件编译：

```javascript
// #ifdef MP-WEIXIN
wx.showShareMenu()
// #endif

// #ifdef H5
document.title = 'H5 页面'
// #endif
```

**2. 封装统一的 API 层**

```javascript
// utils/platform.js
export const platform = {
  // 显示提示
  showToast(title) {
    // #ifdef MP-WEIXIN
    wx.showToast({ title, icon: 'none' })
    // #endif
    
    // #ifdef MP-ALIPAY
    my.showToast({ content: title })
    // #endif
    
    // #ifdef H5
    alert(title)
    // #endif
  },
  
  // 获取系统信息
  getSystemInfo() {
    return new Promise((resolve) => {
      // #ifdef MP-WEIXIN
      wx.getSystemInfo({ success: resolve })
      // #endif
      
      // #ifdef MP-ALIPAY
      my.getSystemInfo({ success: resolve })
      // #endif
      
      // #ifdef H5
      resolve({
        platform: 'h5',
        screenWidth: window.screen.width,
        screenHeight: window.screen.height
      })
      // #endif
    })
  }
}
```

**3. 使用平台特定的组件**

```vue
<template>
  <view>
    <!-- 微信小程序特有组件 -->
    <!-- #ifdef MP-WEIXIN -->
    <open-data type="userAvatarUrl"></open-data>
    <!-- #endif -->
    
    <!-- H5 特有组件 -->
    <!-- #ifdef H5 -->
    <div class="web-only">H5 专属内容</div>
    <!-- #endif -->
    
    <!-- 通用组件 -->
    <view class="common">通用内容</view>
  </view>
</template>
```

**4. 样式差异处理**

```css
/* 通用样式 */
.container {
  padding: 20rpx;
}

/* #ifdef MP-WEIXIN */
/* 微信小程序特有样式 */
.container {
  background: #f5f5f5;
}
/* #endif */

/* #ifdef H5 */
/* H5 特有样式 */
.container {
  max-width: 750px;
  margin: 0 auto;
}
/* #endif */
```

### Taro/Uniapp 如何实现路由跳转？

**Taro 路由跳转：**

```javascript
import Taro from '@tarojs/taro'

// 1. 保留当前页面，跳转到应用内的某个页面
Taro.navigateTo({
  url: '/pages/detail/index?id=1'
})

// 2. 关闭当前页面，跳转到应用内的某个页面
Taro.redirectTo({
  url: '/pages/detail/index?id=1'
})

// 3. 跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面
Taro.switchTab({
  url: '/pages/index/index'
})

// 4. 关闭所有页面，打开到应用内的某个页面
Taro.reLaunch({
  url: '/pages/index/index'
})

// 5. 返回上一页面或多级页面
Taro.navigateBack({
  delta: 1 // 返回的页面数
})

// 6. 获取路由参数
import { useRouter } from '@tarojs/taro'

function Detail() {
  const router = useRouter()
  console.log(router.params.id) // 获取 id 参数
}
```

**Uniapp 路由跳转：**

```javascript
// 1. 保留当前页面，跳转到应用内的某个页面
uni.navigateTo({
  url: '/pages/detail/index?id=1'
})

// 2. 关闭当前页面，跳转到应用内的某个页面
uni.redirectTo({
  url: '/pages/detail/index?id=1'
})

// 3. 跳转到 tabBar 页面
uni.switchTab({
  url: '/pages/index/index'
})

// 4. 关闭所有页面，打开到应用内的某个页面
uni.reLaunch({
  url: '/pages/index/index'
})

// 5. 返回上一页面或多级页面
uni.navigateBack({
  delta: 1
})

// 6. 获取路由参数
export default {
  onLoad(options) {
    console.log(options.id) // 获取 id 参数
  }
}
```

**路由传参的最佳实践：**

```javascript
// 1. 简单参数：直接拼接 URL
Taro.navigateTo({
  url: `/pages/detail/index?id=1&name=test`
})

// 2. 复杂参数：使用 encodeURIComponent
const data = { id: 1, list: [1, 2, 3] }
Taro.navigateTo({
  url: `/pages/detail/index?data=${encodeURIComponent(JSON.stringify(data))}`
})

// 接收参数
onLoad(options) {
  const data = JSON.parse(decodeURIComponent(options.data))
}

// 3. 大量数据：使用全局状态管理或本地存储
// 发送页面
Taro.setStorageSync('detailData', data)
Taro.navigateTo({ url: '/pages/detail/index' })

// 接收页面
onLoad() {
  const data = Taro.getStorageSync('detailData')
}
```

### 如何在 Taro/Uniapp 中使用第三方 UI 组件库？

**Taro 常用 UI 组件库：**

1. **Taro UI**（官方推荐）

```bash
# 安装
npm install taro-ui

# 使用
import { AtButton } from 'taro-ui'
import 'taro-ui/dist/style/index.scss'

function Index() {
  return <AtButton type="primary">按钮</AtButton>
}
```

2. **NutUI**（京东风格）

```bash
npm install @nutui/nutui-react-taro

import { Button } from '@nutui/nutui-react-taro'
```

**Uniapp 常用 UI 组件库：**

1. **uView UI**（最流行）

```bash
# 安装
npm install uview-ui

# main.js 引入
import uView from 'uview-ui'
Vue.use(uView)

# 使用
<template>
  <u-button type="primary">按钮</u-button>
</template>
```

2. **uni-ui**（官方组件库）

```bash
# 通过 HBuilderX 导入插件
# 或通过 npm 安装
npm install @dcloudio/uni-ui

# 使用
<template>
  <uni-badge text="1"></uni-badge>
</template>
```

3. **ColorUI**（纯 CSS 组件库）

```vue
<!-- 直接引入 CSS -->
<style>
@import "colorui/main.css";
@import "colorui/icon.css";
</style>

<template>
  <view class="cu-btn bg-blue">按钮</view>
</template>
```

**注意事项：**

1. 确认组件库支持的平台
2. 注意按需引入，减少包体积
3. 检查组件库的更新频率和社区活跃度
4. 考虑组件库的定制化能力

## 五、性能优化

### Taro/Uniapp 项目如何进行性能优化？

**1. 包体积优化**

```javascript
// Taro 配置（config/index.js）
module.exports = {
  mini: {
    // 压缩代码
    minify: {
      enable: true
    },
    // 分包配置
    subPackages: [
      {
        root: 'pages/sub',
        pages: ['detail/index']
      }
    ],
    // 预下载分包
    preloadRule: {
      'pages/index/index': {
        network: 'all',
        packages: ['pages/sub']
      }
    }
  }
}

// Uniapp 配置（pages.json）
{
  "subPackages": [
    {
      "root": "pages/sub",
      "pages": [
        { "path": "detail/index" }
      ]
    }
  ],
  "preloadRule": {
    "pages/index/index": {
      "network": "all",
      "packages": ["pages/sub"]
    }
  }
}
```

**2. 渲染性能优化**

```javascript
// 避免频繁 setData
// ❌ 错误：频繁更新
for (let i = 0; i < 100; i++) {
  this.setState({ count: i })
}

// ✅ 正确：批量更新
this.setState({ count: 100 })

// 使用虚拟列表
// Taro
import { VirtualList } from '@tarojs/components'

<VirtualList
  height={500}
  itemCount={1000}
  itemSize={100}
  item={({ index }) => <View>{index}</View>}
/>

// Uniapp
<recycle-list>
  <recycle-item v-for="item in list" :key="item.id">
    {{ item.name }}
  </recycle-item>
</recycle-list>
```

**3. 图片优化**

```javascript
// 使用 webp 格式
<image src="image.webp" mode="aspectFill" />

// 懒加载
<image src="image.jpg" lazy-load />

// 压缩图片
// 使用 CDN 图片处理参数
const imageUrl = 'https://cdn.example.com/image.jpg?x-oss-process=image/resize,w_750'
```

**4. 请求优化**

```javascript
// 请求合并
async function fetchData() {
  const [user, orders, products] = await Promise.all([
    api.getUser(),
    api.getOrders(),
    api.getProducts()
  ])
}

// 请求缓存
const cache = new Map()

async function fetchWithCache(url) {
  if (cache.has(url)) {
    return cache.get(url)
  }
  const data = await fetch(url)
  cache.set(url, data)
  return data
}

// 防抖节流
import { debounce, throttle } from 'lodash'

// 搜索防抖
const handleSearch = debounce((keyword) => {
  api.search(keyword)
}, 500)

// 滚动节流
const handleScroll = throttle((e) => {
  console.log(e.scrollTop)
}, 200)
```

**5. 长列表优化**

```javascript
// 分页加载
export default {
  data() {
    return {
      list: [],
      page: 1,
      loading: false,
      finished: false
    }
  },
  
  onReachBottom() {
    if (this.loading || this.finished) return
    this.loadMore()
  },
  
  async loadMore() {
    this.loading = true
    const data = await api.getList(this.page)
    
    if (data.length === 0) {
      this.finished = true
    } else {
      this.list = [...this.list, ...data]
      this.page++
    }
    
    this.loading = false
  }
}
```

### Taro/Uniapp 如何优化首屏加载速度？

**1. 分包加载**

```javascript
// 主包只保留首页和 tabBar 页面
// 其他页面放到分包中

// pages.json
{
  "pages": [
    "pages/index/index",
    "pages/user/index"
  ],
  "subPackages": [
    {
      "root": "pages/sub",
      "pages": [
        "detail/index",
        "list/index"
      ]
    }
  ]
}
```

**2. 按需加载组件**

```javascript
// Taro - 动态导入
const Detail = lazy(() => import('./pages/detail'))

// Uniapp - 异步组件
export default {
  components: {
    Detail: () => import('./components/Detail.vue')
  }
}
```

**3. 骨架屏**

```vue
<!-- Uniapp 骨架屏 -->
<template>
  <view v-if="loading" class="skeleton">
    <view class="skeleton-avatar"></view>
    <view class="skeleton-line"></view>
    <view class="skeleton-line"></view>
  </view>
  <view v-else>
    <!-- 实际内容 -->
  </view>
</template>

<style>
.skeleton-avatar {
  width: 100rpx;
  height: 100rpx;
  background: linear-gradient(90deg, #f2f2f2 25%, #e6e6e6 37%, #f2f2f2 63%);
  animation: skeleton-loading 1.4s ease infinite;
}

@keyframes skeleton-loading {
  0% { background-position: 100% 50%; }
  100% { background-position: 0 50%; }
}
</style>
```

**4. 数据预加载**

```javascript
// App.vue
export default {
  onLaunch() {
    // 预加载用户信息
    this.preloadUserInfo()
    // 预加载配置信息
    this.preloadConfig()
  },
  
  methods: {
    async preloadUserInfo() {
      const userInfo = await api.getUserInfo()
      // 存储到全局状态
      this.$store.commit('setUserInfo', userInfo)
    }
  }
}
```

**5. 静态资源优化**

```javascript
// 使用 CDN
const imageUrl = 'https://cdn.example.com/image.jpg'

// 图片压缩
// 使用 tinypng 或 imagemin 压缩图片

// 字体图标按需引入
// 只引入使用到的图标
```

### 如何实现一个高性能的虚拟列表？

虚拟列表是长列表性能优化的终极方案，核心思路是**只渲染可视区域内的元素**，非可视区域只渲染占位空白。

**1. 实现原理**

```
┌──────────────────────────┐
│   占位空白（visibleStart 之前的高度） │
├──────────────────────────┤
│   ↑ 缓冲区（bufferTop）            │
├──────────────────────────┤
│                                  │
│   ★ 可视区域（实际渲染）           │
│                                  │
├──────────────────────────┤
│   ↓ 缓冲区（bufferBottom）         │
├──────────────────────────┤
│   占位空白（visibleEnd 之后的高度）  │
└──────────────────────────┘
```

**2. 简易实现（固定行高）**

```javascript
// Vue 实现
<template>
  <scroll-view 
    scroll-y 
    :style="{ height: containerHeight + 'px' }"
    @scroll="onScroll"
  >
    <!-- 顶部占位 -->
    <view :style="{ height: topHeight + 'px' }"></view>
    
    <!-- 实际渲染的元素 -->
    <view 
      v-for="item in visibleItems" 
      :key="item.id"
      :style="{ height: itemHeight + 'px' }"
    >
      {{ item.name }}
    </view>
    
    <!-- 底部占位 -->
    <view :style="{ height: bottomHeight + 'px' }"></view>
  </scroll-view>
</template>

<script>
export default {
  data() {
    return {
      list: [],            // 完整数据
      itemHeight: 100,     // 每项高度
      containerHeight: 600, // 容器高度
      bufferCount: 5,      // 缓冲数量（避免快速滚动出现白屏）
      scrollTop: 0
    }
  },
  
  computed: {
    // 可视区域开始索引
    startIndex() {
      const idx = Math.floor(this.scrollTop / this.itemHeight) - this.bufferCount
      return Math.max(0, idx)
    },
    
    // 可视区域结束索引
    endIndex() {
      const visibleCount = Math.ceil(this.containerHeight / this.itemHeight)
      const idx = this.startIndex + visibleCount + this.bufferCount * 2
      return Math.min(this.list.length, idx)
    },
    
    // 可视元素
    visibleItems() {
      return this.list.slice(this.startIndex, this.endIndex)
    },
    
    // 顶部占位高度
    topHeight() {
      return this.startIndex * this.itemHeight
    },
    
    // 底部占位高度
    bottomHeight() {
      return (this.list.length - this.endIndex) * this.itemHeight
    }
  },
  
  methods: {
    onScroll(e) {
      this.scrollTop = e.detail.scrollTop
    }
  }
}
</script>
```

**3. 进阶优化：使用 IntersectionObserver**

小程序提供了 `createIntersectionObserver` API，可以监听节点是否进入可视区域，比 scroll 事件性能更好。

```javascript
// 监听可视区域变化
const observer = uni.createIntersectionObserver(this, {
  observeAll: true,         // 观察多个节点
  thresholds: [0],          // 触发阈值
  initialRatio: 0           // 初始相交比例
})

observer.relativeToViewport({ top: 200, bottom: 200 })  // 上下扩展 200px 缓冲区
  .observe('.list-item', (res) => {
    if (res.intersectionRatio > 0) {
      // 元素进入可视区域，渲染
      const index = res.dataset.index
      this.renderItem(index)
    } else {
      // 元素离开可视区域，可以销毁
      this.destroyItem(res.dataset.index)
    }
  })

// 页面卸载时停止观察
onUnload() {
  observer.disconnect()
}
```

**4. 时间切片优化（避免 setData 阻塞）**

```javascript
// 使用 setTimeout 把渲染拆分成多个微任务
async renderInBatch(items) {
  const batchSize = 20
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize)
    
    // 渲染一批
    this.list.push(...batch)
    
    // 让出主线程
    await new Promise(resolve => setTimeout(resolve, 0))
  }
}
```

**5. 不定高度列表的处理**

不定高度列表实现更复杂，需要动态测量每项高度：

```javascript
// 使用 createSelectorQuery 测量元素实际高度
async measureItemHeight(index) {
  return new Promise((resolve) => {
    const query = uni.createSelectorQuery().in(this)
    query.select(`#item-${index}`).boundingClientRect(rect => {
      this.itemHeights[index] = rect.height
      resolve(rect.height)
    }).exec()
  })
}

// 缓存累计高度
calculateOffset(index) {
  let offset = 0
  for (let i = 0; i < index; i++) {
    offset += this.itemHeights[i] || this.estimatedHeight
  }
  return offset
}
```

**6. 关键参数调优建议**

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| `bufferCount` | 5-10 | 缓冲数量，防止快速滚动白屏 |
| `containerHeight` | 一屏高度 | 不要设太大，避免节点过多 |
| `itemSize` | 实际测量 | 不定高度时使用平均值估算 |
| `throttleTime` | 16ms | scroll 事件节流（约 60fps） |

### 小程序分包加载如何配置和优化？

小程序对包体积有严格限制（主包 2M，整包 20M），分包加载是必备的优化手段。

**1. 基础分包配置**

```json
// pages.json (Uniapp) 或 app.config.js (Taro)
{
  "pages": [
    "pages/index/index",
    "pages/user/index"
  ],
  "subPackages": [
    {
      "root": "pagesA",                    // 分包根目录
      "name": "first",                      // 分包别名
      "pages": [
        "detail/index",
        "list/index"
      ]
    },
    {
      "root": "pagesB",
      "pages": [
        "settings/index"
      ]
    }
  ]
}
```

**2. 独立分包（独立运行）**

独立分包不依赖主包，可以独立运行，加载速度更快，常用于活动页等独立功能。

```json
{
  "subPackages": [
    {
      "root": "pagesActivity",
      "pages": ["index/index"],
      "independent": true                   // 独立分包
    }
  ]
}
```

**3. 分包预下载**

在用户访问当前页面时，预先下载将要访问的分包，提升体验。

```json
{
  "preloadRule": {
    "pages/index/index": {                  // 触发预下载的页面
      "network": "all",                     // all（不限） / wifi（仅 WiFi）
      "packages": ["pagesA", "pagesB"]      // 预下载的分包
    }
  }
}
```

**4. 分包间跳转**

```javascript
// 主包跳转到分包
uni.navigateTo({ url: '/pagesA/detail/index?id=1' })

// 分包之间跳转
uni.navigateTo({ url: '/pagesB/settings/index' })

// 分包跳回主包
uni.switchTab({ url: '/pages/index/index' })
```

**5. 分包优化策略**

- **主包只放核心页面**：首页、tabBar 页面、登录页
- **业务模块独立分包**：按业务功能划分（订单、商品、用户中心）
- **大资源放分包**：图片、字体、组件库等
- **使用分包异步化**：分包独立加载减少主包等待时间
- **配合预下载**：常用路径配置预下载
- **公共组件下沉**：多个分包共用的组件放主包，单个分包用的放分包内

**6. 包体积优化检查清单**

```bash
# 1. 分析包大小
# 微信开发者工具 → 详情 → 域名信息 → 代码包大小

# 2. 静态资源外部化
# 图片/字体 → CDN
# 大型 JSON 数据 → 接口动态加载

# 3. Tree shaking
# 按需引入组件库
import { Button } from 'vant-weapp'  // ✅ 按需
// import 'vant-weapp'  // ❌ 全量引入

# 4. 压缩代码
# config/index.js
mini: {
  minify: { enable: true }
}
```

### 如何降低小程序节点数量？

小程序对节点数量有限制：建议页面 WXML **节点数 < 1000**，**树深度 < 30 层**，**子节点数 < 60**。

**1. 减少不必要的嵌套**

```vue
<!-- ❌ 错误：过度嵌套 -->
<view>
  <view>
    <view>
      <view>{{ text }}</view>
    </view>
  </view>
</view>

<!-- ✅ 正确：扁平化结构 -->
<text>{{ text }}</text>
```

**2. 使用 v-if 而非 v-show**

```vue
<!-- v-show 元素始终在 DOM 中 -->
<view v-show="visible">内容</view>

<!-- v-if 元素只在需要时存在 -->
<view v-if="visible">内容</view>
```

**3. 长列表分页或虚拟滚动**

使用上面介绍的虚拟列表方案，控制实际渲染节点数量。

**4. 长文本截断**

```javascript
// 显示省略号而非完整文本
function truncate(text, maxLength = 100) {
  return text.length > maxLength 
    ? text.slice(0, maxLength) + '...' 
    : text
}
```

**5. 使用 cover-view 替代 view（在原生组件上）**

某些原生组件（map、video）有层级问题，需要使用 cover-view 覆盖。

## 六、常见问题与解决方案

### Taro/Uniapp 开发中常见的坑有哪些？

#### 1. 样式单位问题

**问题描述：**

在跨端开发中，不同设备的屏幕尺寸差异很大，如果使用固定的 `px` 单位，会导致在不同设备上显示效果不一致。例如，在 iPhone 6 上显示正常的元素，在 iPad 上可能会显得很小。

**原因分析：**

- `px` 是绝对单位，在所有设备上都是固定大小
- 不同设备的屏幕宽度不同（iPhone 6: 375px, iPad: 768px）
- 小程序和 H5 的渲染机制不同

**解决方案：**

使用 `rpx`（responsive pixel）响应式像素单位。rpx 是小程序和 Uniapp 提供的相对单位，会根据屏幕宽度自动缩放。规则是：无论屏幕多宽，都被分为 750rpx，即 `1rpx = 屏幕宽度 / 750`。

```css
/* ❌ 错误：使用固定 px */
.container {
  width: 375px; /* 在大屏设备上显得很小 */
  padding: 10px;
}

/* ✅ 正确：使用 rpx 自动适配 */
.container {
  width: 750rpx; /* 占满整个屏幕宽度 */
  padding: 20rpx; /* 自动按比例缩放 */
}

/* 注意：1px 边框问题需要特殊处理 */
.border {
  border: 1px solid #ddd; /* 边框仍使用 px */
}
```

#### 2. 异步渲染时序问题

**问题描述：**

在页面刚加载时就尝试获取 DOM 节点信息，经常会获取不到或获取到错误的数据。这是因为页面渲染是异步的，在 `onLoad` 时页面可能还没有完成渲染。

**原因分析：**

- `onLoad`：页面参数已获取，但页面还未开始渲染
- `onShow`：页面显示，但渲染可能还未完成
- `onReady`：页面首次渲染完成，此时可以安全地操作节点

**解决方案：**

需要操作 DOM 节点的逻辑，应该放在 `onReady` 生命周期中，并且最好加上延迟，确保渲染完全完成。

```javascript
// ❌ 错误：在 onLoad 中操作节点
onLoad() {
  const query = Taro.createSelectorQuery()
  query.select('.container').boundingClientRect()
  query.exec((res) => {
    console.log(res) // 可能为 null
  })
}

// ✅ 正确：在 onReady 中操作
onReady() {
  // 加上延迟确保渲染完成
  setTimeout(() => {
    const query = Taro.createSelectorQuery()
    query.select('.container').boundingClientRect()
    query.exec((res) => {
      console.log(res) // 能正确获取到节点信息
    })
  }, 100)
}
```

#### 3. 事件传参问题

**问题描述：**

在小程序中，不能像 Web 开发那样直接在事件处理函数中传递参数，因为小程序的事件绑定机制不同。

**原因分析：**

- 小程序的事件系统基于 WXML，不支持直接传参
- 需要通过 `data-*` 属性来传递数据
- 事件对象中的 `currentTarget.dataset` 包含了所有 `data-*` 属性

**解决方案：**

使用 `data-*` 自定义属性传递参数，在事件处理函数中通过 `e.currentTarget.dataset` 获取。

```vue
<!-- ❌ 错误：直接传参（小程序不支持） -->
<button @click="handleClick(item.id, item.name)">点击</button>

<!-- ✅ 正确：使用 data-* 属性 -->
<button 
  @click="handleClick" 
  :data-id="item.id"
  :data-name="item.name"
>
  点击
</button>

<script>
export default {
  methods: {
    handleClick(e) {
      // 从 dataset 中获取参数
      const { id, name } = e.currentTarget.dataset
      console.log(id, name)
      
      // 注意：dataset 中的属性名会自动转为小写
      // data-userId 会变成 userid
    }
  }
}
</script>
```

#### 4. 页面栈溢出问题

**问题描述：**

小程序的页面栈有最大层数限制（通常是 10 层），如果不断使用 `navigateTo` 跳转，会导致页面栈溢出，无法继续跳转。

**原因分析：**

- 小程序为了控制内存占用，限制了页面栈的最大深度
- `navigateTo` 会保留当前页面，将新页面压入栈中
- 当栈满时，`navigateTo` 会失败

**解决方案：**

根据业务场景选择合适的路由 API：
- 需要返回的页面：使用 `navigateTo`
- 不需要返回的页面：使用 `redirectTo`（关闭当前页）
- 重新开始流程：使用 `reLaunch`（关闭所有页面）

```javascript
// ❌ 错误：无限使用 navigateTo
for (let i = 0; i < 20; i++) {
  Taro.navigateTo({ url: '/pages/detail/index' })
}
// 超过 10 层后会报错

// ✅ 正确：根据场景选择 API
// 场景1：需要返回上一页
Taro.navigateTo({ url: '/pages/detail/index' })

// 场景2：不需要返回（如登录后跳转首页）
Taro.redirectTo({ url: '/pages/index/index' })

// 场景3：重新开始（如退出登录）
Taro.reLaunch({ url: '/pages/login/index' })

// 场景4：跳转到 tabBar 页面
Taro.switchTab({ url: '/pages/index/index' })
```

#### 5. 生命周期执行顺序问题

**问题描述：**

不理解生命周期的执行顺序，导致在错误的时机执行代码，引发各种问题。

**原因分析：**

页面生命周期的执行顺序是固定的：
1. `onLoad`：页面加载，接收路由参数
2. `onShow`：页面显示（每次显示都会触发）
3. `onReady`：页面首次渲染完成（只触发一次）

**解决方案：**

根据需求在合适的生命周期中执行代码：

```javascript
export default {
  // 1. 获取路由参数、初始化数据
  onLoad(options) {
    this.id = options.id
    this.fetchData() // 请求数据
  },
  
  // 2. 页面显示时的逻辑（每次都会执行）
  onShow() {
    this.refreshData() // 刷新数据
    this.startTimer() // 启动定时器
  },
  
  // 3. 页面渲染完成后的操作（只执行一次）
  onReady() {
    this.getNodeInfo() // 获取节点信息
    this.initAnimation() // 初始化动画
  },
  
  // 4. 页面隐藏时的清理
  onHide() {
    this.stopTimer() // 停止定时器
  },
  
  // 5. 页面卸载时的清理
  onUnload() {
    this.clearData() // 清理数据
  }
}
```

#### 6. H5 跨域问题

**问题描述：**

在 H5 环境下，直接请求后端 API 会遇到跨域问题，而小程序环境没有这个限制。

**原因分析：**

- 浏览器的同源策略限制了跨域请求
- 小程序的网络请求是通过原生能力实现的，不受浏览器限制
- 开发环境和生产环境的处理方式不同

**解决方案：**

开发环境使用代理，生产环境配置 CORS 或使用同域部署。

```javascript
// vue.config.js (Uniapp H5 配置)
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'https://api.example.com', // 后端地址
        changeOrigin: true, // 改变请求头中的 origin
        pathRewrite: {
          '^/api': '' // 重写路径
        }
      }
    }
  }
}

// 请求时使用相对路径
// 开发环境：/api/user -> http://localhost:8080/api/user -> https://api.example.com/user
// 生产环境：需要后端配置 CORS 或使用同域部署
uni.request({
  url: '/api/user/info',
  success: (res) => {
    console.log(res)
  }
})
```

#### 7. 图片路径问题

**问题描述：**

使用相对路径引用图片时，在不同平台可能会出现图片加载失败的情况。

**原因分析：**

- 小程序和 H5 的资源加载机制不同
- 相对路径在编译后可能会被错误解析
- 动态路径在小程序中不支持

**解决方案：**

使用绝对路径或网络路径，避免使用相对路径和动态路径。

```vue
<template>
  <view>
    <!-- ❌ 错误：相对路径 -->
    <image src="./image.jpg" />
    <image src="../assets/logo.png" />
    
    <!-- ❌ 错误：动态路径（小程序不支持） -->
    <image :src="require(`@/assets/${imageName}.png`)" />
    
    <!-- ✅ 正确：绝对路径 -->
    <image src="/static/image.jpg" />
    
    <!-- ✅ 正确：网络路径 -->
    <image src="https://cdn.example.com/image.jpg" />
    
    <!-- ✅ 正确：使用条件编译 -->
    <!-- #ifdef H5 -->
    <image :src="require('@/assets/logo.png')" />
    <!-- #endif -->
    
    <!-- #ifndef H5 -->
    <image src="/static/logo.png" />
    <!-- #endif -->
  </view>
</template>

<script>
export default {
  data() {
    return {
      // 图片路径统一管理
      images: {
        logo: '/static/logo.png',
        avatar: 'https://cdn.example.com/avatar.jpg'
      }
    }
  }
}
</script>
```

### 如何在 Taro/Uniapp 中实现全局状态管理？

**Taro 状态管理（使用 Redux）：**

```javascript
// store/index.js
import { createStore } from 'redux'

const initialState = {
  userInfo: null,
  token: ''
}

function reducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_USER_INFO':
      return { ...state, userInfo: action.payload }
    case 'SET_TOKEN':
      return { ...state, token: action.payload }
    default:
      return state
  }
}

export default createStore(reducer)

// App.jsx
import { Provider } from 'react-redux'
import store from './store'

function App({ children }) {
  return (
    <Provider store={store}>
      {children}
    </Provider>
  )
}

// 使用
import { useSelector, useDispatch } from 'react-redux'

function Index() {
  const userInfo = useSelector(state => state.userInfo)
  const dispatch = useDispatch()
  
  const login = () => {
    dispatch({ type: 'SET_USER_INFO', payload: { name: 'Tom' } })
  }
}
```

**Uniapp 状态管理（使用 Vuex）：**

```javascript
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    userInfo: null,
    token: ''
  },
  
  mutations: {
    SET_USER_INFO(state, userInfo) {
      state.userInfo = userInfo
    },
    SET_TOKEN(state, token) {
      state.token = token
    }
  },
  
  actions: {
    async login({ commit }, { username, password }) {
      const res = await api.login(username, password)
      commit('SET_USER_INFO', res.userInfo)
      commit('SET_TOKEN', res.token)
    }
  },
  
  getters: {
    isLogin: state => !!state.token
  }
})

// main.js
import store from './store'

const app = new Vue({
  store,
  ...App
})

// 使用
export default {
  computed: {
    userInfo() {
      return this.$store.state.userInfo
    },
    isLogin() {
      return this.$store.getters.isLogin
    }
  },
  
  methods: {
    login() {
      this.$store.dispatch('login', {
        username: 'admin',
        password: '123456'
      })
    }
  }
}
```

**使用 Pinia（Vue 3）：**

```javascript
// store/user.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    userInfo: null,
    token: ''
  }),
  
  actions: {
    setUserInfo(userInfo) {
      this.userInfo = userInfo
    },
    
    async login(username, password) {
      const res = await api.login(username, password)
      this.userInfo = res.userInfo
      this.token = res.token
    }
  },
  
  getters: {
    isLogin: (state) => !!state.token
  }
})

// 使用
import { useUserStore } from '@/store/user'

export default {
  setup() {
    const userStore = useUserStore()
    
    const login = () => {
      userStore.login('admin', '123456')
    }
    
    return {
      userInfo: userStore.userInfo,
      login
    }
  }
}
```

### 如何在 Taro/Uniapp 中进行网络请求封装？

**Taro 网络请求封装：**

```javascript
// utils/request.js
import Taro from '@tarojs/taro'

const BASE_URL = 'https://api.example.com'

class Request {
  constructor() {
    this.baseURL = BASE_URL
    this.timeout = 10000
  }
  
  // 请求拦截
  interceptors(config) {
    // 添加 token
    const token = Taro.getStorageSync('token')
    if (token) {
      config.header = {
        ...config.header,
        Authorization: `Bearer ${token}`
      }
    }
    return config
  }
  
  // 响应处理
  handleResponse(res) {
    const { statusCode, data } = res
    
    if (statusCode === 200) {
      if (data.code === 0) {
        return data.data
      } else {
        Taro.showToast({
          title: data.message || '请求失败',
          icon: 'none'
        })
        return Promise.reject(data)
      }
    } else if (statusCode === 401) {
      // 未登录，跳转到登录页
      Taro.navigateTo({ url: '/pages/login/index' })
      return Promise.reject(res)
    } else {
      Taro.showToast({
        title: '网络错误',
        icon: 'none'
      })
      return Promise.reject(res)
    }
  }
  
  // 请求方法
  request(options) {
    const config = this.interceptors({
      url: this.baseURL + options.url,
      method: options.method || 'GET',
      data: options.data || {},
      header: options.header || {},
      timeout: this.timeout
    })
    
    return Taro.request(config)
      .then(this.handleResponse)
      .catch(err => {
        console.error('请求错误:', err)
        return Promise.reject(err)
      })
  }
  
  get(url, data) {
    return this.request({ url, method: 'GET', data })
  }
  
  post(url, data) {
    return this.request({ url, method: 'POST', data })
  }
  
  put(url, data) {
    return this.request({ url, method: 'PUT', data })
  }
  
  delete(url, data) {
    return this.request({ url, method: 'DELETE', data })
  }
}

export default new Request()

// 使用
import request from '@/utils/request'

// GET 请求
const userInfo = await request.get('/user/info')

// POST 请求
const result = await request.post('/user/login', {
  username: 'admin',
  password: '123456'
})
```

**Uniapp 网络请求封装：**

```javascript
// utils/request.js
const BASE_URL = 'https://api.example.com'

class Request {
  constructor() {
    this.baseURL = BASE_URL
    this.timeout = 10000
  }
  
  request(options) {
    return new Promise((resolve, reject) => {
      // 添加 token
      const token = uni.getStorageSync('token')
      const header = {
        ...options.header,
        Authorization: token ? `Bearer ${token}` : ''
      }
      
      uni.request({
        url: this.baseURL + options.url,
        method: options.method || 'GET',
        data: options.data || {},
        header: header,
        timeout: this.timeout,
        
        success: (res) => {
          const { statusCode, data } = res
          
          if (statusCode === 200) {
            if (data.code === 0) {
              resolve(data.data)
            } else {
              uni.showToast({
                title: data.message || '请求失败',
                icon: 'none'
              })
              reject(data)
            }
          } else if (statusCode === 401) {
            uni.navigateTo({ url: '/pages/login/index' })
            reject(res)
          } else {
            uni.showToast({
              title: '网络错误',
              icon: 'none'
            })
            reject(res)
          }
        },
        
        fail: (err) => {
          uni.showToast({
            title: '网络请求失败',
            icon: 'none'
          })
          reject(err)
        }
      })
    })
  }
  
  get(url, data) {
    return this.request({ url, method: 'GET', data })
  }
  
  post(url, data) {
    return this.request({ url, method: 'POST', data })
  }
}

export default new Request()
```

### 如何实现自定义导航栏和 TabBar？

**Taro 自定义导航栏：**

```javascript
// app.config.js
export default {
  window: {
    navigationStyle: 'custom' // 隐藏默认导航栏
  }
}

// components/NavBar/index.jsx
import { View, Text } from '@tarojs/components'
import Taro from '@tarojs/taro'
import { useState, useEffect } from 'react'
import './index.scss'

function NavBar({ title, showBack = true }) {
  const [statusBarHeight, setStatusBarHeight] = useState(0)
  
  useEffect(() => {
    const { statusBarHeight } = Taro.getSystemInfoSync()
    setStatusBarHeight(statusBarHeight)
  }, [])
  
  const handleBack = () => {
    Taro.navigateBack()
  }
  
  return (
    <View className="nav-bar" style={{ paddingTop: `${statusBarHeight}px` }}>
      <View className="nav-bar-content">
        {showBack && (
          <View className="nav-bar-back" onClick={handleBack}>
            返回
          </View>
        )}
        <Text className="nav-bar-title">{title}</Text>
      </View>
    </View>
  )
}

export default NavBar
```

**Uniapp 自定义导航栏：**

```vue
<!-- components/NavBar/index.vue -->
<template>
  <view class="nav-bar" :style="{ paddingTop: statusBarHeight + 'px' }">
    <view class="nav-bar-content">
      <view v-if="showBack" class="nav-bar-back" @click="handleBack">
        返回
      </view>
      <text class="nav-bar-title">{{ title }}</text>
    </view>
  </view>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      default: ''
    },
    showBack: {
      type: Boolean,
      default: true
    }
  },
  
  data() {
    return {
      statusBarHeight: 0
    }
  },
  
  mounted() {
    const { statusBarHeight } = uni.getSystemInfoSync()
    this.statusBarHeight = statusBarHeight
  },
  
  methods: {
    handleBack() {
      uni.navigateBack()
    }
  }
}
</script>

<style scoped>
.nav-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background: #fff;
  z-index: 999;
}

.nav-bar-content {
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.nav-bar-back {
  position: absolute;
  left: 16px;
}

.nav-bar-title {
  font-size: 18px;
  font-weight: bold;
}
</style>
```

**自定义 TabBar：**

```vue
<!-- components/TabBar/index.vue -->
<template>
  <view class="tab-bar">
    <view
      v-for="(item, index) in list"
      :key="index"
      class="tab-bar-item"
      @click="switchTab(item, index)"
    >
      <image
        class="tab-bar-icon"
        :src="current === index ? item.selectedIconPath : item.iconPath"
      />
      <text
        class="tab-bar-text"
        :class="{ active: current === index }"
      >
        {{ item.text }}
      </text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      current: 0,
      list: [
        {
          pagePath: '/pages/index/index',
          text: '首页',
          iconPath: '/static/home.png',
          selectedIconPath: '/static/home-active.png'
        },
        {
          pagePath: '/pages/user/index',
          text: '我的',
          iconPath: '/static/user.png',
          selectedIconPath: '/static/user-active.png'
        }
      ]
    }
  },
  
  methods: {
    switchTab(item, index) {
      this.current = index
      uni.switchTab({
        url: item.pagePath
      })
    }
  }
}
</script>
```

## 七、实战经验

### 如何实现下拉刷新和上拉加载？

**Taro 实现：**

```javascript
import { View, ScrollView } from '@tarojs/components'
import { useState } from 'react'
import Taro from '@tarojs/taro'

function Index() {
  const [list, setList] = useState([])
  const [page, setPage] = useState(1)
  const [loading, setLoading] = useState(false)
  const [finished, setFinished] = useState(false)
  
  // 下拉刷新
  const onPullDownRefresh = async () => {
    setPage(1)
    setFinished(false)
    const data = await fetchList(1)
    setList(data)
    Taro.stopPullDownRefresh()
  }
  
  // 上拉加载
  const onReachBottom = async () => {
    if (loading || finished) return
    
    setLoading(true)
    const nextPage = page + 1
    const data = await fetchList(nextPage)
    
    if (data.length === 0) {
      setFinished(true)
    } else {
      setList([...list, ...data])
      setPage(nextPage)
    }
    
    setLoading(false)
  }
  
  const fetchList = async (page) => {
    const res = await api.getList({ page, pageSize: 20 })
    return res.data
  }
  
  return (
    <View>
      {list.map(item => (
        <View key={item.id}>{item.name}</View>
      ))}
      {loading && <View>加载中...</View>}
      {finished && <View>没有更多了</View>}
    </View>
  )
}

// 配置文件中启用下拉刷新
export default {
  enablePullDownRefresh: true
}
```

**Uniapp 实现：**

```vue
<template>
  <view>
    <view v-for="item in list" :key="item.id">
      {{ item.name }}
    </view>
    <view v-if="loading">加载中...</view>
    <view v-if="finished">没有更多了</view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      list: [],
      page: 1,
      loading: false,
      finished: false
    }
  },
  
  onLoad() {
    this.loadData()
  },
  
  // 下拉刷新
  onPullDownRefresh() {
    this.page = 1
    this.finished = false
    this.loadData().then(() => {
      uni.stopPullDownRefresh()
    })
  },
  
  // 上拉加载
  onReachBottom() {
    if (this.loading || this.finished) return
    this.loadMore()
  },
  
  methods: {
    async loadData() {
      const data = await this.fetchList(1)
      this.list = data
    },
    
    async loadMore() {
      this.loading = true
      const nextPage = this.page + 1
      const data = await this.fetchList(nextPage)
      
      if (data.length === 0) {
        this.finished = true
      } else {
        this.list = [...this.list, ...data]
        this.page = nextPage
      }
      
      this.loading = false
    },
    
    async fetchList(page) {
      const res = await this.$api.getList({ page, pageSize: 20 })
      return res.data
    }
  }
}
</script>
```

### 如何实现图片上传和预览？

**Taro 图片上传：**

```javascript
import Taro from '@tarojs/taro'
import { View, Image } from '@tarojs/components'
import { useState } from 'react'

function Upload() {
  const [imageList, setImageList] = useState([])
  
  // 选择图片
  const chooseImage = () => {
    Taro.chooseImage({
      count: 9,
      sizeType: ['compressed'],
      sourceType: ['album', 'camera'],
      success: (res) => {
        const tempFilePaths = res.tempFilePaths
        uploadImages(tempFilePaths)
      }
    })
  }
  
  // 上传图片
  const uploadImages = async (filePaths) => {
    const uploadPromises = filePaths.map(filePath => {
      return Taro.uploadFile({
        url: 'https://api.example.com/upload',
        filePath: filePath,
        name: 'file',
        header: {
          Authorization: `Bearer ${Taro.getStorageSync('token')}`
        }
      })
    })
    
    try {
      const results = await Promise.all(uploadPromises)
      const urls = results.map(res => JSON.parse(res.data).url)
      setImageList([...imageList, ...urls])
      
      Taro.showToast({
        title: '上传成功',
        icon: 'success'
      })
    } catch (err) {
      Taro.showToast({
        title: '上传失败',
        icon: 'none'
      })
    }
  }
  
  // 预览图片
  const previewImage = (current) => {
    Taro.previewImage({
      current: current,
      urls: imageList
    })
  }
  
  // 删除图片
  const deleteImage = (index) => {
    const newList = imageList.filter((_, i) => i !== index)
    setImageList(newList)
  }
  
  return (
    <View>
      {imageList.map((url, index) => (
        <View key={index} className="image-item">
          <Image
            src={url}
            mode="aspectFill"
            onClick={() => previewImage(url)}
          />
          <View className="delete-btn" onClick={() => deleteImage(index)}>
            删除
          </View>
        </View>
      ))}
      <View className="upload-btn" onClick={chooseImage}>
        + 上传图片
      </View>
    </View>
  )
}
```

**Uniapp 图片上传：**

```vue
<template>
  <view>
    <view v-for="(url, index) in imageList" :key="index" class="image-item">
      <image
        :src="url"
        mode="aspectFill"
        @click="previewImage(url)"
      />
      <view class="delete-btn" @click="deleteImage(index)">删除</view>
    </view>
    <view class="upload-btn" @click="chooseImage">+ 上传图片</view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      imageList: []
    }
  },
  
  methods: {
    // 选择图片
    chooseImage() {
      uni.chooseImage({
        count: 9,
        sizeType: ['compressed'],
        sourceType: ['album', 'camera'],
        success: (res) => {
          this.uploadImages(res.tempFilePaths)
        }
      })
    },
    
    // 上传图片
    async uploadImages(filePaths) {
      uni.showLoading({ title: '上传中...' })
      
      const uploadPromises = filePaths.map(filePath => {
        return new Promise((resolve, reject) => {
          uni.uploadFile({
            url: 'https://api.example.com/upload',
            filePath: filePath,
            name: 'file',
            header: {
              Authorization: `Bearer ${uni.getStorageSync('token')}`
            },
            success: (res) => {
              const data = JSON.parse(res.data)
              resolve(data.url)
            },
            fail: reject
          })
        })
      })
      
      try {
        const urls = await Promise.all(uploadPromises)
        this.imageList = [...this.imageList, ...urls]
        uni.hideLoading()
        uni.showToast({ title: '上传成功', icon: 'success' })
      } catch (err) {
        uni.hideLoading()
        uni.showToast({ title: '上传失败', icon: 'none' })
      }
    },
    
    // 预览图片
    previewImage(current) {
      uni.previewImage({
        current: current,
        urls: this.imageList
      })
    },
    
    // 删除图片
    deleteImage(index) {
      this.imageList.splice(index, 1)
    }
  }
}
</script>
```

### 如何实现支付功能？

**Taro 微信支付：**

```javascript
import Taro from '@tarojs/taro'

async function wxPay(orderId) {
  try {
    // 1. 调用后端接口获取支付参数
    const payParams = await api.createOrder(orderId)
    
    // 2. 调起微信支付
    const res = await Taro.requestPayment({
      timeStamp: payParams.timeStamp,
      nonceStr: payParams.nonceStr,
      package: payParams.package,
      signType: payParams.signType,
      paySign: payParams.paySign
    })
    
    // 3. 支付成功
    Taro.showToast({
      title: '支付成功',
      icon: 'success'
    })
    
    // 4. 跳转到订单详情
    Taro.redirectTo({
      url: `/pages/order/detail?id=${orderId}`
    })
    
  } catch (err) {
    if (err.errMsg === 'requestPayment:fail cancel') {
      Taro.showToast({
        title: '取消支付',
        icon: 'none'
      })
    } else {
      Taro.showToast({
        title: '支付失败',
        icon: 'none'
      })
    }
  }
}
```

**Uniapp 多平台支付：**

```javascript
export default {
  methods: {
    async handlePay(orderId) {
      // 1. 获取支付参数
      const payParams = await this.$api.createOrder(orderId)
      
      // 2. 根据平台调用不同的支付方式
      // #ifdef MP-WEIXIN
      this.wxPay(payParams)
      // #endif
      
      // #ifdef MP-ALIPAY
      this.alipayPay(payParams)
      // #endif
      
      // #ifdef H5
      this.h5Pay(payParams)
      // #endif
    },
    
    // 微信支付
    wxPay(payParams) {
      uni.requestPayment({
        provider: 'wxpay',
        timeStamp: payParams.timeStamp,
        nonceStr: payParams.nonceStr,
        package: payParams.package,
        signType: payParams.signType,
        paySign: payParams.paySign,
        success: () => {
          uni.showToast({ title: '支付成功', icon: 'success' })
          this.checkPayResult()
        },
        fail: (err) => {
          if (err.errMsg === 'requestPayment:fail cancel') {
            uni.showToast({ title: '取消支付', icon: 'none' })
          } else {
            uni.showToast({ title: '支付失败', icon: 'none' })
          }
        }
      })
    },
    
    // 支付宝支付
    alipayPay(payParams) {
      uni.requestPayment({
        provider: 'alipay',
        orderInfo: payParams.orderInfo,
        success: () => {
          uni.showToast({ title: '支付成功', icon: 'success' })
          this.checkPayResult()
        },
        fail: () => {
          uni.showToast({ title: '支付失败', icon: 'none' })
        }
      })
    },
    
    // H5 支付
    h5Pay(payParams) {
      // 跳转到支付页面
      window.location.href = payParams.payUrl
    },
    
    // 检查支付结果
    async checkPayResult() {
      const result = await this.$api.checkPayStatus(this.orderId)
      if (result.status === 'success') {
        uni.redirectTo({
          url: `/pages/order/detail?id=${this.orderId}`
        })
      }
    }
  }
}
```

### 如何实现分享功能？

**Taro 分享功能：**

```javascript
import Taro from '@tarojs/taro'
import { useShareAppMessage, useShareTimeline } from '@tarojs/taro'

function Index() {
  // 分享给朋友
  useShareAppMessage(() => {
    return {
      title: '分享标题',
      path: '/pages/index/index?id=123',
      imageUrl: 'https://cdn.example.com/share.jpg'
    }
  })
  
  // 分享到朋友圈（微信小程序）
  useShareTimeline(() => {
    return {
      title: '分享到朋友圈',
      query: 'id=123',
      imageUrl: 'https://cdn.example.com/share.jpg'
    }
  })
  
  // 主动触发分享
  const handleShare = () => {
    Taro.showShareMenu({
      withShareTicket: true,
      menus: ['shareAppMessage', 'shareTimeline']
    })
  }
  
  return (
    <View>
      <Button onClick={handleShare}>分享</Button>
    </View>
  )
}

// 类组件写法
class Index extends Component {
  onShareAppMessage() {
    return {
      title: '分享标题',
      path: '/pages/index/index',
      imageUrl: 'https://cdn.example.com/share.jpg'
    }
  }
  
  onShareTimeline() {
    return {
      title: '分享到朋友圈'
    }
  }
}
```

**Uniapp 分享功能：**

```vue
<template>
  <view>
    <button open-type="share">分享</button>
  </view>
</template>

<script>
export default {
  // 分享给朋友
  onShareAppMessage(res) {
    // 来自页面内转发按钮
    if (res.from === 'button') {
      console.log(res.target)
    }
    
    return {
      title: '分享标题',
      path: '/pages/index/index?id=123',
      imageUrl: 'https://cdn.example.com/share.jpg'
    }
  },
  
  // 分享到朋友圈（微信小程序）
  onShareTimeline() {
    return {
      title: '分享到朋友圈',
      query: 'id=123',
      imageUrl: 'https://cdn.example.com/share.jpg'
    }
  },
  
  methods: {
    // 主动触发分享
    handleShare() {
      // #ifdef MP-WEIXIN
      uni.showShareMenu({
        withShareTicket: true,
        menus: ['shareAppMessage', 'shareTimeline']
      })
      // #endif
      
      // #ifdef H5
      // H5 可以使用 Web Share API
      if (navigator.share) {
        navigator.share({
          title: '分享标题',
          text: '分享内容',
          url: window.location.href
        })
      }
      // #endif
    }
  }
}
</script>
```

**动态生成分享图片：**

```javascript
// 使用 Canvas 生成分享图片
async function generateShareImage() {
  return new Promise((resolve) => {
    const query = Taro.createSelectorQuery()
    query.select('#shareCanvas')
      .fields({ node: true, size: true })
      .exec((res) => {
        const canvas = res[0].node
        const ctx = canvas.getContext('2d')
        
        // 设置画布尺寸
        canvas.width = 500
        canvas.height = 400
        
        // 绘制背景
        ctx.fillStyle = '#fff'
        ctx.fillRect(0, 0, 500, 400)
        
        // 绘制标题
        ctx.fillStyle = '#000'
        ctx.font = 'bold 24px sans-serif'
        ctx.fillText('分享标题', 20, 50)
        
        // 绘制二维码
        const qrcode = canvas.createImage()
        qrcode.onload = () => {
          ctx.drawImage(qrcode, 350, 250, 120, 120)
          
          // 导出图片
          Taro.canvasToTempFilePath({
            canvas: canvas,
            success: (res) => {
              resolve(res.tempFilePath)
            }
          })
        }
        qrcode.src = '/static/qrcode.png'
      })
  })
}
```

### Taro/Uniapp 常见兼容性问题及解决方案

#### 1. 不同平台的样式差异

**问题描述：**

在跨端开发中，同一套样式代码在不同平台上的表现可能不一致。主要体现在：rpx 单位的计算方式、CSS 属性的支持程度、默认样式的差异等。

**原因分析：**

- 小程序使用自己的渲染引擎，H5 使用浏览器渲染引擎
- 不同平台对 CSS 新特性的支持程度不同
- 各平台的默认样式（如字体、行高）存在差异
- rpx 在 H5 中需要运行时转换，可能存在精度问题

**解决方案：**

使用条件编译针对不同平台编写样式，或使用 CSS 变量统一管理。对于关键样式，建议在各平台实际测试。

```css
/* 方案1：使用条件编译 */
/* #ifdef MP-WEIXIN */
.container {
  padding: 20rpx;
  font-size: 28rpx;
  line-height: 1.5;
}
/* #endif */

/* #ifdef H5 */
.container {
  padding: 10px;
  font-size: 14px;
  line-height: 1.6; /* H5 可能需要不同的行高 */
}
/* #endif */

/* 方案2：使用 CSS 变量统一管理 */
:root {
  --padding-base: 20rpx;
  --font-size-base: 28rpx;
  --color-primary: #007aff;
}

.container {
  padding: var(--padding-base);
  font-size: var(--font-size-base);
  color: var(--color-primary);
}

/* 方案3：重置默认样式 */
page {
  /* 统一字体 */
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
  /* 统一行高 */
  line-height: 1.5;
  /* 统一背景色 */
  background-color: #f5f5f5;
}
```

#### 2. 小程序和 H5 的路由差异

**问题描述：**

小程序和 H5 的路由机制完全不同，导致路由跳转、参数传递、页面栈管理等方面存在差异。

**原因分析：**

- 小程序使用自己的路由系统，有页面栈限制（10层）
- H5 使用浏览器的 History API，理论上无限制
- 参数传递方式不同：小程序通过 URL query，H5 可以使用多种方式
- 小程序的 tabBar 页面只能用 `switchTab` 跳转

**解决方案：**

封装统一的路由跳转方法，内部根据平台和场景选择合适的 API。对于复杂参数，建议使用全局状态管理或本地存储。

```javascript
// 统一路由封装
class Router {
  // 跳转页面
  navigateTo(url, params = {}) {
    // 处理参数
    const query = this.buildQuery(params)
    const fullUrl = query ? `${url}?${query}` : url
    
    // #ifdef H5
    // H5 可以使用 router.push
    uni.navigateTo({ url: fullUrl })
    // #endif
    
    // #ifndef H5
    // 小程序需要检查页面栈
    const pages = getCurrentPages()
    if (pages.length >= 10) {
      // 页面栈满了，使用 redirectTo
      uni.redirectTo({ url: fullUrl })
    } else {
      uni.navigateTo({ url: fullUrl })
    }
    // #endif
  }
  
  // 构建 query 字符串
  buildQuery(params) {
    return Object.keys(params)
      .map(key => {
        const value = params[key]
        // 复杂对象需要序列化
        if (typeof value === 'object') {
          return `${key}=${encodeURIComponent(JSON.stringify(value))}`
        }
        return `${key}=${encodeURIComponent(value)}`
      })
      .join('&')
  }
  
  // 解析 query 参数
  parseQuery(query) {
    const params = {}
    for (const key in query) {
      try {
        // 尝试解析 JSON
        params[key] = JSON.parse(decodeURIComponent(query[key]))
      } catch {
        // 普通字符串
        params[key] = decodeURIComponent(query[key])
      }
    }
    return params
  }
}

export default new Router()

// 使用示例
import router from '@/utils/router'

// 简单参数
router.navigateTo('/pages/detail/index', { id: 123 })

// 复杂参数
router.navigateTo('/pages/detail/index', {
  id: 123,
  data: { name: 'test', list: [1, 2, 3] }
})

// 接收参数
onLoad(options) {
  const params = router.parseQuery(options)
  console.log(params.data) // { name: 'test', list: [1, 2, 3] }
}
```

**3. 不同平台的存储限制**

**问题描述：**

不同平台对本地存储的容量限制不同，如果不加控制，可能导致存储失败或数据丢失。

**原因分析：**

- 微信小程序：单个 key 最大 1MB，总容量 10MB
- 支付宝小程序：单个 key 最大 200KB，总容量 10MB
- H5 localStorage：5-10MB（取决于浏览器）
- App：理论上无限制（受设备存储影响）

**解决方案：**

封装存储方法，添加容量检测、数据压缩、过期清理等机制。对于大数据，考虑分片存储或使用云存储。

```javascript
class Storage {
  constructor() {
    this.maxSize = 1024 * 1024 // 1MB
    this.prefix = 'app_'
  }
  
  // 设置存储
  set(key, value, expire) {
    try {
      const data = {
        value: value,
        expire: expire ? Date.now() + expire : null,
        timestamp: Date.now()
      }
      
      const jsonStr = JSON.stringify(data)
      const size = new Blob([jsonStr]).size
      
      // 检查数据大小
      if (size > this.maxSize) {
        console.warn(`数据过大(${size}字节)，建议分片存储`)
        // 可以考虑压缩或分片
        return this.setLarge(key, value, expire)
      }
      
      uni.setStorageSync(this.prefix + key, jsonStr)
      return true
      
    } catch (error) {
      if (error.name === 'QuotaExceededError') {
        // 存储空间不足，清理过期数据
        this.clearExpired()
        // 重试一次
        return this.set(key, value, expire)
      }
      console.error('存储失败:', error)
      return false
    }
  }
  
  // 获取存储
  get(key) {
    try {
      const jsonStr = uni.getStorageSync(this.prefix + key)
      if (!jsonStr) return null
      
      const data = JSON.parse(jsonStr)
      
      // 检查是否过期
      if (data.expire && Date.now() > data.expire) {
        this.remove(key)
        return null
      }
      
      return data.value
      
    } catch (error) {
      console.error('读取失败:', error)
      return null
    }
  }
  
  // 清理过期数据
  clearExpired() {
    try {
      const info = uni.getStorageInfoSync()
      const keys = info.keys.filter(k => k.startsWith(this.prefix))
      
      keys.forEach(key => {
        const data = this.get(key.replace(this.prefix, ''))
        if (data === null) {
          // 已过期或无效，已被 get 方法删除
        }
      })
    } catch (error) {
      console.error('清理失败:', error)
    }
  }
  
  // 大数据分片存储
  setLarge(key, value, expire) {
    const jsonStr = JSON.stringify(value)
    const chunkSize = 500 * 1024 // 500KB 一片
    const chunks = Math.ceil(jsonStr.length / chunkSize)
    
    // 存储分片信息
    this.set(`${key}_meta`, { chunks, expire })
    
    // 存储各个分片
    for (let i = 0; i < chunks; i++) {
      const chunk = jsonStr.slice(i * chunkSize, (i + 1) * chunkSize)
      uni.setStorageSync(`${this.prefix}${key}_${i}`, chunk)
    }
    
    return true
  }
  
  // 获取大数据
  getLarge(key) {
    const meta = this.get(`${key}_meta`)
    if (!meta) return null
    
    let jsonStr = ''
    for (let i = 0; i < meta.chunks; i++) {
      const chunk = uni.getStorageSync(`${this.prefix}${key}_${i}`)
      jsonStr += chunk
    }
    
    return JSON.parse(jsonStr)
  }
}

export default new Storage()

// 使用示例
import storage from '@/utils/storage'

// 普通存储
storage.set('userInfo', { name: 'Tom', age: 18 })

// 带过期时间（7天）
storage.set('token', 'xxx', 7 * 24 * 60 * 60 * 1000)

// 大数据存储
storage.set('bigData', largeArray) // 自动判断是否需要分片
```

**4. 不同平台的网络请求差异**

**问题描述：**

H5 环境下存在浏览器的同源策略限制，会导致跨域请求失败；而小程序和 App 环境没有这个限制。此外，不同平台的请求超时时间、并发限制也不同。

**原因分析：**

- 浏览器的同源策略：协议、域名、端口必须完全相同才能请求
- 小程序的网络请求通过原生能力实现，不受浏览器限制
- 微信小程序最多同时存在 10 个网络请求
- H5 的并发请求数取决于浏览器（通常是 6 个）

**解决方案：**

开发环境使用代理解决跨域，生产环境配置 CORS 或使用同域部署。封装请求方法统一处理平台差异。

```javascript
// 1. 配置开发代理（vue.config.js for Uniapp H5）
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'https://api.example.com', // 后端服务地址
        changeOrigin: true, // 改变请求头中的 origin
        pathRewrite: {
          '^/api': '' // 去掉 /api 前缀
        },
        // 配置 HTTPS
        secure: false
      }
    }
  }
}

// 2. 封装统一的请求方法
class Request {
  constructor() {
    this.baseURL = this.getBaseURL()
    this.timeout = 10000
    this.requestQueue = [] // 请求队列
    this.maxConcurrent = 10 // 最大并发数
  }
  
  // 根据平台获取 baseURL
  getBaseURL() {
    // #ifdef H5
    // H5 开发环境使用代理
    if (process.env.NODE_ENV === 'development') {
      return '/api'
    }
    // H5 生产环境使用完整 URL
    return 'https://api.example.com'
    // #endif
    
    // #ifndef H5
    // 小程序和 App 直接使用完整 URL
    return 'https://api.example.com'
    // #endif
  }
  
  // 请求方法
  request(options) {
    return new Promise((resolve, reject) => {
      // 检查并发数
      if (this.requestQueue.length >= this.maxConcurrent) {
        // 等待队列有空位
        const timer = setInterval(() => {
          if (this.requestQueue.length < this.maxConcurrent) {
            clearInterval(timer)
            this.doRequest(options, resolve, reject)
          }
        }, 100)
      } else {
        this.doRequest(options, resolve, reject)
      }
    })
  }
  
  // 执行请求
  doRequest(options, resolve, reject) {
    const requestId = Date.now() + Math.random()
    this.requestQueue.push(requestId)
    
    uni.request({
      url: this.baseURL + options.url,
      method: options.method || 'GET',
      data: options.data || {},
      header: this.getHeaders(options.header),
      timeout: this.timeout,
      
      success: (res) => {
        this.removeFromQueue(requestId)
        this.handleResponse(res, resolve, reject)
      },
      
      fail: (err) => {
        this.removeFromQueue(requestId)
        this.handleError(err, reject)
      }
    })
  }
  
  // 移除队列
  removeFromQueue(requestId) {
    const index = this.requestQueue.indexOf(requestId)
    if (index > -1) {
      this.requestQueue.splice(index, 1)
    }
  }
  
  // 获取请求头
  getHeaders(customHeader = {}) {
    const token = uni.getStorageSync('token')
    return {
      'Content-Type': 'application/json',
      'Authorization': token ? `Bearer ${token}` : '',
      ...customHeader
    }
  }
  
  // 处理响应
  handleResponse(res, resolve, reject) {
    const { statusCode, data } = res
    
    if (statusCode === 200) {
      if (data.code === 0) {
        resolve(data.data)
      } else {
        uni.showToast({
          title: data.message || '请求失败',
          icon: 'none'
        })
        reject(data)
      }
    } else if (statusCode === 401) {
      // 未登录，跳转登录页
      uni.reLaunch({ url: '/pages/login/index' })
      reject(res)
    } else {
      uni.showToast({
        title: `请求失败(${statusCode})`,
        icon: 'none'
      })
      reject(res)
    }
  }
  
  // 处理错误
  handleError(err, reject) {
    console.error('请求错误:', err)
    
    // 判断错误类型
    if (err.errMsg.includes('timeout')) {
      uni.showToast({
        title: '请求超时，请重试',
        icon: 'none'
      })
    } else if (err.errMsg.includes('fail')) {
      uni.showToast({
        title: '网络连接失败',
        icon: 'none'
      })
    }
    
    reject(err)
  }
}

export default new Request()

// 3. 生产环境后端配置 CORS（Node.js 示例）
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'https://your-h5-domain.com')
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE')
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization')
  res.header('Access-Control-Allow-Credentials', 'true')
  next()
})
```

**5. 不同平台的授权机制差异**

**问题描述：**

不同平台的用户授权流程和 API 完全不同，需要分别处理。微信小程序需要用户主动触发授权，支付宝小程序可以直接调用，H5 使用浏览器 API。

**原因分析：**

- 微信小程序：需要通过 button 组件的 open-type 触发授权
- 支付宝小程序：可以直接调用 my.authorize()
- H5：使用浏览器的 Geolocation API、Camera API 等
- 各平台的授权 scope 名称不同

**解决方案：**

封装统一的授权方法，内部根据平台调用不同的 API。对于敏感权限，提供友好的引导和说明。

```javascript
// 授权管理类
class AuthManager {
  // 权限映射表
  scopeMap = {
    location: {
      weapp: 'scope.userLocation',
      alipay: 'location',
      h5: 'geolocation'
    },
    camera: {
      weapp: 'scope.camera',
      alipay: 'camera',
      h5: 'camera'
    },
    album: {
      weapp: 'scope.writePhotosAlbum',
      alipay: 'album',
      h5: null
    }
  }
  
  // 请求授权
  async requestAuth(type) {
    const platform = this.getPlatform()
    const scope = this.scopeMap[type][platform]
    
    if (!scope) {
      throw new Error(`${platform} 平台不支持 ${type} 权限`)
    }
    
    try {
      // #ifdef MP-WEIXIN
      return await this.requestWeixinAuth(scope)
      // #endif
      
      // #ifdef MP-ALIPAY
      return await this.requestAlipayAuth(scope)
      // #endif
      
      // #ifdef H5
      return await this.requestH5Auth(type)
      // #endif
      
    } catch (error) {
      this.handleAuthError(type, error)
      throw error
    }
  }
  
  // 微信小程序授权
  async requestWeixinAuth(scope) {
    // 1. 先检查是否已授权
    const setting = await uni.getSetting()
    
    if (setting.authSetting[scope] === true) {
      return true // 已授权
    }
    
    if (setting.authSetting[scope] === false) {
      // 已拒绝，引导用户打开设置
      return await this.openSetting(scope)
    }
    
    // 2. 首次请求授权
    try {
      await uni.authorize({ scope })
      return true
    } catch (error) {
      // 用户拒绝授权
      return false
    }
  }
  
  // 支付宝小程序授权
  async requestAlipayAuth(scope) {
    return new Promise((resolve, reject) => {
      my.authorize({
        scopes: [scope],
        success: () => resolve(true),
        fail: (err) => {
          if (err.errorCode === 11) {
            // 用户拒绝授权
            resolve(false)
          } else {
            reject(err)
          }
        }
      })
    })
  }
  
  // H5 授权
  async requestH5Auth(type) {
    if (type === 'location') {
      return new Promise((resolve, reject) => {
        if (!navigator.geolocation) {
          reject(new Error('浏览器不支持定位'))
          return
        }
        
        navigator.geolocation.getCurrentPosition(
          () => resolve(true),
          (err) => {
            if (err.code === 1) {
              // 用户拒绝
              resolve(false)
            } else {
              reject(err)
            }
          }
        )
      })
    }
    
    if (type === 'camera') {
      try {
        await navigator.mediaDevices.getUserMedia({ video: true })
        return true
      } catch (error) {
        return false
      }
    }
    
    return false
  }
  
  // 打开设置页面
  async openSetting(scope) {
    return new Promise((resolve) => {
      uni.showModal({
        title: '需要授权',
        content: this.getAuthTip(scope),
        confirmText: '去设置',
        success: (res) => {
          if (res.confirm) {
            uni.openSetting({
              success: (setting) => {
                resolve(setting.authSetting[scope] === true)
              }
            })
          } else {
            resolve(false)
          }
        }
      })
    })
  }
  
  // 获取授权提示文案
  getAuthTip(scope) {
    const tips = {
      'scope.userLocation': '需要获取您的位置信息，用于提供更好的服务',
      'scope.camera': '需要使用您的相机，用于拍照或扫码',
      'scope.writePhotosAlbum': '需要保存图片到相册'
    }
    return tips[scope] || '需要您的授权才能继续'
  }
  
  // 处理授权错误
  handleAuthError(type, error) {
    console.error(`${type} 授权失败:`, error)
    
    uni.showToast({
      title: '授权失败，请重试',
      icon: 'none'
    })
  }
  
  // 获取平台
  getPlatform() {
    // #ifdef MP-WEIXIN
    return 'weapp'
    // #endif
    
    // #ifdef MP-ALIPAY
    return 'alipay'
    // #endif
    
    // #ifdef H5
    return 'h5'
    // #endif
  }
}

export default new AuthManager()

// 使用示例
import authManager from '@/utils/authManager'

export default {
  methods: {
    async getLocation() {
      try {
        // 请求定位权限
        const granted = await authManager.requestAuth('location')
        
        if (granted) {
          // 获取位置信息
          const location = await uni.getLocation({
            type: 'gcj02'
          })
          console.log('位置信息:', location)
        } else {
          uni.showToast({
            title: '需要定位权限',
            icon: 'none'
          })
        }
      } catch (error) {
        console.error('获取位置失败:', error)
      }
    }
  }
}
```

**6. 不同平台的键盘弹起处理**

**问题描述：**

iOS 和 Android 在键盘弹起时的表现完全不同。iOS 会将整个页面向上推，导致 fixed 定位的元素跟随移动；Android 会压缩页面高度，但不会自动滚动，可能导致输入框被遮挡。

**原因分析：**

- iOS WebView：键盘弹起时会改变 viewport 高度，整个页面向上移动
- Android WebView：键盘弹起时会压缩可视区域，但页面不会自动滚动
- 小程序的键盘处理由平台控制，开发者可控性有限
- H5 可以通过监听 resize 事件来检测键盘状态

**解决方案：**

针对不同平台采用不同的处理策略。iOS 需要在键盘收起后重置页面位置，Android 需要主动滚动到输入框位置。

```javascript
// 键盘管理类
class KeyboardManager {
  constructor() {
    this.isIOS = false
    this.isAndroid = false
    this.originalHeight = 0
    this.init()
  }
  
  init() {
    const systemInfo = uni.getSystemInfoSync()
    this.isIOS = systemInfo.platform === 'ios'
    this.isAndroid = systemInfo.platform === 'android'
    this.originalHeight = systemInfo.windowHeight
    
    // #ifdef H5
    // H5 监听窗口大小变化
    window.addEventListener('resize', this.handleResize.bind(this))
    // #endif
  }
  
  // 处理窗口大小变化
  handleResize() {
    const currentHeight = window.innerHeight
    
    if (currentHeight < this.originalHeight) {
      // 键盘弹起
      this.onKeyboardShow()
    } else {
      // 键盘收起
      this.onKeyboardHide()
    }
  }
  
  // 键盘弹起
  onKeyboardShow() {
    console.log('键盘弹起')
    
    // 可以在这里隐藏底部固定元素
    // 或调整页面布局
  }
  
  // 键盘收起
  onKeyboardHide() {
    console.log('键盘收起')
    
    if (this.isIOS) {
      // iOS 需要重置页面位置
      setTimeout(() => {
        window.scrollTo(0, 0)
        // 或使用 uni API
        uni.pageScrollTo({
          scrollTop: 0,
          duration: 0
        })
      }, 100)
    }
  }
  
  // 输入框聚焦处理
  handleInputFocus(e) {
    if (this.isAndroid) {
      // Android 需要主动滚动到输入框
      setTimeout(() => {
        // 方案1：使用 scrollIntoView
        e.target.scrollIntoView({
          behavior: 'smooth',
          block: 'center'
        })
        
        // 方案2：使用 uni API
        const query = uni.createSelectorQuery()
        query.select(`#${e.target.id}`).boundingClientRect()
        query.exec((res) => {
          if (res[0]) {
            const top = res[0].top
            uni.pageScrollTo({
              scrollTop: top - 100, // 留出一些空间
              duration: 300
            })
          }
        })
      }, 300) // 等待键盘弹起
    }
  }
  
  // 输入框失焦处理
  handleInputBlur() {
    if (this.isIOS) {
      // iOS 键盘收起后重置页面
      setTimeout(() => {
        window.scrollTo(0, 0)
      }, 100)
    }
  }
}

export default new KeyboardManager()

// 在页面中使用
import keyboardManager from '@/utils/keyboardManager'

export default {
  data() {
    return {
      showFooter: true // 控制底部元素显示
    }
  },
  
  methods: {
    // 输入框聚焦
    handleFocus(e) {
      keyboardManager.handleInputFocus(e)
      
      // 隐藏底部固定元素
      this.showFooter = false
    },
    
    // 输入框失焦
    handleBlur() {
      keyboardManager.handleInputBlur()
      
      // 显示底部固定元素
      this.showFooter = true
    }
  }
}
```

```vue
<!-- 模板中使用 -->
<template>
  <view class="page">
    <view class="content">
      <input
        type="text"
        placeholder="请输入内容"
        @focus="handleFocus"
        @blur="handleBlur"
      />
    </view>
    
    <!-- 底部固定元素 -->
    <view v-if="showFooter" class="footer">
      底部内容
    </view>
  </view>
</template>

<style>
.page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.content {
  flex: 1;
  padding: 20rpx;
}

.footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100rpx;
  background: #fff;
  /* iOS 安全区域适配 */
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
</style>
```

**7. 不同平台的图片处理差异**

**问题描述：**

不同平台对图片格式的支持程度不同，WebP 格式在小程序中支持良好，但在某些浏览器中不支持。此外，图片加载失败的处理、图片压缩、图片预览等功能在不同平台也有差异。

**原因分析：**

- WebP 格式：微信小程序支持，部分老旧浏览器不支持
- 图片大小限制：小程序单张图片不能超过 2MB
- 图片加载机制：小程序有图片缓存，H5 依赖浏览器缓存
- 图片预览：小程序有原生预览组件，H5 需要自己实现

**解决方案：**

根据平台能力选择合适的图片格式，提供降级方案。封装统一的图片处理工具类。

```javascript
// 图片管理类
class ImageManager {
  constructor() {
    this.supportsWebp = false
    this.checkWebpSupport()
  }
  
  // 检测 WebP 支持
  checkWebpSupport() {
    // #ifdef H5
    const canvas = document.createElement('canvas')
    if (canvas.getContext && canvas.getContext('2d')) {
      this.supportsWebp = canvas.toDataURL('image/webp').indexOf('data:image/webp') === 0
    }
    // #endif
    
    // #ifndef H5
    // 小程序默认支持 WebP
    this.supportsWebp = true
    // #endif
  }
  
  // 获取图片 URL（自动选择格式）
  getImageUrl(imageName, options = {}) {
    const {
      width,      // 宽度
      quality = 80, // 质量
      format      // 强制格式
    } = options
    
    const baseUrl = 'https://cdn.example.com/'
    
    // 确定图片格式
    let imageFormat = format
    if (!imageFormat) {
      imageFormat = this.supportsWebp ? 'webp' : 'jpg'
    }
    
    // 构建 URL（使用 CDN 图片处理参数）
    let url = `${baseUrl}${imageName}.${imageFormat}`
    
    // 添加处理参数
    const params = []
    if (width) {
      params.push(`w_${width}`)
    }
    if (quality) {
      params.push(`q_${quality}`)
    }
    
    if (params.length > 0) {
      url += `?x-oss-process=image/resize,${params.join(',')}`
    }
    
    return url
  }
  
  // 压缩图片
  async compressImage(filePath, options = {}) {
    const {
      quality = 80,
      maxWidth = 1920,
      maxHeight = 1920
    } = options
    
    return new Promise((resolve, reject) => {
      uni.compressImage({
        src: filePath,
        quality: quality,
        width: maxWidth,
        height: maxHeight,
        success: (res) => {
          resolve(res.tempFilePath)
        },
        fail: (err) => {
          // 压缩失败，返回原图
          console.warn('图片压缩失败，使用原图:', err)
          resolve(filePath)
        }
      })
    })
  }
  
  // 选择图片
  async chooseImage(options = {}) {
    const {
      count = 9,
      sizeType = ['compressed'],
      sourceType = ['album', 'camera'],
      compress = true
    } = options
    
    return new Promise((resolve, reject) => {
      uni.chooseImage({
        count: count,
        sizeType: sizeType,
        sourceType: sourceType,
        success: async (res) => {
          let tempFilePaths = res.tempFilePaths
          
          // 是否需要压缩
          if (compress) {
            tempFilePaths = await Promise.all(
              tempFilePaths.map(path => this.compressImage(path))
            )
          }
          
          resolve(tempFilePaths)
        },
        fail: reject
      })
    })
  }
  
  // 预览图片
  previewImage(current, urls) {
    uni.previewImage({
      current: current,
      urls: urls,
      // #ifdef H5
      // H5 可以添加长按保存功能
      longPressActions: {
        itemList: ['保存图片'],
        success: (data) => {
          if (data.tapIndex === 0) {
            this.saveImage(urls[data.index])
          }
        }
      }
      // #endif
    })
  }
  
  // 保存图片
  async saveImage(url) {
    try {
      // 1. 下载图片
      const downloadRes = await uni.downloadFile({ url })
      
      // 2. 保存到相册
      await uni.saveImageToPhotosAlbum({
        filePath: downloadRes.tempFilePath
      })
      
      uni.showToast({
        title: '保存成功',
        icon: 'success'
      })
      
    } catch (error) {
      if (error.errMsg.includes('auth')) {
        // 需要授权
        uni.showModal({
          title: '提示',
          content: '需要授权保存图片到相册',
          success: (res) => {
            if (res.confirm) {
              uni.openSetting()
            }
          }
        })
      } else {
        uni.showToast({
          title: '保存失败',
          icon: 'none'
        })
      }
    }
  }
  
  // 图片加载错误处理
  handleImageError(e, fallbackUrl) {
    // 尝试降级为 jpg 格式
    if (e.target.src.includes('.webp')) {
      e.target.src = e.target.src.replace('.webp', '.jpg')
    } else if (fallbackUrl) {
      // 使用备用图片
      e.target.src = fallbackUrl
    } else {
      // 使用默认占位图
      e.target.src = '/static/placeholder.png'
    }
  }
}

export default new ImageManager()

// 使用示例
import imageManager from '@/utils/imageManager'

export default {
  data() {
    return {
      imageUrl: '',
      imageList: []
    }
  },
  
  onLoad() {
    // 获取优化后的图片 URL
    this.imageUrl = imageManager.getImageUrl('product/123', {
      width: 750,
      quality: 80
    })
  },
  
  methods: {
    // 选择图片
    async handleChooseImage() {
      try {
        const images = await imageManager.chooseImage({
          count: 9,
          compress: true
        })
        this.imageList = images
      } catch (error) {
        console.error('选择图片失败:', error)
      }
    },
    
    // 预览图片
    handlePreview(index) {
      imageManager.previewImage(
        this.imageList[index],
        this.imageList
      )
    },
    
    // 图片加载失败
    handleImageError(e) {
      imageManager.handleImageError(e, '/static/default.jpg')
    }
  }
}
```

```vue
<!-- 模板中使用 -->
<template>
  <view>
    <!-- 显示图片 -->
    <image
      :src="imageUrl"
      mode="aspectFill"
      @error="handleImageError"
      @click="handlePreview(0)"
    />
    
    <!-- 图片列表 -->
    <view class="image-list">
      <image
        v-for="(url, index) in imageList"
        :key="index"
        :src="url"
        mode="aspectFill"
        @click="handlePreview(index)"
      />
    </view>
    
    <!-- 选择图片按钮 -->
    <button @click="handleChooseImage">选择图片</button>
  </view>
</template>
```

**8. 不同平台的视频播放差异**

**问题描述：**

不同平台的 video 组件属性和行为差异很大，特别是自动播放、全屏控制、播放控件等方面。iOS 对视频自动播放有严格限制，Android 相对宽松。

**原因分析：**

- iOS Safari 禁止视频自动播放（除非静音）
- 微信小程序的 video 组件是原生组件，层级最高
- H5 的 video 标签需要考虑浏览器兼容性
- 不同平台的视频格式支持不同（MP4、HLS、FLV）

**解决方案：**

使用条件编译为不同平台配置合适的属性，对于自动播放需求，可以引导用户手动触发。

```vue
<template>
  <view class="video-container">
    <!-- 微信小程序 -->
    <!-- #ifdef MP-WEIXIN -->
    <video
      :id="videoId"
      :src="videoUrl"
      :controls="true"
      :show-center-play-btn="true"
      :enable-progress-gesture="true"
      :show-fullscreen-btn="true"
      :object-fit="objectFit"
      @play="handlePlay"
      @pause="handlePause"
      @ended="handleEnded"
      @error="handleError"
    />
    <!-- #endif -->
    
    <!-- H5 -->
    <!-- #ifdef H5 -->
    <video
      :id="videoId"
      :src="videoUrl"
      controls
      playsinline
      webkit-playsinline
      x5-playsinline
      :muted="autoplay"
      @play="handlePlay"
      @pause="handlePause"
      @ended="handleEnded"
      @error="handleError"
    />
    <!-- #endif -->
    
    <!-- App -->
    <!-- #ifdef APP-PLUS -->
    <video
      :id="videoId"
      :src="videoUrl"
      :controls="true"
      :enable-play-gesture="true"
      :show-fullscreen-btn="true"
      @play="handlePlay"
      @pause="handlePause"
      @ended="handleEnded"
      @error="handleError"
    />
    <!-- #endif -->
  </view>
</template>

<script>
export default {
  data() {
    return {
      videoId: 'myVideo',
      videoUrl: '',
      videoContext: null,
      autoplay: false,
      objectFit: 'contain' // contain | fill | cover
    }
  },
  
  onReady() {
    // 创建 video 上下文
    this.videoContext = uni.createVideoContext(this.videoId, this)
  },
  
  methods: {
    // 播放视频
    playVideo() {
      if (this.videoContext) {
        this.videoContext.play()
      }
    },
    
    // 暂停视频
    pauseVideo() {
      if (this.videoContext) {
        this.videoContext.pause()
      }
    },
    
    // 跳转到指定位置
    seekVideo(position) {
      if (this.videoContext) {
        this.videoContext.seek(position)
      }
    },
    
    // 全屏播放
    requestFullScreen() {
      if (this.videoContext) {
        this.videoContext.requestFullScreen({
          direction: 0 // 0: 正常竖向, 90: 屏幕逆时针90度, -90: 屏幕顺时针90度
        })
      }
    },
    
    // 退出全屏
    exitFullScreen() {
      if (this.videoContext) {
        this.videoContext.exitFullScreen()
      }
    },
    
    // 播放事件
    handlePlay() {
      console.log('视频开始播放')
    },
    
    // 暂停事件
    handlePause() {
      console.log('视频暂停')
    },
    
    // 播放结束
    handleEnded() {
      console.log('视频播放结束')
      // 可以在这里播放下一个视频或显示推荐
    },
    
    // 错误处理
    handleError(e) {
      console.error('视频播放错误:', e)
      
      uni.showToast({
        title: '视频加载失败',
        icon: 'none'
      })
      
      // 尝试降级处理
      this.handleVideoError(e)
    },
    
    // 视频错误降级
    handleVideoError(e) {
      // 如果是 HLS 格式不支持，尝试 MP4
      if (this.videoUrl.includes('.m3u8')) {
        this.videoUrl = this.videoUrl.replace('.m3u8', '.mp4')
      }
    }
  },
  
  onUnload() {
    // 页面卸载时停止播放
    if (this.videoContext) {
      this.videoContext.stop()
    }
  }
}
</script>

<style scoped>
.video-container {
  width: 100%;
  height: 400rpx;
  background: #000;
}

video {
  width: 100%;
  height: 100%;
}
</style>
```

**9. 不同平台的日期处理差异**

**问题描述：**

iOS 的 JavaScript 引擎对日期字符串的解析非常严格，不支持 `YYYY-MM-DD` 格式，必须使用 `YYYY/MM/DD` 格式。这会导致在 iOS 设备上出现 `Invalid Date` 错误。

**原因分析：**

- iOS Safari 使用 WebKit 引擎，对日期格式要求严格
- Android 和 Chrome 对日期格式比较宽容
- 不同平台的时区处理也可能不同
- 小程序的日期 API 与浏览器有差异

**解决方案：**

统一使用标准的日期格式，或使用第三方日期库（如 dayjs）来处理日期。

```javascript
// 日期工具类
class DateUtil {
  // 解析日期字符串（兼容 iOS）
  parseDate(dateString) {
    if (!dateString) return null
    
    // 将 YYYY-MM-DD 转换为 YYYY/MM/DD
    const formattedDate = dateString.replace(/-/g, '/')
    
    // 创建日期对象
    const date = new Date(formattedDate)
    
    // 检查是否有效
    if (isNaN(date.getTime())) {
      console.error('无效的日期:', dateString)
      return null
    }
    
    return date
  }
  
  // 格式化日期
  formatDate(date, format = 'YYYY-MM-DD') {
    if (!date) return ''
    
    // 确保是 Date 对象
    if (typeof date === 'string') {
      date = this.parseDate(date)
    }
    
    if (!date) return ''
    
    const year = date.getFullYear()
    const month = String(date.getMonth() + 1).padStart(2, '0')
    const day = String(date.getDate()).padStart(2, '0')
    const hour = String(date.getHours()).padStart(2, '0')
    const minute = String(date.getMinutes()).padStart(2, '0')
    const second = String(date.getSeconds()).padStart(2, '0')
    
    return format
      .replace('YYYY', year)
      .replace('MM', month)
      .replace('DD', day)
      .replace('HH', hour)
      .replace('mm', minute)
      .replace('ss', second)
  }
  
  // 获取相对时间
  getRelativeTime(date) {
    if (typeof date === 'string') {
      date = this.parseDate(date)
    }
    
    if (!date) return ''
    
    const now = new Date()
    const diff = now.getTime() - date.getTime()
    
    const minute = 60 * 1000
    const hour = 60 * minute
    const day = 24 * hour
    const month = 30 * day
    const year = 365 * day
    
    if (diff < minute) {
      return '刚刚'
    } else if (diff < hour) {
      return Math.floor(diff / minute) + '分钟前'
    } else if (diff < day) {
      return Math.floor(diff / hour) + '小时前'
    } else if (diff < month) {
      return Math.floor(diff / day) + '天前'
    } else if (diff < year) {
      return Math.floor(diff / month) + '个月前'
    } else {
      return Math.floor(diff / year) + '年前'
    }
  }
  
  // 日期选择器值转换
  datePickerValue(date) {
    if (typeof date === 'string') {
      date = this.parseDate(date)
    }
    
    if (!date) {
      date = new Date()
    }
    
    // 小程序日期选择器需要 YYYY-MM-DD 格式
    return this.formatDate(date, 'YYYY-MM-DD')
  }
  
  // 时间戳转日期
  timestampToDate(timestamp) {
    // 处理秒级时间戳
    if (timestamp < 10000000000) {
      timestamp = timestamp * 1000
    }
    
    return new Date(timestamp)
  }
}

export default new DateUtil()

// 使用示例
import dateUtil from '@/utils/dateUtil'

export default {
  data() {
    return {
      dateValue: '',
      displayDate: ''
    }
  },
  
  onLoad() {
    // 从后端获取的日期字符串
    const dateString = '2024-01-01 12:00:00'
    
    // ❌ 错误：直接使用（iOS 会报错）
    // const date = new Date(dateString)
    
    // ✅ 正确：使用工具类解析
    const date = dateUtil.parseDate(dateString)
    
    // 格式化显示
    this.displayDate = dateUtil.formatDate(date, 'YYYY年MM月DD日')
    
    // 日期选择器初始值
    this.dateValue = dateUtil.datePickerValue(date)
  },
  
  methods: {
    // 日期选择
    handleDateChange(e) {
      const dateString = e.detail.value
      const date = dateUtil.parseDate(dateString)
      
      this.displayDate = dateUtil.formatDate(date, 'YYYY年MM月DD日')
    }
  }
}

// 或使用第三方库 dayjs（推荐）
import dayjs from 'dayjs'

export default {
  methods: {
    formatDate() {
      // dayjs 自动处理各种格式
      const date = dayjs('2024-01-01')
      
      // 格式化
      console.log(date.format('YYYY年MM月DD日'))
      
      // 相对时间
      console.log(date.fromNow())
      
      // 日期计算
      console.log(date.add(7, 'day').format('YYYY-MM-DD'))
    }
  }
}
```

**10. 不同平台的分享功能差异**

**问题描述：**

不同平台的分享机制完全不同。微信小程序有专门的分享 API，H5 可以使用 Web Share API 或自定义分享，App 需要调用原生分享能力。

**原因分析：**

- 微信小程序：通过 `onShareAppMessage` 和 `onShareTimeline` 实现
- H5：部分浏览器支持 Web Share API，否则需要自己实现
- App：需要调用原生的分享功能
- 各平台的分享参数格式不同

**解决方案：**

封装统一的分享方法，根据平台调用不同的 API。提供降级方案（如复制链接）。

```javascript
// 分享管理类
class ShareManager {
  // 分享到好友/群聊
  shareToFriend(options = {}) {
    const {
      title = '分享标题',
      desc = '分享描述',
      path = '/pages/index/index',
      imageUrl = ''
    } = options
    
    // #ifdef MP-WEIXIN
    // 微信小程序在页面中使用 onShareAppMessage
    return {
      title: title,
      path: path,
      imageUrl: imageUrl
    }
    // #endif
    
    // #ifdef H5
    // H5 使用 Web Share API 或降级方案
    return this.shareH5(options)
    // #endif
    
    // #ifdef APP-PLUS
    // App 使用原生分享
    return this.shareApp(options)
    // #endif
  }
  
  // H5 分享
  async shareH5(options) {
    const { title, desc, url = window.location.href } = options
    
    // 检查是否支持 Web Share API
    if (navigator.share) {
      try {
        await navigator.share({
          title: title,
          text: desc,
          url: url
        })
        return true
      } catch (error) {
        if (error.name !== 'AbortError') {
          console.error('分享失败:', error)
        }
        return false
      }
    } else {
      // 降级方案：复制链接
      return this.copyLink(url)
    }
  }
  
  // App 分享
  shareApp(options) {
    const { title, desc, imageUrl } = options
    
    // #ifdef APP-PLUS
    plus.share.sendWithSystem({
      type: 'text',
      content: `${title}\n${desc}`,
      href: imageUrl
    }, (res) => {
      console.log('分享成功')
    }, (err) => {
      console.error('分享失败:', err)
    })
    // #endif
  }
  
  // 复制链接
  copyLink(url) {
    return new Promise((resolve) => {
      uni.setClipboardData({
        data: url,
        success: () => {
          uni.showToast({
            title: '链接已复制',
            icon: 'success'
          })
          resolve(true)
        },
        fail: () => {
          uni.showToast({
            title: '复制失败',
            icon: 'none'
          })
          resolve(false)
        }
      })
    })
  }
  
  // 生成分享海报
  async generatePoster(options = {}) {
    const {
      title = '分享标题',
      desc = '分享描述',
      imageUrl = '',
      qrcodeUrl = ''
    } = options
    
    return new Promise((resolve) => {
      const query = uni.createSelectorQuery()
      query.select('#posterCanvas')
        .fields({ node: true, size: true })
        .exec((res) => {
          const canvas = res[0].node
          const ctx = canvas.getContext('2d')
          
          // 设置画布尺寸
          const dpr = uni.getSystemInfoSync().pixelRatio
          canvas.width = 375 * dpr
          canvas.height = 667 * dpr
          ctx.scale(dpr, dpr)
          
          // 绘制背景
          ctx.fillStyle = '#fff'
          ctx.fillRect(0, 0, 375, 667)
          
          // 绘制标题
          ctx.fillStyle = '#000'
          ctx.font = 'bold 20px sans-serif'
          ctx.fillText(title, 20, 50)
          
          // 绘制描述
          ctx.fillStyle = '#666'
          ctx.font = '14px sans-serif'
          this.drawText(ctx, desc, 20, 80, 335, 20)
          
          // 绘制图片
          if (imageUrl) {
            const img = canvas.createImage()
            img.onload = () => {
              ctx.drawImage(img, 20, 120, 335, 200)
              this.drawQrcode(canvas, ctx, qrcodeUrl, resolve)
            }
            img.src = imageUrl
          } else {
            this.drawQrcode(canvas, ctx, qrcodeUrl, resolve)
          }
        })
    })
  }
  
  // 绘制二维码
  drawQrcode(canvas, ctx, qrcodeUrl, resolve) {
    if (qrcodeUrl) {
      const qrcode = canvas.createImage()
      qrcode.onload = () => {
        ctx.drawImage(qrcode, 275, 567, 80, 80)
        this.exportPoster(canvas, resolve)
      }
      qrcode.src = qrcodeUrl
    } else {
      this.exportPoster(canvas, resolve)
    }
  }
  
  // 导出海报
  exportPoster(canvas, resolve) {
    uni.canvasToTempFilePath({
      canvas: canvas,
      success: (res) => {
        resolve(res.tempFilePath)
      }
    })
  }
  
  // 绘制多行文本
  drawText(ctx, text, x, y, maxWidth, lineHeight) {
    const words = text.split('')
    let line = ''
    let currentY = y
    
    for (let i = 0; i < words.length; i++) {
      const testLine = line + words[i]
      const metrics = ctx.measureText(testLine)
      
      if (metrics.width > maxWidth && i > 0) {
        ctx.fillText(line, x, currentY)
        line = words[i]
        currentY += lineHeight
      } else {
        line = testLine
      }
    }
    
    ctx.fillText(line, x, currentY)
  }
}

export default new ShareManager()

// 在页面中使用
import shareManager from '@/utils/shareManager'

export default {
  // 微信小程序分享
  onShareAppMessage() {
    return shareManager.shareToFriend({
      title: '这是一个很棒的页面',
      path: '/pages/index/index?id=123',
      imageUrl: 'https://cdn.example.com/share.jpg'
    })
  },
  
  // 分享到朋友圈
  onShareTimeline() {
    return {
      title: '分享到朋友圈',
      query: 'id=123',
      imageUrl: 'https://cdn.example.com/share.jpg'
    }
  },
  
  methods: {
    // 主动分享（H5/App）
    async handleShare() {
      const success = await shareManager.shareToFriend({
        title: '这是一个很棒的页面',
        desc: '快来看看吧',
        url: window.location.href
      })
      
      if (success) {
        console.log('分享成功')
      }
    },
    
    // 生成分享海报
    async handleGeneratePoster() {
      uni.showLoading({ title: '生成中...' })
      
      const posterPath = await shareManager.generatePoster({
        title: '分享标题',
        desc: '分享描述',
        imageUrl: 'https://cdn.example.com/image.jpg',
        qrcodeUrl: 'https://cdn.example.com/qrcode.jpg'
      })
      
      uni.hideLoading()
      
      // 预览海报
      uni.previewImage({
        urls: [posterPath]
      })
    }
  }
}
```

### 如何进行调试和错误监控？

**Taro 调试方法：**

```javascript
// 1. 开发环境调试
// 使用 console.log
console.log('调试信息', data)

// 使用 Taro.showToast 快速调试
Taro.showToast({
  title: JSON.stringify(data),
  icon: 'none',
  duration: 3000
})

// 2. 真机调试
// 微信小程序：使用微信开发者工具的真机调试功能
// H5：使用 Chrome DevTools 的远程调试

// 3. 错误监控
class ErrorMonitor {
  constructor() {
    this.init()
  }
  
  init() {
    // 监听全局错误
    Taro.onError((error) => {
      this.reportError({
        type: 'runtime',
        message: error,
        stack: error.stack
      })
    })
    
    // 监听未处理的 Promise 错误
    Taro.onUnhandledRejection((error) => {
      this.reportError({
        type: 'promise',
        message: error.reason
      })
    })
  }
  
  // 上报错误
  reportError(error) {
    const systemInfo = Taro.getSystemInfoSync()
    
    Taro.request({
      url: 'https://api.example.com/error/report',
      method: 'POST',
      data: {
        ...error,
        platform: process.env.TARO_ENV,
        system: systemInfo.system,
        version: systemInfo.version,
        timestamp: Date.now()
      }
    })
  }
  
  // 手动上报错误
  captureError(error) {
    this.reportError({
      type: 'manual',
      message: error.message,
      stack: error.stack
    })
  }
}

export default new ErrorMonitor()

// 使用
import errorMonitor from '@/utils/errorMonitor'

try {
  // 业务代码
} catch (error) {
  errorMonitor.captureError(error)
}
```

**Uniapp 调试方法：**

```javascript
// 1. 条件编译调试
// #ifdef H5
console.log('H5 环境调试')
// #endif

// #ifdef MP-WEIXIN
console.log('微信小程序调试')
// #endif

// 2. vconsole 调试（H5）
// main.js
// #ifdef H5
import VConsole from 'vconsole'
if (process.env.NODE_ENV === 'development') {
  new VConsole()
}
// #endif

// 3. 错误监控
// App.vue
export default {
  onLaunch() {
    this.initErrorMonitor()
  },
  
  onError(err) {
    this.reportError({
      type: 'app',
      message: err
    })
  },
  
  methods: {
    initErrorMonitor() {
      // 监听页面错误
      uni.onError((error) => {
        this.reportError({
          type: 'runtime',
          message: error
        })
      })
      
      // 监听网络状态
      uni.onNetworkStatusChange((res) => {
        if (!res.isConnected) {
          uni.showToast({
            title: '网络已断开',
            icon: 'none'
          })
        }
      })
    },
    
    reportError(error) {
      const systemInfo = uni.getSystemInfoSync()
      
      uni.request({
        url: 'https://api.example.com/error/report',
        method: 'POST',
        data: {
          ...error,
          platform: systemInfo.platform,
          system: systemInfo.system,
          version: systemInfo.version,
          timestamp: Date.now()
        }
      })
    }
  }
}

// 4. 性能监控
class PerformanceMonitor {
  constructor() {
    this.startTime = Date.now()
  }
  
  // 记录页面加载时间
  recordPageLoad(pageName) {
    const loadTime = Date.now() - this.startTime
    this.report({
      type: 'page_load',
      page: pageName,
      duration: loadTime
    })
  }
  
  // 记录接口请求时间
  recordApiCall(apiName, duration) {
    this.report({
      type: 'api_call',
      api: apiName,
      duration: duration
    })
  }
  
  report(data) {
    uni.request({
      url: 'https://api.example.com/performance/report',
      method: 'POST',
      data: data
    })
  }
}

export default new PerformanceMonitor()
```

## 八、项目实战总结

### Taro/Uniapp 项目开发的最佳实践有哪些？

**1. 项目结构规范**

```
project/
├── src/
│   ├── pages/              # 页面
│   │   ├── index/
│   │   └── user/
│   ├── components/         # 组件
│   │   ├── common/        # 通用组件
│   │   └── business/      # 业务组件
│   ├── utils/             # 工具函数
│   │   ├── request.js     # 网络请求
│   │   ├── storage.js     # 本地存储
│   │   └── platform.js    # 平台判断
│   ├── api/               # API 接口
│   ├── store/             # 状态管理
│   ├── styles/            # 全局样式
│   ├── config/            # 配置文件
│   └── static/            # 静态资源
├── config/                # 编译配置
└── package.json
```

**2. 代码规范**

```javascript
// 使用 ESLint + Prettier
// .eslintrc.js
module.exports = {
  extends: ['taro/react', 'prettier'],
  rules: {
    'react/jsx-uses-react': 'off',
    'react/react-in-jsx-scope': 'off'
  }
}

// .prettierrc
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "none"
}
```

**3. 性能优化清单**

- [ ] 使用分包加载
- [ ] 图片懒加载和压缩
- [ ] 长列表虚拟滚动
- [ ] 防抖节流
- [ ] 请求合并和缓存
- [ ] 骨架屏
- [ ] 按需加载组件
- [ ] 避免频繁 setData

**4. 兼容性处理**

```javascript
// 统一的平台判断
export const platform = {
  isWeapp: process.env.TARO_ENV === 'weapp',
  isAlipay: process.env.TARO_ENV === 'alipay',
  isH5: process.env.TARO_ENV === 'h5',
  isApp: process.env.TARO_ENV === 'rn'
}

// 统一的 API 封装
export const api = {
  showToast(title) {
    if (platform.isWeapp) {
      wx.showToast({ title, icon: 'none' })
    } else if (platform.isAlipay) {
      my.showToast({ content: title })
    } else if (platform.isH5) {
      alert(title)
    }
  }
}
```

**5. 安全规范**

```javascript
// 敏感信息加密存储
import CryptoJS from 'crypto-js'

const SECRET_KEY = 'your-secret-key'

export const secureStorage = {
  set(key, value) {
    const encrypted = CryptoJS.AES.encrypt(
      JSON.stringify(value),
      SECRET_KEY
    ).toString()
    Taro.setStorageSync(key, encrypted)
  },
  
  get(key) {
    const encrypted = Taro.getStorageSync(key)
    if (!encrypted) return null
    
    const decrypted = CryptoJS.AES.decrypt(encrypted, SECRET_KEY)
    return JSON.parse(decrypted.toString(CryptoJS.enc.Utf8))
  }
}

// 防止 XSS 攻击
export function escapeHtml(str) {
  const map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#x27;',
    '/': '&#x2F;'
  }
  return str.replace(/[&<>"'/]/g, (char) => map[char])
}
```

**6. 测试规范**

```javascript
// 使用 Jest 进行单元测试
// __tests__/utils.test.js
import { formatDate } from '@/utils/date'

describe('formatDate', () => {
  test('should format date correctly', () => {
    const date = new Date('2024-01-01')
    expect(formatDate(date)).toBe('2024-01-01')
  })
})
```

**7. 发布流程**

```bash
# 1. 代码检查
npm run lint

# 2. 运行测试
npm run test

# 3. 构建生产版本
npm run build:weapp
npm run build:h5

# 4. 上传代码
# 微信小程序：使用微信开发者工具上传
# H5：部署到服务器

# 5. 提交审核
# 在对应平台的管理后台提交审核
```

### 总结：Taro 和 Uniapp 的选型建议

**选择 Taro 的理由：**

1. 团队使用 React 技术栈
2. 需要使用 React 生态的库和工具
3. 对 JSX 语法有偏好
4. 需要更灵活的组件化方案
5. 有京东团队背书，社区活跃

**选择 Uniapp 的理由：**

1. 团队使用 Vue 技术栈
2. 需要快速开发，使用现成模板
3. 对性能要求较高
4. 需要丰富的插件市场支持
5. HBuilderX 集成开发体验好

**共同点：**

1. 都支持多端开发，一套代码多端运行
2. 都有完善的文档和社区支持
3. 都支持主流的小程序平台和 H5
4. 都有丰富的组件库和工具链

**最终建议：**

根据团队技术栈和项目需求选择，没有绝对的优劣之分。如果团队熟悉 React，选择 Taro；如果团队熟悉 Vue，选择 Uniapp。两者都能很好地完成跨端开发任务。



