[toc]



## 一、基础认知

### 说说你对 Vue 的理解？

Vue 是一套用于构建用户界面的渐进式 JavaScript 框架，核心特点是响应式数据驱动和组件化开发。

面试回答时可以从几个角度展开：

1. 渐进式：既可以只把它当成视图层库使用，也可以结合路由、状态管理、工程化方案构建完整单页应用。
2. 响应式：数据变化会自动驱动视图更新，开发者更多关注状态本身，而不是手动操作 DOM。
3. 组件化：把页面拆成可复用、可组合、可维护的组件，便于协作和复杂系统拆分。
4. 生态完善：Vue Router、Pinia、构建工具、SSR 方案比较成熟，适合中后台和业务系统开发。
5. 上手成本低：模板语法直观，文档完整，适合团队落地。

如果再深入一点，可以补充 Vue 的本质是“声明式 UI + 组件化 + 响应式更新机制”。

### 说说你对 SPA（单页应用）的理解？

SPA 指 Single Page Application，整个应用只有一个 HTML 页面，页面切换主要通过前端路由完成，数据交互通过 Ajax 或 Fetch 与后端通信。

它的优点：

1. 页面切换体验好，接近原生应用。
2. 前后端职责分离更清晰。
3. 公共资源只加载一次，后续切换成本较低。

它的缺点：

1. 首屏加载可能较慢。
2. SEO 天然不友好。
3. 前端路由 history 模式部署时，刷新可能出现 404。
4. 项目复杂度更高，对工程化要求更强。

所以 SPA 更适合交互复杂、后台管理类、业务流程型系统。

### 说说你对双向绑定的理解？

双向绑定本质上是“数据变化驱动视图更新，用户在视图中的输入变化也能同步回数据”。在 Vue 中常见表现是 `v-model`。

严格来说，Vue 并不是任意场景下的绝对双向绑定，而是：

1. 数据到视图：依赖响应式系统完成自动更新。
2. 视图到数据：通过事件监听，比如 `input`、`change`，把新值再写回数据。

在 Vue2 中，通常是 `value + input` 的组合；在 Vue3 中，组件上 `v-model` 对应的是 `modelValue + update:modelValue`。

所以面试中最好表述为：Vue 的双向绑定是语法层面的封装，底层仍然是单向数据流加事件回传。

## 二、生命周期与响应式

### Vue 实例挂载的过程中发生了什么？

Vue 挂载过程可以概括为：

1. 初始化实例，合并配置项。
2. 初始化生命周期、事件、数据代理、响应式数据。
3. 调用 `beforeCreate`。
4. 完成 `data`、`computed`、`watch`、`methods` 等初始化后调用 `created`。
5. 如果存在 `el` 或手动调用 `$mount`，开始编译模板。
6. 把 template/render 转成 render 函数。
7. 执行 render，生成虚拟 DOM。
8. 通过 patch 生成真实 DOM 并挂载到页面。
9. 挂载完成后调用 `mounted`。

核心可以归纳为：初始化数据、生成虚拟 DOM、创建真实 DOM、完成挂载。

### 说说你对 Vue 生命周期的理解？

生命周期指 Vue 实例或组件从创建到销毁的整个过程里，会在特定阶段暴露钩子函数，方便开发者插入业务逻辑。

常见生命周期可以这样理解：

1. `beforeCreate`：实例初始化后，数据观测和事件还没完成。
2. `created`：数据、方法、计算属性已可用，常用于发起请求、初始化数据。
3. `beforeMount`：模板编译完成，真实 DOM 挂载前。
4. `mounted`：组件已经渲染到页面，适合做 DOM 操作、第三方库初始化。
5. `beforeUpdate`：数据更新后，DOM 重新渲染前。
6. `updated`：DOM 更新完成后。
7. `beforeDestroy`/`beforeUnmount`：组件销毁前，适合清理定时器、事件监听、订阅。
8. `destroyed`/`unmounted`：组件已经销毁。
9. `activated` / `deactivated`：配合 `keep-alive` 使用的缓存组件生命周期。

面试时要强调：异步请求通常放在 `created` 或 `mounted`，DOM 依赖操作一般放在 `mounted`。

**为什么这么分？** 关键在于"这个钩子里 DOM 是否已经存在"：

1. **`created` 阶段**：data、methods、computed 都已就绪，但模板还没编译成真实 DOM。这时发请求的好处是更早启动网络往返——网络是 IO，请求和后续的模板编译可以并行，不阻塞渲染。坏处是不能访问 `this.$el` 或 `$refs`。
2. **`mounted` 阶段**：真实 DOM 已经挂载到页面。这时发请求略晚于 `created`，但能访问 DOM；如果请求结果要立刻操作 DOM（比如根据数据计算图表尺寸），放在 `mounted` 更稳。
3. **DOM 操作必须在 `mounted`**：在 `created` 里访问 `$refs` 拿到的是 `undefined`，会直接报错。第三方库（如 ECharts、Swiper）的初始化也必须等 DOM 就位，所以一律放 `mounted`。

简单选型：纯请求选 `created`（更早），请求 + DOM 操作选 `mounted`（更稳）。SSR 项目还要注意，`mounted` 只在客户端执行，服务端只跑到 `created`，所以服务端预取数据要走 `created` 或专用的 `serverPrefetch`。

### 为什么 data 属性是一个函数而不是一个对象？

这是组件中的要求，目的是避免多个组件实例共享同一份数据。

如果 `data` 直接写成对象，那么组件每次复用时拿到的是同一个对象引用，一个实例修改数据，其他实例也会受影响。

把 `data` 定义成函数后，每次创建组件实例都会返回一个全新的对象，这样每个组件实例的数据都是独立的。

根实例可以是对象，因为根实例通常只创建一次，不存在复用问题。

### computed 和 watch 有什么区别？分别适合什么场景？

两者都和响应式数据有关，但职责不同。

1. `computed` 更适合基于已有状态推导出新值，本质上是派生状态。
2. `watch` 更适合监听某个数据变化后执行副作用逻辑。

常见区别：

1. `computed` 有缓存，依赖不变时会复用结果。
2. `watch` 没有缓存，监听到变化就执行回调。
3. `computed` 更适合模板绑定和数据计算。
4. `watch` 更适合异步请求、日志记录、联动处理等副作用。

一句话总结：能用 `computed` 解决的，通常不要用 `watch` 去写纯计算逻辑。

**为什么这条建议有意义？** 三方面原因：

1. **缓存差异**：`computed` 在依赖未变时会复用上次结果，多次访问也只算一次；`watch` 每次依赖变化都会触发回调，没有缓存，自己写的派生逻辑可能反复跑。
2. **声明式 vs 命令式**：`computed` 是"我要什么值，怎么从已有数据算"——纯函数式表达，模板里直接 `{{ fullName }}` 就能用；`watch` 是"X 变了我要做什么"——本质是命令式副作用，要手动赋值给另一个 data，再让模板使用，链路绕一层。
3. **数据流可追踪性**：`computed` 的依赖关系是隐式自动追踪的，DevTools 里能直接看出"它依赖了 firstName 和 lastName"；用 `watch` 派生数据则要靠人脑串联"watch firstName → 改 fullName"，重构时容易遗漏。

举个反例就一目了然：

```js
// ❌ 不推荐：用 watch 派生
data() { return { firstName: '', lastName: '', fullName: '' } },
watch: {
  firstName() { this.fullName = this.firstName + ' ' + this.lastName },
  lastName() { this.fullName = this.firstName + ' ' + this.lastName },
}

// ✅ 推荐：用 computed
computed: {
  fullName() { return this.firstName + ' ' + this.lastName }
}
```

watch 的版本要写两个监听器，逻辑重复，且一旦再加个 `middleName` 就要再加一个 watch；computed 只需要一行。**watch 的真正用武之地是"数据变化后的副作用"**——发请求、写日志、操作非响应式资源，这些是 computed 做不到的。

## 三、模板、指令与渲染机制

### Vue 中的 v-show 和 v-if 怎么理解？

两者都能控制元素显示隐藏，但原理不同：

1. `v-if` 是“真正的条件渲染”，条件不成立时不会渲染对应节点，会触发组件的创建和销毁。
2. `v-show` 是通过切换元素的 `display: none` 控制显示隐藏，DOM 一直存在。

使用建议：

1. 条件很少变化，用 `v-if`，因为初始不渲染更省资源。
2. 频繁切换显示状态，用 `v-show`，因为不需要反复销毁和创建 DOM。

一句话总结：`v-if` 更适合低频切换，`v-show` 更适合高频切换。

**为什么频率是分水岭？** 两者的开销分布不同：

1. **`v-if` 开销集中在切换时**：每次条件变化都会真正创建/销毁 DOM 和组件实例——子组件的 `created`、`mounted`、`unmounted` 全套生命周期都会触发，事件监听、定时器、watcher 都要重建。如果切换频率高，反复 mount/unmount 的成本会非常可观。
2. **`v-show` 开销集中在初次渲染**：DOM 一开始就创建好，之后只是切换 `display: none` 和原始值——浏览器的样式重计算和 paint，比创建销毁节点便宜得多。代价是 DOM 一直占内存，初始首屏会更重。

具体例子：

1. 详情页 Tab 切换、抽屉显隐、菜单展开：用户操作密集，DOM 复用价值高 → `v-show`。
2. 路由级条件渲染、登录后才显示的整块区域、错误页：基本不切换，初始能省一份 DOM 树 → `v-if`。
3. 含重型子组件（图表、富文本）的区域：组件初始化本身慢，频繁 mount 卡顿明显，更要用 `v-show` 或 `keep-alive`。

还有两点容易忽略：`v-show` 不能用在 `<template>` 上（它需要一个真实根元素挂 style）；`v-show` 切换时不会触发任何生命周期，所以"想在显示时拉数据"的场景必须用 `v-if` 或自己 watch。

### 为什么 Vue 中的 v-if 和 v-for 不建议一起用？

因为 `v-for` 的优先级通常高于 `v-if`，会先遍历再判断，相当于每次都先循环整份列表，再对每一项做条件判断，性能较差，而且语义不清晰。

更好的做法有两个：

1. 在计算属性中先把需要展示的数据过滤好，再进行 `v-for`。
2. 把 `v-if` 提到外层容器，减少无意义的循环。

本质原因不是不能用，而是不推荐把数据筛选逻辑直接写在模板里。

**展开说明：**

1. **Vue 2 和 Vue 3 优先级正好反过来**，这点是常见坑：
   - Vue 2：`v-for` 优先级高于 `v-if`，写在同一节点上等价于"循环每一项再判断显示"，逻辑能跑但性能差。
   - Vue 3：`v-if` 优先级高于 `v-for`，这意味着 `v-if` 里读不到 `v-for` 的循环变量（如 `item`），直接报错或语义错误。
2. **性能损耗的具体来源**：假设有 1000 项列表，活跃项只有 10 个。`v-for` + `v-if` 的写法每次更新都要遍历 1000 次再判断；如果先用 `computed` 算出 `activeList` 再渲染，只遍历活跃的 10 项，diff 也只看 10 个节点。
3. **可读性**：把过滤逻辑写到 `computed` 里，命名清晰（`activeUsers`、`pendingOrders`），模板专注于"展示什么"——这是 Vue 提倡的"模板尽量声明式"原则的具体落地。
4. **测试友好**：`computed` 是纯函数，可以单独测试过滤逻辑；模板里的 `v-if="item.status === 'active'"` 是无法被单元测试覆盖的。

```vue
<!-- ❌ Vue 2 能跑但性能差；Vue 3 直接报错（v-if 拿不到 user） -->
<li v-for="user in users" v-if="user.active" :key="user.id">{{ user.name }}</li>

<!-- ✅ 推荐：用 computed 过滤 -->
<script>
computed: { activeUsers() { return this.users.filter(u => u.active) } }
</script>
<template>
  <li v-for="user in activeUsers" :key="user.id">{{ user.name }}</li>
</template>

<!-- ✅ 也可以：把整段 v-if 提到外层 -->
<ul v-if="showList">
  <li v-for="user in users" :key="user.id">{{ user.name }}</li>
</ul>
```

### Vue 常用的修饰符有哪些？有什么应用场景？

常见修饰符可以分三类：

1. 事件修饰符：`.stop`、`.prevent`、`.capture`、`.self`、`.once`、`.passive`
   场景是阻止冒泡、阻止默认行为、只触发一次等。
2. 按键修饰符：`.enter`、`.tab`、`.esc` 等
   场景是键盘快捷操作。
3. `v-model` 修饰符：`.lazy`、`.number`、`.trim`
   场景是延迟同步、自动转数字、去除首尾空格。

面试时可以强调：修饰符的作用是把通用 DOM 处理逻辑用声明式方式写在模板中，提高可读性。

### 说说你对 nextTick 的理解？

`nextTick` 的作用是在下次 DOM 更新循环结束后执行延迟回调。

原因在于 Vue 为了性能优化，不会在数据一变化时就立刻更新 DOM，而是把同一事件循环中的多个更新合并，异步批量执行。

所以当你修改数据后立刻访问 DOM，拿到的可能还是旧 DOM，这时就需要 `nextTick`。

常见场景：

1. 更新数据后获取最新 DOM 结构。
2. 更新列表后让输入框自动聚焦。
3. 在切换显示状态后获取元素尺寸。

一句话概括：`nextTick` 是“等待 Vue 完成异步 DOM 更新后再执行逻辑”。

### Vue 中 key 的原理是什么？说说你的理解？

`key` 的本质是给虚拟 DOM 节点一个稳定且唯一的身份标识，便于 diff 过程准确判断节点是否可以复用。

它的作用主要有：

1. 提高 diff 效率。
2. 避免错误复用 DOM 节点。
3. 保证列表重排、插入、删除时状态正确。

如果不加 `key`，Vue 会尽量复用相同位置的节点，这在某些场景会导致输入框内容错乱、组件状态错位。

实际开发中，`key` 要尽量使用稳定唯一的 id，不建议直接用数组下标，尤其是在列表会增删、排序时。

**为什么不能用数组下标？**

下标的本质是"位置编号"，而不是"内容编号"——数组结构一旦变化，同一个下标就会指向不同的数据，导致 diff 把"已经换了内容的旧节点"误当成"内容没变的复用节点"，结果就是 DOM 复用错位、组件内部状态保留到错误的项上。来看一个具体例子：

```html
<!-- 列表初始：A B C，分别带一个输入框 -->
<li v-for="(item, i) in list" :key="i">
  <input />     <!-- 用户在 A 的 input 里输入了 "hello" -->
  {{ item }}
</li>
```

现在在头部插入一个 X，列表变成 `X A B C`：

| 下标 | 旧数据 | 新数据 | diff 判定 | 实际表现 |
| --- | --- | --- | --- | --- |
| 0 | A | X | key 相同（都是 0）→ 复用 | A 的 DOM 被复用渲染 X，**input 里的 "hello" 留在了 X 上** |
| 1 | B | A | key 相同 → 复用 | B 的 DOM 被复用渲染 A |
| 2 | C | B | key 相同 → 复用 | C 的 DOM 被复用渲染 B |
| 3 | - | C | key 不存在 → 新建 | 创建 C 的全新 DOM |

而如果用唯一 id 作 key：

| key | 旧位置 | 新位置 | diff 判定 | 表现 |
| --- | --- | --- | --- | --- |
| id-A | 0 | 1 | 找到匹配 → 移动 | A 的 DOM 整体移到位置 1，"hello" 跟着 A 走 ✓ |
| id-X | - | 0 | 不存在 → 新建 | 全新 X 节点，input 是空的 ✓ |
| id-B / id-C | 类似处理 | | | |

**性能上的差异：**

1. 用下标时，diff 倾向"原地复用 + 修改属性"，看起来快但实际触发了几乎所有节点的属性 patch，且组件内部 state 错位。
2. 用稳定 id 时，diff 通过"位置移动"完成更新，不需要修改任何节点的内容，DOM 操作反而更少。

**什么场景下用下标是安全的？**

只有满足三个条件之一才可以放心用下标：列表只渲染不增删、列表元素是无状态纯展示、列表永远不会被排序或过滤。一旦项内含输入框、组件、动画状态等"位置敏感的内容"，就必须换成稳定唯一的 id。

### 什么是虚拟 DOM？如何实现一个虚拟 DOM？说说你的思路。

虚拟 DOM 本质上是对真实 DOM 的 JavaScript 对象描述。它把真实 DOM 抽象成对象树，数据变化时先生成新的虚拟 DOM，再和旧虚拟 DOM 做比较，最后把差异更新到真实 DOM。

它的价值在于：

1. 让跨平台渲染成为可能。
2. 把直接操作 DOM 转成对象计算，便于框架统一调度。
3. 配合 diff 算法减少不必要的 DOM 操作。

如果手写一个简化版，大致步骤是：

1. 用对象描述标签名、属性、子节点。
2. 写一个 `render` 方法，把虚拟节点递归转成真实 DOM。
3. 数据变化后生成新的虚拟 DOM。
4. 对新旧节点做 diff，对比标签、属性、子节点。
5. 把差异通过 patch 应用到真实 DOM。

### 了解过 Vue 中的 diff 算法吗？说说看。

Vue 的 diff 是同层比较，不会跨层级做复杂比对，这样时间复杂度更可控。

核心思路：

1. 先比较节点类型，如果标签不同，直接替换。
2. 如果标签相同，复用节点，再比较属性差异。
3. 子节点比较时，尽量通过 `key` 识别可复用节点。
4. 列表更新里会采用首尾指针等优化策略，减少移动和创建操作。

面试时要说清楚：Vue 并不是暴力全量更新，而是通过虚拟 DOM + diff + patch 找出最小修改集，再更新真实 DOM。

### Vue 中给对象添加新属性界面不刷新？

这是 Vue2 的典型响应式限制。因为 Vue2 使用 `Object.defineProperty` 劫持已有属性，初始化时不存在的属性后续直接新增，Vue 无法拦截到。

解决方式：

1. 使用 `Vue.set(obj, key, value)` 或 `this.$set`。
2. 重新赋值一个新对象，例如：`this.obj = { ...this.obj, newKey: value }`。

Vue3 使用 `Proxy` 实现响应式，这个问题基本被解决了。

### Vue.observable 你有了解过吗？说说看。

`Vue.observable` 是 Vue2.6 提供的一个 API，可以让一个普通对象变成响应式对象。

常见用途：

1. 在简单场景下做轻量级状态共享。
2. 不引入 Vuex 的情况下，快速实现跨组件共享状态。

示例思路是：定义一个 `state = Vue.observable({ count: 0 })`，多个组件引用这个对象后，状态变化会同步更新。

但它更适合小型场景，复杂状态管理仍然建议使用 Vuex 或 Vue3 中的组合式状态管理方案。

### Vue 中的过滤器了解吗？过滤器的应用场景有哪些？

过滤器本质上是对展示层数据做格式化处理，比如日期、金额、状态文本转换。

优点是模板写法简洁，适合简单文本格式化。

缺点也很明显：

1. 只能用于模板展示，复用范围有限。
2. 逻辑一复杂就不利于维护。
3. Vue3 已移除过滤器，推荐使用方法、计算属性或组合式函数替代。

所以面试中可以表述为：过滤器适合轻量展示格式化，但不是复杂业务逻辑的合适载体。

**为什么过滤器只适合"轻量"？** 它有几个先天约束：

1. **作用域受限**：过滤器只能在模板的 `{{ }}` 或 `v-bind` 里链式调用（`{{ price | currency | round }}`），无法在 JS 逻辑里用，导致同一段格式化逻辑在 script 和 template 之间无法复用。
2. **不可组合参数化**：过滤器接收的总是被过滤的值本身，传额外参数要靠 `| filter(arg)` 这种特殊语法，写起来像 DSL，TS 类型推导也无法跟进。
3. **不参与依赖追踪的优化**：过滤器是每次 render 时执行的纯函数，没有缓存。同一个昂贵格式化跑 100 次列表项就执行 100 次。
4. **维护性差**：全局注册的过滤器分散在 main.js、模块中，调用处只看到一个 `| someFilter`，跳转、查找、重命名都很费劲。
5. **Vue 3 直接移除**：官方明确建议用 method、computed 或 composable 替代，过滤器在 Vue 3 下是兼容旧代码的"历史包袱"。

具体迁移建议：

```vue
<!-- Vue 2 过滤器 -->
{{ time | formatDate('YYYY-MM-DD') }}

<!-- Vue 3 直接调方法（最简单） -->
{{ formatDate(time, 'YYYY-MM-DD') }}

<!-- Vue 3 + computed（带缓存） -->
<script setup>
const formattedTime = computed(() => formatDate(time.value, 'YYYY-MM-DD'))
</script>
{{ formattedTime }}

<!-- Vue 3 + composable（跨组件复用） -->
<script setup>
const { format } = useDateFormat(time, 'YYYY-MM-DD')
</script>
```

### 你有写过自定义指令吗？自定义指令的应用场景有哪些？

自定义指令适合封装底层 DOM 行为复用，核心价值是直接操作元素。

常见场景：

1. 输入框自动聚焦。
2. 权限控制。
3. 元素拖拽。
4. 防抖、节流。
5. 图片懒加载。
6. 点击元素外部区域自动关闭。

它和组件的区别在于：组件是对 UI 结构和逻辑的抽象，指令是对 DOM 行为的抽象。

### 说说你对 Vue mixin 的理解，有什么应用场景？

`mixin` 是一种逻辑复用机制，可以把多个组件共用的选项配置提取出来，再混入到组件内部。

它能混入的内容包括：`data`、`methods`、生命周期、计算属性等。

优点是能复用逻辑，缺点也很明显：

1. 命名冲突风险高。
2. 数据来源不直观，可读性差。
3. 多个 mixin 叠加后维护成本高。

典型场景是早期项目里复用表格分页、弹窗控制、列表查询逻辑。

如果面试官继续追问，可以补充：Vue3 更推荐使用 Composition API 替代 mixin，因为逻辑来源更清晰。

**为什么 Composition API 更好？** 对照三个 mixin 经典痛点看：

1. **命名冲突 → 显式 import**：mixin 把所有字段注入到组件，两个 mixin 同时定义 `loading` 就静默覆盖，且 IDE 不会报错；Composition API 通过 `const { loading } = useFetch()` 显式取出，重名直接报错或要改名 `const { loading: postLoading } = useFetch()`。
2. **数据来源不直观 → 来源即定义**：mixin 写了 `this.fetchList()`，模板里看不出来这个方法是哪来的；Composition API 一眼能看见 `useList()` 的 import 语句，跳转到定义、追溯依赖都很直接。
3. **多 mixin 叠加难维护 → 函数组合自然解决**：mixin 之间没有显式依赖关系，`mixinA` 默默用了 `mixinB` 的方法，删 `mixinB` 时编辑器不会提示；composables 是普通函数，`useA()` 内部可以 `const b = useB()`，依赖图清晰且 TS 能完整推导。

具体对比：

```js
// ❌ Vue 2 mixin
const fetchMixin = {
  data: () => ({ loading: false, list: [] }),
  methods: { async fetch() { /* ... */ } },
}
// 组件里用
export default { mixins: [fetchMixin] }  // 看不出来注入了什么

// ✅ Vue 3 composable
function useFetch() {
  const loading = ref(false)
  const list = ref([])
  async function fetch() { /* ... */ }
  return { loading, list, fetch }
}
// 组件里用
const { loading, list, fetch } = useFetch()  // 一目了然
```

更重要的是**类型安全**：composable 的返回类型 TS 能完整推导，组件里使用时有自动补全；mixin 的注入字段在 TS 里几乎拿不到类型，要么 `any` 要么写大量 declaration merging。这是 Vue 团队彻底放弃 mixin 推 Composition API 的根本动力。

## 四、组件化与复用

### 说说你对 slot 的理解？slot 使用场景有哪些？

`slot` 就是插槽，本质上是组件内容分发机制，让父组件把一部分模板内容传给子组件指定位置渲染。

常见类型：

1. 默认插槽。
2. 具名插槽。
3. 作用域插槽。

应用场景：

1. 封装通用弹窗、表格、卡片、表单容器。
2. 组件只负责结构和行为，具体内容由调用方决定。
3. 作用域插槽用于把子组件内部数据暴露给父组件自定义渲染。

一句话总结：插槽提升了组件的可扩展性和复用能力。

### `$attrs` 和 `$listeners` 是什么？有什么作用？

这两个能力主要用于组件封装和多层透传场景。

1. `$attrs` 表示父组件传入、但当前组件没有通过 `props` 显式声明的属性集合。
2. `$listeners` 表示父组件传入的事件监听器集合。

它们的常见作用：

1. 在高阶组件或中间包装组件中，把属性和事件继续透传给更底层组件。
2. 减少中间层组件重复声明大量 `props` 和事件。

如果是 Vue3，可以补充：事件和属性透传方式有所调整，但核心思想仍然是“把外部能力继续往下传”。

### provide / inject 的原理和使用场景是什么？

`provide / inject` 是 Vue 提供的祖先组件向后代组件跨层级传值的机制。

常见场景：

1. 组件库中的表单、主题、配置透传。
2. 祖先组件向深层子组件共享上下文信息。
3. 减少层层 `props` 透传。

它的特点：

1. 更适合跨层级但范围有限的共享。
2. 不适合替代完整状态管理。
3. 数据来源相对隐式，使用时要注意可维护性。

面试时可以强调：简单跨层级共享用它合适，但全局复杂状态还是更适合 Vuex 或 Pinia。

**为什么 provide/inject 不该承担"全局状态管理"角色？** 三个核心局限：

1. **可追溯性差**：provide 是"谁注入的"在祖先组件里；inject 是"谁用了"散落在后代组件里。一个数据被改后，定位"是谁改的"非常困难，没有 DevTools 的 mutation 时间线、没有 action 历史。状态管理库的核心价值之一就是把这条链路显式化。
2. **调试工具弱**：Pinia/Vuex 都有专门的 DevTools 面板，可以看时间旅行、热更新、状态树快照；inject 的值在 Vue DevTools 里只能看到当前快照，没有变更历史。
3. **缺乏纪律约束**：provide 的值可以是 ref、对象、方法甚至 store 实例，没有"修改入口必须经过特定方法"的约束——任何后代拿到响应式引用都能直接改。状态管理库会通过 action/mutation 把"谁能改、什么时候改"显式化，避免大型项目里的"状态被四处修改"。
4. **无法做插件化扩展**：状态管理库有持久化、订阅、热更新、SSR 同构等成熟生态，provide/inject 全要自己造轮子。
5. **组件树外拿不到**：路由守卫、HTTP 拦截器、工具函数等"非组件"环境无法 inject——必须先有组件实例上下文。Pinia store 可以在任何地方 `useUserStore()` 直接拿到。

**所以选型规则是：**

1. **范围小、内容稳定、面向组件树内部**：provide/inject。比如表单组件给内部子组件传 `formItem` 上下文、主题色、i18n 实例。
2. **范围大、跨页面、跨组件树、需要在 JS 模块里访问**：Pinia/Vuex。比如用户登录态、购物车、全局通知队列。

一个常被忽略的中间地带是"组件库内部用 provide/inject 是最佳实践"——比如 Element Plus 的 `<el-form>` 给 `<el-form-item>` 传 model 和 rules，就是 provide/inject 的经典用法，因为它们语义上就是父子上下文，不需要全局状态库。

### 怎么缓存当前的组件？缓存后怎么更新？说说你对 keep-alive 的理解是什么？

`keep-alive` 是 Vue 内置组件，用来缓存动态组件或路由组件，避免频繁销毁和重建。

适用场景：

1. 列表页切详情页后返回时保留滚动位置和筛选条件。
2. 多标签页切换保留表单输入状态。

它的常见能力：

1. `include` / `exclude` 控制哪些组件被缓存。
2. `max` 控制最大缓存数量。
3. 组件会触发 `activated` 和 `deactivated` 生命周期。

缓存后如果想更新：

1. 可以通过改变 `key` 强制重新渲染。
2. 在 `activated` 中重新拉取数据。
3. 配合路由元信息控制是否缓存。

### Vue 组件间通信方式都有哪些？

Vue 中常见通信方式有：

1. 父传子：`props`。
2. 子传父：`$emit`。
3. 父访问子：`ref`。
4. 祖先传后代：`provide / inject`。
5. 跨组件共享状态：Vuex、Pinia。
6. 事件总线：适合简单场景，但大型项目不推荐滥用。

面试时不要只列名词，要补一句选型原则：

1. 简单层级通信优先 `props` + `emit`。
2. 跨层级但范围小用 `provide/inject`。
3. 全局共享状态用状态管理工具。

### Vue 中组件和插件有什么区别？

组件和插件的定位不同：

1. 组件是页面 UI 和业务功能的组成单元，用来构建界面。
2. 插件是对 Vue 全局能力的扩展，通常通过 `install` 方法注入全局方法、指令、混入、全局组件等。

比如：

1. 按钮、弹窗、表格属于组件。
2. 路由、状态管理、UI 库注册、全局埋点能力属于插件。

一句话区分：组件是“拿来渲染”的，插件是“拿来扩展框架能力”的。

## 五、工程化与项目实践

### Vue 项目中有封装过 axios 吗？怎么封装的？

实际项目里一般会封装 axios，而不是到处直接调用，主要目的是统一请求行为和便于维护。

常见封装思路：

1. 创建 axios 实例，统一 `baseURL`、超时时间、请求头。
2. 请求拦截器里统一注入 token、traceId、公共参数。
3. 响应拦截器里统一处理业务码、异常提示、登录失效、权限不足。
4. 再封装 `get/post/upload/download` 等方法。
5. 按业务模块拆分 API 文件，例如 `user.ts`、`order.ts`。
6. 对重复请求取消、幂等、防抖等高级能力做统一治理。

面试时可以强调：封装 axios 的目标不是“包一层”，而是形成统一网络层规范。

### 说说你对 Vuex 的理解？它的核心原理是什么？

Vuex 是 Vue2 时代常见的集中式状态管理方案，用来统一管理跨组件共享状态。

核心概念通常包括：

1. `state`
2. `getters`
3. `mutations`
4. `actions`
5. `modules`

它的工作流通常是：

1. 组件读取 `state` 或 `getters`。
2. 组件通过 `commit` 提交 `mutation` 修改状态。
3. 异步逻辑一般放在 `action` 中，再去 `commit`。

核心原理可以概括为：

1. 借助 Vue 的响应式系统让 `state` 具备可追踪更新能力。
2. 通过统一入口修改状态，保证状态流更可预测。

如果面试官继续追问，可以补一句：Vue3 项目里很多场景已经更常使用 Pinia，但理解 Vuex 的状态流设计仍然很有价值。

### Vuex 的五大核心概念分别承担什么职责？

每个概念都有明确分工，理解它们的边界比记 API 更重要。

1. **`state`**：单一数据源。整个应用的所有共享状态都放在这棵对象树里，便于序列化、调试。
2. **`getters`**：派生状态。类似组件里的 `computed`，可以基于 state 计算出新值，并自带缓存——只要依赖的 state 没变，多次读取不会重复计算。
3. **`mutations`**：**唯一允许同步修改 state 的入口**。每个 mutation 是一个 `(state, payload) => void` 的函数，必须同步执行。这是 Vuex 时间旅行调试能成立的根本约束。
4. **`actions`**：处理异步逻辑或复杂业务流。action 不直接改 state，只能通过 `context.commit('xxx')` 触发 mutation。
5. **`modules`**：把大型 store 按业务领域拆分成嵌套子模块，每个模块有自己的 state、getters、mutations、actions。

四类调用方式：

```js
this.$store.state.user.name              // 读 state
this.$store.getters.fullName             // 读 getter
this.$store.commit('user/setName', 'A')  // 触发 mutation
this.$store.dispatch('user/login', form) // 派发 action
```

### Vuex 中 mutation 和 action 有什么区别？为什么不直接在 action 里改 state？

两者最大的差异是**同步与异步**：

| 维度 | mutation | action |
| --- | --- | --- |
| 是否能改 state | 是（且是唯一合法入口） | 否，只能 `commit` mutation |
| 是否同步 | 必须同步 | 可以异步 |
| 调用方式 | `commit(type, payload)` | `dispatch(type, payload)` |
| 调试器是否记录 | 是（每次 mutation 都有快照） | 否（只是流程协调） |

为什么强制 mutation 同步？因为 Vuex DevTools 需要在 mutation 前后各打一个快照实现"时间旅行"，如果 mutation 内含异步，回放就无法准确还原状态。所以 Vuex 的纪律是：**异步流程在 action 里组织，最终落到状态变更时再 commit mutation**。

### Vuex 模块化和命名空间是怎么用的？

大型项目状态会膨胀，模块化是避免巨型 store 的标准做法。

```js
const user = {
  namespaced: true,
  state: () => ({ name: '' }),
  mutations: { setName(state, v) { state.name = v } },
  actions: { async login({ commit }, form) { /* ... */ } },
}

const store = new Vuex.Store({
  modules: { user }
})
```

要点：

1. **`namespaced: true`** 让模块内部的 mutations/actions/getters 自带命名空间前缀，调用时要写 `commit('user/setName')`，避免不同模块间命名冲突。
2. **state 用函数写**：和组件的 data 同理，多实例场景下保证每个 store 实例拿到独立对象。
3. **跨模块通信**：在 action 里可以 `dispatch('other/xxx', null, { root: true })` 触发别的模块。
4. **嵌套模块**：模块里再嵌模块也支持，但层级别太深，否则路径难维护。

### 在组件里如何方便地访问 Vuex？

直接写 `this.$store.state.x.y.z` 太啰嗦，Vuex 提供了四个辅助函数把 store 内容映射到组件：

```js
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'

export default {
  computed: {
    ...mapState('user', ['name', 'avatar']),
    ...mapGetters('user', ['isAdmin']),
  },
  methods: {
    ...mapMutations('user', ['setName']),
    ...mapActions('user', ['login', 'logout']),
  },
}
```

这套写法把 store 内容当成组件里的 computed 和 method 用，模板里直接写 `name` / `setName(...)` 即可。Composition API 时代它略显啰嗦，所以 Pinia 取消了 mutations 概念，直接 `const store = useStore()` 就能用。

### Vuex 状态如何持久化？

Vuex 的 state 默认存内存，刷新就丢。常见持久化方案有三种：

1. **手动 + localStorage**：在 mutation 里同步写 localStorage，初始化时从中恢复——侵入性最强，灵活度最高。
2. **`vuex-persistedstate` 插件**：基于 Vuex 的 `subscribe` 机制，每次 mutation 后自动写入 storage，支持配置 paths（只持久化部分模块）、自定义 storage（localStorage/sessionStorage/cookie）。
3. **基于路由或登录态决定持久化范围**：登录信息持久化到 localStorage，会话信息只放 sessionStorage，敏感字段不存。

需要注意的边界：持久化方案选错粒度会引发"刷新后状态错乱"——比如把 token 存了但用户信息没存，初始化时拿到 token 却不知道自己是谁。最佳实践是状态分层：纯 UI 状态不持久化，关键标识持久化，可重新拉取的数据通过路由钩子重新请求。

### 说说你对 Pinia 的理解？为什么 Vue3 项目优先选 Pinia？

Pinia 是 Vue 团队官方推荐的下一代状态管理方案（作者 Eduardo San Martin Morote 同时也是 Vue Router 和 Vuex 的核心维护者）。它**已经成为 Vue 官方推荐**，Vuex 进入维护模式。

**核心设计理念：**

1. **去 mutation 化**：直接在 actions 里改 state，不再区分 mutation/action，心智负担减半。
2. **天然 TypeScript 友好**：完全用 TS 重写，类型推导无需任何额外声明。
3. **没有嵌套模块**：每个 store 是平级的，按业务领域拆分多个 store 即可。
4. **Composition API 化**：API 表面就是 `defineStore` + 一个工厂函数，写法和 Vue3 完全一致。
5. **更小**：核心包约 1.5KB（Vuex 4 约 9KB）。
6. **DevTools 一等公民**：支持时间旅行（虽然没有强制 mutation，但内部仍能追踪每次变更）、热更新、自动补全。

**Options 风格的 Pinia store：**

```ts
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0, history: [] as number[] }),
  getters: {
    double: (state) => state.count * 2,
    // 访问别的 getter
    quadruple(): number { return this.double * 2 },
  },
  actions: {
    increment() {
      this.count++
      this.history.push(this.count)
    },
    async fetchInitial() {
      const { value } = await api.getCount()
      this.count = value
    },
  },
})
```

组件里用：

```vue
<script setup>
import { storeToRefs } from 'pinia'
import { useCounterStore } from '@/stores/counter'

const store = useCounterStore()
const { count, double } = storeToRefs(store)  // 保持响应式解构
const { increment } = store                    // action 可以直接解构
</script>

<template>
  <button @click="increment">{{ count }} / {{ double }}</button>
</template>
```

### Pinia 的两种 store 写法（Options vs Setup）有什么区别？

Pinia 支持两种语法风格，对应 Vue 的两种 API。

**Options 风格（上面的例子）**：用 `state/getters/actions` 三块组织，写起来像 Vuex，但少了 mutation。

**Setup 风格**：完全是 Composition API 的写法，灵活度更高：

```ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)                           // 相当于 state
  const double = computed(() => count.value * 2) // 相当于 getter
  function increment() {                          // 相当于 action
    count.value++
  }
  return { count, double, increment }
})
```

**两种写法对比：**

| 维度 | Options 风格 | Setup 风格 |
| --- | --- | --- |
| 心智模型 | 像 Vuex/Vue Options API | 像 Vue Composition API |
| 灵活度 | 受限于三大块结构 | 任意——可以用 `watch`、`provide`、内部辅助函数 |
| TS 友好 | 好 | 极好 |
| 私有状态 | 必须放进 state | 可以用未 return 的局部变量做"私有" |
| 推荐 | 简单 store、迁移自 Vuex | 复杂 store、需要响应式副作用 |

新项目优先选 Setup 风格，和组件代码风格一致，长期更易维护。

### Pinia 在工程实践中怎么组织？常见最佳实践？

1. **按业务域拆 store**：`useUserStore`、`useCartStore`、`useProductStore` 平级存在，不要塞进单一 store。
2. **跨 store 调用**：直接 `import { useOtherStore } from ...; const other = useOtherStore()`，平级 import 互相协作。
3. **响应式解构必须用 `storeToRefs`**：直接解构 `const { count } = store` 会丢响应式（因为 store 是 reactive proxy，解构出的是普通值）。
4. **持久化用 `pinia-plugin-persistedstate`**：在 store 里加 `persist: true` 即可自动持久化，可配置 storage、paths、序列化方式。
5. **订阅变化用 `$subscribe` 和 `$onAction`**：分别监听 state 变化和 action 调用，常用于埋点、日志、调试。
6. **重置 store**：调用 `store.$reset()` 一键回到初始状态（Setup 风格不支持自动 reset，要自己实现）。
7. **配合 SSR**：每次请求新建一个 pinia 实例传给应用，避免请求间状态污染。

```ts
// 跨 store 协作
export const useCartStore = defineStore('cart', () => {
  const userStore = useUserStore()  // 直接拿到另一个 store
  const items = ref<Item[]>([])
  const canCheckout = computed(() => userStore.isLoggedIn && items.value.length > 0)
  return { items, canCheckout }
})
```

### Vuex 和 Pinia 全方位对比？怎么从 Vuex 迁移到 Pinia？

**1. API 设计**

| 维度 | Vuex | Pinia |
| --- | --- | --- |
| 核心概念 | state / getters / mutations / actions / modules | state / getters / actions（无 mutations、无 modules） |
| 修改 state 入口 | 必须 commit mutation | 直接在 action 里改 / 组件里 `store.x = y` |
| 模块化 | 嵌套 modules + namespaced | 多个 store 平级存在，按业务拆 |
| 异步处理 | action 内 `commit` mutation | action 内直接改 state |
| 调用方式 | `commit('user/setName')` 字符串路径 | `userStore.setName()` 直接方法调用 |

**2. TypeScript 体验**

| 维度 | Vuex | Pinia |
| --- | --- | --- |
| 类型推导 | 几乎没有，要写 `ModuleTree<RootState>` 等大量样板 | 完全自动推导，零样板 |
| store 自动补全 | 字符串路径无补全 | 完整方法、属性补全 |
| getter 类型 | 需要手动声明 | 根据返回值自动推导 |

**3. 工程化维度**

| 维度 | Vuex | Pinia |
| --- | --- | --- |
| 包大小（min+gzip） | Vuex 4 约 9KB | 约 1.5KB |
| Vue 版本 | Vue 2 / Vue 3 | Vue 2（pinia ≤2.x）/ Vue 3 |
| DevTools | 支持 | 支持，且支持热更新 |
| SSR | 支持但要小心单例 | 推荐每次请求 new 一个 pinia |
| 学习曲线 | 中（要懂 mutation 约束） | 低（几乎零额外概念） |
| Vue 官方推荐 | 维护模式 | **当前推荐** |

**4. 心智差异**

1. Vuex 强调"显式状态变更"——所有变化必须经过 commit，便于追溯；代价是繁琐。
2. Pinia 信任开发者直接修改——配合 DevTools 仍能看到变更历史，写法更轻。

**5. 从 Vuex 迁移到 Pinia 的步骤：**

1. 安装 pinia，在入口 `app.use(createPinia())`。
2. 把每个 Vuex 模块拆成独立的 `defineStore`。
3. mutations 直接合并到 actions（或删除，组件里直接改）。
4. namespaced 路径调用 → 改成 `useXxxStore().xxx()`。
5. 渐进式迁移：Pinia 和 Vuex 可以共存，新功能用 Pinia，旧模块逐步替换。
6. 持久化插件、辅助函数等周边配套也要替换。

**一句话选型：**

1. 老的 Vue 2 项目，沉淀深、改造成本高 → 继续用 Vuex 没问题。
2. 新项目，无论 Vue 2 还是 Vue 3 → **直接上 Pinia**。
3. 老 Vue 3 项目正用 Vuex 4 → 找合适窗口期迁移到 Pinia。

### 你了解 Axios 的原理吗？有看过它的源码吗？

Axios 本质上是对浏览器 `XMLHttpRequest` 和 Node.js `http` 模块的封装，并在外层做了 Promise 化、拦截器、请求配置合并、响应转换等处理。

它的核心点包括：

1. 创建实例时会合并默认配置和用户配置。
2. 请求发起前会按顺序执行请求拦截器。
3. 请求发送交给适配器 `adapter`，浏览器端通常是 XHR。
4. 响应回来后执行响应拦截器。
5. 最终返回 Promise。

所以 axios 的价值不只是发请求，而是提供一套可扩展、可拦截、可统一治理的请求链路。

### 跨域是什么？Vue 项目中你是如何解决跨域的呢？

跨域本质上是浏览器同源策略限制，当前页面去请求协议、域名、端口任意一个不同的接口时，就属于跨域请求。

常见解决方案：

1. 开发环境通过代理转发，比如 Vite 或 Webpack devServer proxy。
2. 后端配置 CORS 响应头，这是最常规也最规范的方式。
3. 同域部署，通过 Nginx 反向代理统一入口。
4. JSONP 只适用于 GET，现在基本很少用了。

在 Vue 项目里最常见的回答是：开发阶段用代理，生产阶段通过网关或 Nginx 转发，并由后端正确配置 CORS。

### 说下你的 Vue 项目的目录结构，如果是大型项目你该怎么划分结构和划分组件呢？

大型 Vue 项目目录设计要满足可维护、可扩展、低耦合。

常见划分思路：

1. `src/api`：接口层。
2. `src/utils`：工具方法。
3. `src/router`：路由配置。
4. `src/store`：状态管理。
5. `src/views`：页面级组件。
6. `src/components`：通用组件。
7. `src/layout`：整体布局。
8. `src/hooks` 或 `composables`：可复用逻辑。
9. `src/directives`：自定义指令。
10. `src/styles`：全局样式与主题变量。

组件划分原则：

1. 页面组件负责组装。
2. 业务组件承载具体业务功能。
3. 基础组件保持通用和稳定。
4. 逻辑复用尽量抽成 hooks，而不是堆到组件里。

如果项目更大，可以按业务域拆模块，而不是只按技术类型拆目录。

### Vue 要做权限管理该怎么做？控制到按钮级别的权限怎么做？

权限管理一般分三层：

1. 路由级权限。
2. 页面内容级权限。
3. 按钮级权限。

实现思路通常是：

1. 登录后获取用户角色、权限标识。
2. 前端根据权限动态生成可访问路由。
3. 页面内根据权限码控制按钮、菜单、操作区显示隐藏。
4. 后端必须做最终权限校验，前端权限控制只能提升体验，不能替代安全校验。

按钮级权限常见实现方式：

1. 自定义指令，如 `v-permission`。
2. 封装权限组件。
3. 在模板中配合权限集合做条件渲染。

面试时一定要强调：权限控制必须前后端配合，前端不是安全边界。

### 你是怎么处理 Vue 项目中的错误的？

错误处理通常分为运行时错误、请求错误、资源加载错误和全局兜底。

常见方案：

1. 使用 `app.config.errorHandler` 或 Vue2 的全局错误处理捕获组件异常。
2. 对 Promise、接口请求统一做异常拦截。
3. 监听 `window.onerror` 和 `unhandledrejection`。
4. 路由切换异常做单独处理。
5. 错误上报到日志平台，比如 Sentry。
6. 对用户展示友好的兜底提示页面。

成熟项目中，错误处理的核心不是打印日志，而是“发现问题、定位问题、恢复问题”。

### SPA（单页应用）首屏加载速度慢怎么解决？

首屏慢本质上是首次要加载的资源太多、执行太重。

常见优化手段：

1. 路由懒加载、组件按需加载。
2. 合理拆包，避免主包过大。
3. 静态资源压缩，如 gzip、brotli。
4. 图片压缩、WebP、懒加载。
5. CDN 加速静态资源。
6. 开启浏览器缓存和协商缓存。
7. 减少首屏无关代码执行。
8. 服务端渲染 SSR 或预渲染。
9. 使用骨架屏提升感知速度。
10. 对第三方库按需引入，避免全量打包。

回答这题时，最好从“网络传输、资源体积、代码执行、渲染策略”四个维度组织语言。

### SSR 解决了什么问题？有做过 SSR 吗？你是怎么做的？

SSR 即服务端渲染，指的是页面 HTML 在服务端先生成，再返回给浏览器。

它主要解决两个问题：

1. 首屏渲染速度更快，尤其对弱网环境更友好。
2. SEO 更好，搜索引擎更容易抓取完整内容。

它的代价也要说明：

1. 服务端压力更大。
2. 开发复杂度更高。
3. 需要处理客户端和服务端运行环境差异。

如果回答“怎么做”，可以说：

1. 使用 Nuxt 或 Vue 官方 SSR 方案。
2. 路由级预取数据。
3. 服务端渲染首屏 HTML。
4. 客户端接管并完成 hydration。
5. 配合缓存提升服务端性能。

### Vue 项目如何部署？有遇到部署服务器后刷新 404 问题吗？

Vue 项目部署本质上是把构建后的静态资源部署到 Nginx、CDN、对象存储或 Node 服务中。

如果使用 history 路由模式，刷新 404 的原因是：浏览器会向服务器请求当前路径对应资源，但服务器并没有这个真实文件。

解决方式：

1. Nginx 配置 `try_files $uri $uri/ /index.html;`
2. Node 服务统一把非静态资源请求回退到 `index.html`
3. 或者改用 hash 模式路由，但 URL 不够美观。

所以这题的关键点不是“怎么部署”，而是要说明 history 模式的服务端回退配置。

## 六、Vue3 相关

### Vue3 有了解过吗？能说说跟 Vue2 的区别吗？

Vue3 和 Vue2 的差异主要体现在以下几方面：

1. 响应式实现不同：Vue2 用 `Object.defineProperty`，Vue3 用 `Proxy`。
2. 代码组织方式不同：Vue3 增加 Composition API，逻辑复用更灵活。
3. 性能更好：编译优化、patch 优化、Tree Shaking 更友好。
4. 生命周期命名有变化：如 `beforeDestroy` 变成 `beforeUnmount`。
5. 更好的 TypeScript 支持。
6. 新特性更多：Fragment、Teleport、Suspense 等。
7. 过滤器移除，`v-model` 和响应式 API 设计都有升级。

面试时建议表达为：Vue3 不是简单升级，而是在响应式底层、编译优化和代码组织方式上都做了较大重构。

### Vue 2 和 Vue 3 在底层和 API 上具体有哪些差异？

可以从 7 个维度系统对比，这也是中高级面试重点。

**1. 响应式系统**

| 维度 | Vue 2 | Vue 3 |
| --- | --- | --- |
| 拦截机制 | `Object.defineProperty` | `Proxy` + `Reflect` |
| 新增属性 | 不响应（要 `Vue.set`） | 自动响应 |
| 删除属性 | 不响应（要 `Vue.delete`） | 自动响应 |
| 数组下标赋值 | 不响应 | 自动响应 |
| 数组 length 修改 | 不响应 | 自动响应 |
| Map / Set 支持 | 不响应 | 自动响应 |
| 初始化成本 | 递归遍历所有属性 | 懒代理（用到才代理子属性） |

**2. 全局 API**

```js
// Vue 2 —— 全局 API 直接挂在 Vue 上
Vue.component(...)
Vue.directive(...)
Vue.use(...)
Vue.prototype.$http = axios
new Vue({ render: h => h(App) }).$mount('#app')

// Vue 3 —— 改成应用实例 API，避免全局污染
const app = createApp(App)
app.component(...)
app.directive(...)
app.use(...)
app.config.globalProperties.$http = axios
app.mount('#app')
```

意义：Vue 3 一个页面可以同时跑多个独立的 app 实例，配置互不干扰，对微前端、组件库 demo、多入口场景更友好。

**3. 模板与组件**

1. **多根节点（Fragment）**：Vue 3 模板可以有多个根元素，无需为了一个根节点强行包 div。
2. **`v-model` 升级**：Vue 2 是 `value + input`，Vue 3 是 `modelValue + update:modelValue`，且支持多个 `v-model:title="..."` 绑定到同一组件。
3. **`v-if` / `v-for` 优先级反转**：Vue 2 是 `v-for` 高，Vue 3 是 `v-if` 高（虽然官方仍不推荐两者写在一起）。
4. **过滤器移除**：用 method、computed 或 composables 替代。
5. **`$listeners` 合并到 `$attrs`**：事件和属性透传统一为 `$attrs`，且支持 `inheritAttrs: false` 配合 `v-bind="$attrs"` 精细控制。

**4. 组件 API**

| 能力 | Vue 2 | Vue 3 |
| --- | --- | --- |
| 主要 API 风格 | Options API | Options + Composition |
| 逻辑复用 | mixin / HOC | composables（自定义 hook） |
| TS 支持 | 弱（要 vue-class-component / vue-property-decorator） | 一等公民 |
| 单文件入口 | `<script>` | `<script setup>` |
| Tree Shaking | 弱 | 强（API 大量改成函数式） |

**5. 内置新组件**

1. **`<Teleport>`**：把内容渲染到 DOM 树任意节点，常用于 Modal / Drawer / Tooltip。
2. **`<Suspense>`**：处理异步组件 loading 和 fallback（实验性，但可用）。
3. **`<Transition>` / `<TransitionGroup>`**：更名（去掉 `vue-` 前缀），API 略有调整。

**6. 生命周期对照**

| Vue 2 | Vue 3 Options | Vue 3 Composition |
| --- | --- | --- |
| beforeCreate | beforeCreate | （setup 取代） |
| created | created | （setup 取代） |
| beforeMount | beforeMount | onBeforeMount |
| mounted | mounted | onMounted |
| beforeUpdate | beforeUpdate | onBeforeUpdate |
| updated | updated | onUpdated |
| beforeDestroy | **beforeUnmount**（重命名） | onBeforeUnmount |
| destroyed | **unmounted**（重命名） | onUnmounted |
| activated | activated | onActivated |
| deactivated | deactivated | onDeactivated |
| errorCaptured | errorCaptured | onErrorCaptured |
| - | renderTracked / renderTriggered | onRenderTracked / onRenderTriggered |

**7. 编译优化**

Vue 3 的编译器在编译期就做了很多 Vue 2 做不到的事：

1. **静态提升（Static Hoisting）**：把没有动态绑定的节点提升到 render 函数外，每次 render 复用同一个 vnode。
2. **PatchFlag**：编译时给每个动态节点打上"会变什么"的标记（class/style/text/props/...），运行时 diff 只对比标记部分。
3. **Block Tree**：把动态节点拍平成数组，避免遍历整棵 vnode 树。
4. **缓存事件处理器**：内联 `@click="..."` 在 Vue 2 每次 render 都创建新函数，Vue 3 编译成 `cache[0] || (cache[0] = ...)`。
5. **SSR 优化**：纯静态片段直接编译成字符串拼接，跳过 vnode。

这些优化叠加，使得 Vue 3 同等模板的运行时性能比 Vue 2 提升约 1.3~2 倍，内存占用减少约 50%。

### Vue3 Composition API 怎么用？

Composition API 的核心入口是 `setup` 函数（或 `<script setup>` 语法糖）。它在组件创建之初执行一次，返回的内容暴露给模板和实例。

**完整示例：**

```vue
<script setup lang="ts">
import { ref, reactive, computed, watch, watchEffect, onMounted } from 'vue'

// 响应式状态
const count = ref(0)
const user = reactive({ name: 'Ada', age: 30 })

// 派生状态
const double = computed(() => count.value * 2)

// 副作用 —— watch 显式声明依赖
watch(count, (newVal, oldVal) => {
  console.log(`count: ${oldVal} → ${newVal}`)
})

// 副作用 —— watchEffect 自动追踪依赖
watchEffect(() => {
  console.log('current name:', user.name)
})

// 生命周期
onMounted(() => {
  console.log('mounted, count =', count.value)
})

// 事件处理
function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">{{ count }} / {{ double }}</button>
  <p>{{ user.name }} - {{ user.age }}</p>
</template>
```

要点：

1. **`<script setup>`** 是 Vue 3.2 引入的编译时语法糖，省掉了显式 `setup() { return {...} }` 的样板代码，所有顶层变量自动暴露给模板。
2. **`ref`** 包装基本类型 / 对象，访问值要 `.value`（模板里自动解包）。
3. **`reactive`** 只能用于对象/数组，返回 Proxy 代理，访问无需 `.value`。
4. **`computed`** 是惰性 + 缓存的派生值，Vue 3 里完全是函数式 API。
5. **`watch`** 显式指定要监听的源，回调能拿到新旧值；**`watchEffect`** 自动收集回调里读过的响应式数据作为依赖。

### ref 和 reactive 有什么区别？什么时候用哪个？

两者都是声明响应式状态的入口，但内部机制和使用习惯不同。

**核心区别：**

| 维度 | `ref` | `reactive` |
| --- | --- | --- |
| 适用类型 | 任意（基本类型、对象、数组） | 仅对象/数组/Map/Set |
| 访问方式 | JS 里要 `.value`，模板自动解包 | 直接像普通对象用 |
| 响应根源 | 内部基于 `RefImpl` 类，对 `.value` 的 get/set 做拦截 | 基于 `Proxy` 代理整个对象 |
| 解构是否保持响应 | 解构出来的还是 ref，保持响应 | 解构后丢响应（要 `toRefs`） |
| 整体替换 | 直接 `xxx.value = newObj`，引用保持响应 | 不能整体替换（会丢响应），只能逐字段改 |

**两者的选用经验：**

1. 基本类型只能用 `ref`。
2. 对象类型两者都行，但**推荐统一用 `ref`**——好处是"所有状态访问方式一致"，看到 `.value` 就知道是响应式状态。
3. 如果觉得 `.value` 烦，可以考虑用 `reactive` 包大对象，但要注意不能整体替换、解构会丢响应。
4. 框架代码、组合式函数返回值优先用 `ref`，符合 Vue 官方风格。

**常见陷阱**：把 ref 直接放进 reactive 时，外层 reactive 会自动解包内层 ref：

```js
const count = ref(0)
const state = reactive({ count })
state.count        // 0，不是 ref
state.count = 5    // 等价于 count.value = 5
```

这是 Vue 的"自动解包"特性，模板里也是同理。

### `<script setup>` 相比普通 `setup()` 函数有何优势？

`<script setup>` 是 Vue 3.2 起稳定的编译时语法糖，它把组件写法压缩到极致：

```vue
<!-- 普通 setup -->
<script>
import { ref, computed } from 'vue'
export default {
  props: { msg: String },
  emits: ['update'],
  setup(props, { emit }) {
    const count = ref(0)
    const double = computed(() => count.value * 2)
    function inc() { count.value++; emit('update', count.value) }
    return { count, double, inc }
  }
}
</script>

<!-- script setup -->
<script setup>
import { ref, computed } from 'vue'
const props = defineProps({ msg: String })
const emit = defineEmits(['update'])
const count = ref(0)
const double = computed(() => count.value * 2)
function inc() { count.value++; emit('update', count.value) }
</script>
```

**优势：**

1. **零样板**：顶层声明的变量自动暴露给模板，不用 return。
2. **更好的类型推导**：组件 props/emits/expose 直接走 TS 类型。
3. **运行时性能更好**：编译器把模板和 script 当作一个作用域处理，省掉 setup 函数调用开销。
4. **import 即用**：导入的组件直接用，不用注册到 components 选项。

**配套编译宏（无需 import，编译期处理）：**

| 宏 | 作用 |
| --- | --- |
| `defineProps` | 声明 props，可基于 TS 类型自动推导 |
| `defineEmits` | 声明 emits |
| `defineExpose` | 显式暴露给父组件 ref 的内容（`<script setup>` 默认全部不暴露） |
| `defineSlots` | 声明插槽类型（3.3+） |
| `defineModel` | 双向绑定声明（3.4 稳定） |
| `defineOptions` | 声明 name、inheritAttrs 等 options |
| `withDefaults` | 给 props 设默认值（3.5 之前类型声明专用） |

**TS + props 的现代写法：**

```ts
// 3.5+ 直接给默认值，不需要 withDefaults
const props = defineProps<{
  title?: string
  items?: string[]
}>()
// 解构默认值（3.5 起编译器保持响应式）
const { title = 'Hello', items = () => [] } = defineProps<...>()
```

### Vue 3 中 v-model 有哪些变化？

**1. 默认绑定名变了：**

```vue
<!-- Vue 2：value + input -->
<MyInput v-model="text" />
<!-- 等价于 -->
<MyInput :value="text" @input="text = $event" />

<!-- Vue 3：modelValue + update:modelValue -->
<MyInput v-model="text" />
<!-- 等价于 -->
<MyInput :modelValue="text" @update:modelValue="text = $event" />
```

**2. 支持多个 v-model：**

一个组件可以绑定多个值，每个 v-model 后跟一个参数名：

```vue
<UserForm v-model:name="name" v-model:age="age" />
```

子组件分别声明：

```vue
<script setup>
defineProps(['name', 'age'])
defineEmits(['update:name', 'update:age'])
</script>
```

**3. 支持自定义修饰符：**

```vue
<MyInput v-model.capitalize="text" />
```

子组件能拿到修饰符配置：

```js
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) },
})
```

**4. Vue 3.4+ 推荐用 `defineModel`：**

```vue
<script setup>
const model = defineModel<string>()  // 自动声明 props + emits
function updateLater() {
  model.value = 'new value'
}
</script>
```

`defineModel` 会自动生成 `modelValue` props 和 `update:modelValue` emit，组件代码量减半。

### Teleport 是什么？解决了什么问题？

`<Teleport>` 是 Vue 3 内置组件，作用是**把组件内容渲染到 DOM 树的任意位置**，但保留组件的逻辑归属。

**典型场景**：Modal、Drawer、Tooltip、Notification——它们逻辑上属于触发点的子组件，但视觉上必须渲染到 body（避免被父级 `overflow: hidden`、`transform`、`z-index` 层叠上下文影响）。

```vue
<template>
  <button @click="open = true">打开</button>
  <Teleport to="body">
    <div v-if="open" class="modal">
      <h2>{{ title }}</h2>
      <slot />
    </div>
  </Teleport>
</template>
```

要点：

1. `to` 属性接收 CSS 选择器或 DOM 元素，指定挂载目标。
2. **逻辑上仍是当前组件的子节点**——props/emit/inject 全部正常工作。
3. `disabled` 属性可以临时取消 teleport（比如响应式切换是否传送）。
4. 多个 Teleport 传送到同一目标时，按渲染顺序追加。

Vue 2 时代要实现这个能力得自己用 `appendChild` 操作 DOM，且很难和组件生命周期对齐；Teleport 把这件事内置成一等公民。

### Suspense 怎么用？有什么限制？

`<Suspense>` 用于处理**异步组件的 loading / error 状态**，让你声明式地展示 fallback。

```vue
<template>
  <Suspense>
    <template #default>
      <AsyncUserProfile :id="userId" />
    </template>
    <template #fallback>
      <Spinner />
    </template>
  </Suspense>
</template>
```

要点：

1. 子组件可以用 `async setup()`，Suspense 会等它的 setup Promise resolve 后才渲染。
2. 嵌套 Suspense 支持，各自管理 fallback。
3. **目前仍是实验性 API**——3.x 整个版本里都标着 experimental，但生产中已被广泛使用，大部分场景稳定。
4. 错误处理需要配合 `onErrorCaptured` 或 `errorCaptured` 钩子，Suspense 自身不直接展示 error。

**限制和坑：**

1. async setup 只能用在 Suspense 包裹的组件下，不然会报错。
2. fallback 期间 Suspense 不会触发 mounted 等生命周期，要等真正内容渲染完。
3. 配合 Vue Router 时，路由切换的过渡和 Suspense 状态有时会冲突，需要小心组织。

### Vue 3 中 watch 和 watchEffect 有什么区别？

两者都是用来监听响应式数据变化执行副作用，但触发方式和适用场景不同。

| 维度 | `watch` | `watchEffect` |
| --- | --- | --- |
| 依赖来源 | 显式指定（第一个参数） | 自动收集回调里读过的响应式数据 |
| 是否立即执行 | 默认懒执行（要 `{ immediate: true }`） | 立即执行一次（用于初始收集依赖） |
| 能否拿到新旧值 | 能 | 不能（只有当前值） |
| 多源监听 | 可以传数组同时监听多个 | 只能在回调里读 |
| 适合场景 | 需要旧值、有条件触发、明确依赖 | 仅做副作用，不关心新旧对比 |

**配置项：**

```js
watch(source, callback, {
  immediate: true,      // 立即执行一次
  deep: true,           // 深度监听对象内部变化
  flush: 'post',        // 'pre' | 'post' | 'sync'，控制回调执行时机
  once: true,           // 3.4+，只触发一次后自动停止
})
```

`flush` 三个值：

1. `'pre'`（默认）：DOM 更新前执行。
2. `'post'`：DOM 更新后执行（可读到最新 DOM）。
3. `'sync'`：同步执行，不推荐（可能引发性能问题）。

### Vue 3 还有哪些常用的响应式 API？

除了 `ref`/`reactive`/`computed`/`watch`，Vue 3 还提供一组更细粒度的工具：

| API | 作用 |
| --- | --- |
| `shallowRef` | 只对 `.value` 替换做响应，内部对象变化不响应。适合大对象做整体替换 |
| `shallowReactive` | 只对第一层属性响应，深层不递归代理 |
| `readonly` | 创建只读响应式代理，常用于 inject 数据防误改 |
| `shallowReadonly` | 只读 + 只第一层 |
| `toRef` | 把 reactive 对象的某个字段转成 ref，保持双向同步 |
| `toRefs` | 把 reactive 对象所有字段转成 ref 集合，常用于解构 |
| `toRaw` | 拿到响应式对象背后的原始对象 |
| `markRaw` | 标记一个对象永远不被代理（如第三方实例、Class） |
| `customRef` | 自定义 ref 的依赖追踪和触发逻辑（实现防抖 ref 等） |
| `isRef` / `isReactive` / `isProxy` | 类型判断 |
| `unref` | `isRef(x) ? x.value : x` 的简写 |
| `effectScope` | 创建独立的 effect 作用域，方便统一停止 |

**典型用法：**

```js
// toRefs 配合解构
function useUser() {
  const state = reactive({ name: 'Ada', age: 30 })
  return toRefs(state)  // 解构后仍是响应式
}
const { name, age } = useUser()

// markRaw 防止代理
const state = reactive({
  map: markRaw(new ThirdPartyMap())  // 不希望被 Proxy 包装
})

// customRef 实现防抖
function debouncedRef(value, delay = 200) {
  let timer
  return customRef((track, trigger) => ({
    get() { track(); return value },
    set(newVal) {
      clearTimeout(timer)
      timer = setTimeout(() => { value = newVal; trigger() }, delay)
    },
  }))
}
```

### Vue 3 异步组件和 defineAsyncComponent 怎么用？

异步组件是路由懒加载、按需加载的基础。Vue 3 的写法：

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() => import('./Heavy.vue'))

// 完整配置
const AsyncCompFull = defineAsyncComponent({
  loader: () => import('./Heavy.vue'),
  loadingComponent: Loading,        // 加载中
  errorComponent: Error,             // 加载失败
  delay: 200,                        // 显示 loading 前等待 ms
  timeout: 3000,                     // 超时则展示 error
  suspensible: true,                 // 是否参与 Suspense
  onError(err, retry, fail, attempts) {
    if (attempts <= 3) retry()
    else fail()
  }
})
```

异步组件常和 `<Suspense>` / 路由懒加载组合使用，是性能优化（按需加载）的标配。

### Vue 3 中 provide/inject 有什么最佳实践？

Vue 3 的 provide/inject 升级了类型安全，并支持 readonly 防误改。

**1. 用 InjectionKey 做类型安全：**

```ts
import type { InjectionKey, Ref } from 'vue'
import { provide, inject, ref, readonly } from 'vue'

interface UserContext {
  user: Ref<User | null>
  login: (form: LoginForm) => Promise<void>
}

export const UserKey: InjectionKey<UserContext> = Symbol('user')

// 父组件
const user = ref<User | null>(null)
provide(UserKey, {
  user: readonly(user),       // 暴露只读，避免子组件直接改
  login: async (form) => { user.value = await api.login(form) },
})

// 子组件
const ctx = inject(UserKey)
if (!ctx) throw new Error('UserKey must be provided')
ctx.user.value  // 类型是 Readonly<Ref<User | null>>
```

**2. 默认值和工厂函数：**

```ts
const theme = inject('theme', 'light')              // 提供默认值
const config = inject('config', () => makeConfig(), true) // 工厂函数 + true 表示该函数本身是工厂
```

**3. 应用级 provide：**

```js
const app = createApp(App)
app.provide('apiBase', '/api/v1')  // 全应用可 inject
```

适合放公共配置、单例服务（HTTP 客户端、i18n 实例等）。

### Vue 3.3、3.4、3.5 都引入了什么新特性？

Vue 3 进入稳定后，每个小版本都在持续优化。

**Vue 3.3（2023.5）：**

1. **泛型组件**：`<script setup lang="ts" generic="T">` 支持泛型组件。
2. **`defineSlots`**：声明插槽类型。
3. **更好的导入类型支持**：`defineProps<MyProps>()` 可以从外部 `.ts` 文件导入类型。
4. **`defineModel`**（实验性）：双向绑定的简写。
5. **`toRef` 增强**：可以接收一个 getter 函数，返回派生 ref。

**Vue 3.4（2023.12）：**

1. **`defineModel` 稳定**：取代原来的"声明 props + emit + 监听"三件套。
2. **新模板解析器**：解析速度提升约 2 倍。
3. **`watch` 更细粒度**：新增 `once: true`、deep 可传数字（控制深度）。
4. **`v-bind` 同名简写**：`<div :id />` 等价于 `<div :id="id" />`。
5. **更好的 Hydration 错误信息**。

**Vue 3.5（2024.9）：**

1. **响应式 props 解构**：`const { count = 0 } = defineProps<{ count?: number }>()`，解构后仍保持响应式（编译器自动处理）。
2. **`useId`**：生成 SSR 安全的唯一 ID。
3. **`useTemplateRef`**：替代 `ref` 拿模板引用，更直观。
4. **响应式系统重写**：内部用更高效的依赖追踪算法，内存占用减少约 56%。
5. **延迟 Teleport**：`<Teleport defer>` 可以等目标 DOM 出现后再传送。
6. **`onWatcherCleanup`**：watcher 内显式注册清理逻辑，替代旧的 onCleanup 参数。

如果团队还在 3.0/3.1，升级到 3.5 几乎全是性能和 DX 收益。

### Vue 3 的编译时优化具体做了什么？

Vue 3 的编译器是性能提升的核心，可以用一段模板对照编译产物来理解。

**模板：**

```vue
<div>
  <h1>Title</h1>
  <p>{{ msg }}</p>
  <button @click="onClick">click</button>
</div>
```

**Vue 2 编译大致产物：**

```js
function render() {
  return _c('div', [
    _c('h1', ['Title']),                          // 每次都创建
    _c('p', [_v(_s(msg))]),                       // 每次都创建
    _c('button', { on: { click: onClick } }, ['click'])  // 每次新事件
  ])
}
```

**Vue 3 编译大致产物：**

```js
const _hoisted_1 = createElementVNode('h1', null, 'Title')  // 静态提升

function render(_ctx, _cache) {
  return (openBlock(), createElementBlock('div', null, [
    _hoisted_1,                                              // 复用同一 vnode
    createElementVNode('p', null, toDisplayString(_ctx.msg), 1 /* TEXT */),  // PatchFlag: 仅 text 动态
    createElementVNode('button', {
      onClick: _cache[0] || (_cache[0] = (...args) => _ctx.onClick(...args))  // 事件缓存
    }, 'click')
  ]))
}
```

四类优化清楚体现：

1. **静态提升**：`<h1>Title</h1>` 完全静态，编译时提到 render 函数外，每次复用同一对象。
2. **PatchFlag**：`<p>` 标了 `1 /* TEXT */`，运行时 diff 时只比对 text，跳过 props 和 children 比较。
3. **事件缓存**：`onClick` 通过 `_cache[0] ||` 缓存，避免每次 render 创建新函数（这点对 `React.memo` 类似优化天然兼容）。
4. **Block Tree**：`openBlock()` 把这棵树的所有动态子节点拍平到一个数组，diff 时只遍历这个数组而不是整棵树。

理解这点能解释一个常见追问："Vue 3 为什么号称比 React 快？"——核心就在编译时对静态/动态的精准分离。

### Vue 3 自定义渲染器是什么？有什么用？

Vue 3 把渲染器从核心库中抽出来，提供了 `createRenderer` API，让用户可以自己定义"如何把 vnode 变成目标平台的元素"。

```js
import { createRenderer } from '@vue/runtime-core'

const renderer = createRenderer({
  createElement(type) { /* 平台相关创建节点 */ },
  insert(child, parent) { /* 平台相关插入 */ },
  remove(el) { /* ... */ },
  patchProp(el, key, prev, next) { /* ... */ },
  // ... 其他 DOM 操作的平台等价物
})

renderer.createApp(App).mount(target)
```

应用场景：

1. 渲染到 Canvas（如 `vue-canvas-renderer`）。
2. 渲染到 WebGL（如 TresJS、Trois.js 把 Vue 组件渲染成 Three.js 场景）。
3. 渲染到原生（NativeScript-Vue）。
4. 渲染到 SSR HTML 字符串（`@vue/server-renderer`）。
5. 渲染到自定义 DSL / 测试虚拟环境。

这是 Vue 3 跨平台能力的基石，也是 Vue 团队拥抱"非 DOM 场景"的关键设计。

### Vue3.0 的设计目标是什么？做了哪些优化？

Vue3 的设计目标可以概括为：更快、更小、更易维护、更友好地支持 TypeScript。

主要优化包括：

1. 用 `Proxy` 重构响应式系统。
2. 引入 Composition API，提升逻辑组织和复用能力。
3. 编译阶段做更多静态提升和 Patch Flag 优化。
4. 更好的 Tree Shaking，减少最终打包体积。
5. 内部结构模块化，便于维护和扩展。
6. 更完整的 TS 类型推导能力。

### Vue3.0 性能提升主要是通过哪几方面体现的？

Vue3 性能提升主要体现在：

1. 响应式性能：`Proxy` 对对象和数组拦截更完整。
2. 编译优化：静态节点提升，避免重复创建。
3. Patch 优化：通过 Patch Flag 精准定位动态内容，减少无效 diff。
4. 事件缓存：避免重复创建事件函数。
5. 更好的 Tree Shaking：未使用功能不进入最终包体。
6. 列表和块树优化：减少不必要遍历。

所以 Vue3 的性能提升，不只是运行时优化，也包括编译时优化。

### Vue3.0 里为什么要用 Proxy API 替代 defineProperty API？

因为 `Object.defineProperty` 存在明显局限：

1. 不能监听对象新增和删除属性。
2. 无法直接监听数组下标变化和长度变化。
3. 需要递归遍历对象初始化，性能成本较高。

`Proxy` 的优势在于：

1. 可以代理整个对象，而不是逐个属性劫持。
2. 能监听新增、删除、in 操作、遍历等行为。
3. 对数组支持更自然。
4. 更适合构建完整的响应式系统。

所以 Vue3 选择 Proxy，本质上是为了解决 Vue2 响应式边界问题，并提升系统扩展性。

### Vue3.0 所采用的 Composition API 与 Vue2.x 使用的 Options API 有什么不同？

Options API 是按配置项组织代码，比如 `data`、`methods`、`computed` 分散定义。

Composition API 是按逻辑功能组织代码，把同一个业务能力相关的状态、计算、方法聚合在一起。

两者区别主要在：

1. 组织方式不同：一个按选项类型，一个按业务逻辑。
2. 复用方式不同：Options API 常用 mixin，Composition API 常用自定义 hooks。
3. 可维护性不同：复杂组件下 Composition API 更适合拆分和组合。
4. 类型推导不同：Composition API 对 TS 更友好。

所以大型项目里，Composition API 在逻辑复用和维护性上通常更占优势。

### 说说 Vue 3.0 中 Treeshaking 特性？举例说明一下？

Tree Shaking 指的是在打包阶段移除未使用代码，减少最终包体积。

Vue3 之所以更适合 Tree Shaking，是因为它的 API 设计和内部模块拆分更加函数化、模块化。

举例来说：

1. 如果项目只用了 `ref`、`computed`，没有用 `watch`，那未使用部分有机会在打包时被移除。
2. 某些内置特性没有被使用，也不会全部打进最终包。

它的前提通常是：

1. 使用 ES Module。
2. 构建工具支持 Tree Shaking，比如 Vite、Rollup、Webpack。

### 用 Vue3.0 写过组件吗？如果想实现一个 Modal 你会怎么设计？

如果让面试官看到设计能力，回答不要只说“写个弹窗”，而要从 API、结构、扩展性、交互细节去讲。

一个较完整的 Modal 设计可以这样说：

1. 基础属性：`modelValue` 控制显示隐藏，`title`、`width`、`fullscreen`、`destroyOnClose` 等控制表现。
2. 事件设计：`update:modelValue`、`open`、`close`、`confirm`、`cancel`。
3. 插槽设计：头部、内容区、底部按钮区都支持插槽，便于扩展。
4. 渲染方式：使用 `Teleport` 挂到 `body`，避免被父容器层级和样式影响。
5. 交互细节：支持 ESC 关闭、遮罩点击关闭、滚动穿透处理、焦点管理。
6. 动画能力：进入离开过渡效果。
7. 可访问性：考虑键盘操作、语义属性和焦点陷阱。
8. 命令式调用：复杂项目里还可以封装 `useModal` 或服务式调用。

如果再进一步，可以补一句：简单弹窗是组件问题，复杂弹窗系统是组件库设计问题。

## 附：常见代码实现

### 1. `computed` 和 `watch` 示例

```javascript
export default {
  data() {
    return { firstName: 'Ada', lastName: 'Lovelace', keyword: '' }
  },
  computed: {
    fullName() {
      return this.firstName + ' ' + this.lastName
    }
  },
  watch: {
    keyword(newVal) {
      this.fetchList(newVal)
    }
  },
  methods: {
    fetchList(keyword) {
      console.log('请求接口', keyword)
    }
  }
}
```

### 2. `provide / inject` 示例

```javascript
// Parent.vue
export default {
  provide() {
    return { theme: 'dark' }
  }
}

// Child.vue
export default {
  inject: ['theme'],
  mounted() {
    console.log(this.theme)
  }
}
```

### 3. 自定义指令示例

```javascript
app.directive('focus', {
  mounted(el) {
    el.focus()
  }
})
```
