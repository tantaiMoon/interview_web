[toc]

## 一、Flutter 基础认知

### 什么是 Flutter？它的优势是什么？

Flutter 是 Google 出的跨端 UI 框架，用 Dart 写代码，编译产物能跑 iOS / Android / Web / Windows / macOS / Linux / 嵌入式。最近几年迭代很快，是跨端方案里最受关注的一套。

它跟其他跨端方案最大的差别是**自绘渲染**——不调用系统组件，直接用 Skia（新版本是 Impeller）画每一个像素。也就是说：

- **像素级一致**：iOS 和 Android 的 Material / Cupertino 控件，Flutter 自己实现一遍，多端表现完全一样
- **性能稳定**：没有"原生组件 + JS Bridge"的转换成本，Dart 直接生成机器码（AOT），帧率接近原生
- **支持复杂动效**：绘制自定义曲线、做高复杂度动画都成本较低
- **代价**：包体积比 RN 大（要带渲染引擎），首次启动稍慢；调试要懂 Dart 和 Widget 树

### Flutter 和 React Native 的主要区别是什么？

| 维度 | Flutter | React Native |
|------|---------|--------------|
| 语言 | Dart | JS / TS |
| 渲染 | 自绘（Skia / Impeller） | 调用原生控件 |
| UI 一致性 | 各端完全一致 | 跟随系统，可能有差异 |
| 性能 | AOT 编译，接近原生 | JS Bridge 有开销（新架构 JSI 改善） |
| 包体积 | 较大（带引擎 ~5MB+） | 较小 |
| 团队迁移 | 要学 Dart + Widget 体系 | 会 React 就能上手 |
| 生态 | Google 强力推进，pub.dev 活跃 | Meta + 社区，npm 生态 |
| 调试 | DevTools 强 | Chrome / Flipper |

选型：UI 一致性要求高、动画复杂、团队没历史包袱选 Flutter；团队已经是 React 技术栈、要复用 Web 代码、迭代偏业务选 RN。

### Flutter 为什么能做到跨平台？

核心就两点：**统一语言 + 统一渲染**。

- **统一语言**：业务和 UI 都用 Dart，没有 JS-原生的边界
- **统一渲染**：上层 Widget 描述"UI 应该长啥样"→ Flutter Engine 负责布局、合成、绘制 → 底层用 Skia / Impeller 把 GPU 命令交给系统图形栈（Metal / Vulkan / OpenGL）

跟 RN 的"调原生组件"完全不同——Flutter 几乎不依赖各端的 UI 系统，平台只是给它一块画布。所以 iOS、Android 看到的按钮本质是 Flutter 自己画的，不是 UIButton / android.widget.Button。

### Dart 为什么适合 Flutter？

Dart 不是 Google 临时找的语言，是专门为 Flutter 这种场景设计的。几个关键能力：

- **JIT + AOT 双模式**：开发时 JIT（即时编译）支持 Hot Reload，发布时 AOT（提前编译）生成原生机器码，性能跟 C++ 一个档次
- **没有 GIL，多 Isolate 并发**：Isolate 之间不共享内存，靠消息传，天然避免数据竞争
- **空安全（Null Safety）**：Dart 2.12 起强制空安全，编译期消灭 NPE
- **类似 JS 但有完整类型系统**：JS / Java / Kotlin 开发者上手都成本较低
- **`async/await` + `Stream`**：异步模型对前端来说很熟悉

关键的是它的**对象分配开销低**——Widget 是不可变的，每帧都要重建大量 Widget 实例，没有 Dart 的快速分配 + GC，这个模型很难保持流畅。

### Hot Reload 和 Hot Restart 有什么区别？

- **Hot Reload**：改代码后注入到运行中的 VM，重建 Widget 树但**保留 State 状态**。500ms 左右完成，日常开发主力
- **Hot Restart**：重启整个 Dart VM，从 `main()` 重新跑，**所有状态清零**。改了 `main()` 入口、改了 `initState` 逻辑、修了全局变量初始化时用它

简记：调样式 / 改 UI 用 Hot Reload；改启动流程 / 全局状态用 Hot Restart。


## 二、核心原理

### Flutter 的整体架构是怎样的？

可以按 3 层来理解：

1. **Framework**
   - Dart 编写
   - 提供 Widget、动画、手势、状态管理、渲染抽象

2. **Engine**
   - 主要由 C++ 实现
   - 负责文本布局、渲染、Dart Runtime、平台通信等

3. **Embedder**
   - 平台接入层
   - 负责把 Flutter 嵌入 Android、iOS、桌面等系统环境中

典型链路：

```text
Dart 业务代码
  ↓
Flutter Framework
  ↓
Flutter Engine
  ↓
Platform Embedder
  ↓
操作系统 / GPU
```

---

### Widget、Element、RenderObject 分别是什么？

这是 Flutter 里一类比较基础的原理题。

**Widget：**

- 描述 UI 的配置
- 是不可变对象
- 关注“长什么样”

**Element：**

- Widget 在运行时的实例
- 连接 Widget 和 RenderObject
### Widget、Element、RenderObject 分别是什么？

Flutter 用三棵树来管理 UI。理解这三棵树是 Flutter 高阶面试的核心：

```
Widget Tree        Element Tree       RenderObject Tree
（描述/不可变）  →  （管理/可变）   →   （布局+绘制）
```

- **Widget**：UI 配置，**不可变**（每次 build 都新建一个）。说"这里应该有个红色按钮"，仅此而已。`Widget` 类本身只是个数据结构，没有渲染逻辑。
- **Element**：Widget 在树里的**实例化**和**位置**。新旧 Widget 进来时，Element 决定是更新自己引用的 Widget，还是销毁重建。生命周期、状态、context 都挂在 Element 上。
- **RenderObject**：真正负责布局（measure）、绘制（paint）、命中测试（hit-test）。一个 RenderBox 知道自己有多大、画什么、能不能接收点击。

每帧 setState 后，三棵树的协作流程：

1. `build()` 返回新的 Widget 树（廉价，纯数据）
2. Element 树拿新旧 Widget 比对：`canUpdate()` 通过就更新引用，不通过就销毁子树重建（这一步就是 Flutter 的 diff）
3. Element 把变化推给 RenderObject，触发布局 / 绘制

为什么要拆三棵：Widget 不可变保证了声明式 UI 的可预测性，但每帧重建太贵；Element 维持身份，让状态、订阅、context 不会因为重建丢失；RenderObject 跟视图栈紧绑定，把昂贵的布局绘制工作隔离开。

---

### Flutter 中为什么说"一切皆 Widget"？

布局、装饰、动画、主题、路由、对齐方式……在 Flutter 里全是 Widget。`Padding` 是 Widget、`Center` 是 Widget、`Theme` 是 Widget，连 `MediaQuery` 也是 Widget。

```dart
Padding(
  padding: const EdgeInsets.all(16),
  child: Center(
    child: Text('Hello Flutter'),
  ),
)
```

这种设计的好处：

- **统一抽象**：UI 的所有维度都靠组合 Widget 表达，没有第二套配置 DSL
- **可组合**：任意 Widget 可以包另一个，逻辑跟视觉混在一起也没关系
- **数据驱动**：所有 UI 状态都映射到 Widget 树，diff 一下就能更新

代价：嵌套深、可读性偶尔差，但 IDE 折叠 + extract widget 重构习惯了就好。

### StatelessWidget 和 StatefulWidget 有什么区别？

- **StatelessWidget**：纯展示组件，UI 完全由构造参数决定，没有内部状态。比如一个 `Avatar(url: ...)`、`Label(text: ...)`
- **StatefulWidget**：UI 会随时间或交互变化，需要内部状态。比如计数器、表单、动画、可展开列表

`StatefulWidget` 本身也是不可变的，状态藏在伴生的 `State` 对象里。`State` 才有 `initState` / `dispose` / `build` 这些生命周期方法。

判断标准：UI 是否会自己改变？只靠 props 不能完全决定时使用 `StatefulWidget`。表单类通常需要状态，列表项 / 卡片 / 文本展示这类组件通常使用 `StatelessWidget`。

性能上：`Stateless` 更轻量，`Stateful` 会创建 State 对象并走完整生命周期。不需要状态时优先使用 `StatelessWidget`。

### setState 的原理是什么？

`setState` 做的事情就一件：把当前 State 对应的 Element 标记为 "dirty"，加入下一帧的更新队列。

更详细流程：

1. `setState(() { ... })` 同步执行回调，更新 State 字段
2. 调用 `markNeedsBuild()` 把 Element 标 dirty
3. 调度器在下一帧 vsync 时，遍历所有 dirty Element 调 `rebuild`
4. `rebuild` 调用对应 State 的 `build()` 返回新 Widget
5. 跟旧 Widget 比对，只更新差异部分（runtimeType + key 相同就复用 Element，否则销毁重建）
6. 触发 RenderObject 布局 + 绘制

要点：

- **不能在 build 里调 setState**——会无限递归
- **build 里别做副作用**——build 可能因为重建一帧跑多次，副作用要放 `initState` / `didChangeDependencies`
- **setState 是异步的视觉效果**——回调里改值是同步的，但屏幕看到变化要等下一帧
- **不要忘了 `mounted` 检查**——异步回调里 setState 前先判 `if (!mounted) return`，否则页面已销毁会报错


### BuildContext 是什么？有哪些常见问题？

`BuildContext` 可以看成当前 Widget 在树中的位置句柄。

常见用途：

- `Theme.of(context)`
- `MediaQuery.of(context)`
- `Navigator.of(context)`
- 查找祖先组件

常见问题：

1. **异步回调里直接使用旧 context**
   - 页面可能已经销毁

2. **在不合适的生命周期里依赖 context**
   - 比如一些依赖继承树的初始化逻辑不适合直接放在构造阶段

3. **组件销毁后仍操作导航或弹窗**
   - 需要先判断 `mounted`

---

### Key 的作用是什么？

`Key` 的作用是：**帮助 Flutter 在重建时识别组件身份，正确复用状态。**

如果没有 Key，Flutter 默认按：

- 类型
- 同层位置

来判断节点是否复用。

在列表插入、删除、重排时，如果身份识别错误，就可能导致：

- 输入框内容错位
- 动画状态错乱
- 列表项状态串位

常见 Key：

- `ValueKey`
- `ObjectKey`
- `UniqueKey`
- `GlobalKey`

其中 `GlobalKey` 功能最强，但成本也最高，不能滥用。

---

## 三、生命周期与渲染

### StatefulWidget 的生命周期有哪些？

常见顺序如下：

1. `createState`
2. `initState`
3. `didChangeDependencies`
4. `build`
5. `didUpdateWidget`
6. `deactivate`
7. `dispose`

这里要重点说明：

- 初始化请求、订阅、控制器创建通常放 `initState`
- 依赖继承组件变化时会触发 `didChangeDependencies`
- 释放资源必须放 `dispose`

---

### Flutter 的渲染流程是怎样的？

高频回答可以这样说：

1. 开发者写出 Widget 树
2. Framework 生成或更新 Element 树
3. Element 关联 RenderObject
4. RenderObject 进行布局和绘制
5. 渲染结果提交到底层图形系统显示到屏幕

如果再简化几个字：

> Widget 负责描述，Element 负责连接和管理，RenderObject 负责布局和绘制。

---

### Flutter 的布局规则是什么？

Flutter 的布局可以用一句很经典的话概括：

> Constraints go down, sizes go up, parent sets position.

也就是：

1. 父组件把约束传给子组件
2. 子组件在约束范围内决定自己的尺寸
3. 父组件决定子组件的位置

这也是不少布局问题排查时最先会想到的思路。

---

### BoxConstraints 是什么？

`BoxConstraints` 是 Flutter 盒模型布局中的约束对象，描述了子组件允许的：

- 最小宽度
- 最大宽度
- 最小高度
- 最大高度

理解它重要，因为不少布局报错都和约束冲突有关，比如：

- `RenderFlex overflowed`
- 无界高度 / 无界宽度
- 嵌套滚动布局异常

---

## 四、状态管理

### Flutter 中常见的状态管理方案有哪些？

常见回答如下：

1. **setState**
   - 适合局部简单状态

2. **InheritedWidget**
   - Flutter 原生共享状态方案

3. **Provider**
   - 基于 `InheritedWidget` 的轻量封装

4. **Riverpod**
   - 依赖关系更清晰，更利于测试

5. **Bloc / Cubit**
   - 事件流、状态流分离，适合中大型项目

6. **GetX / MobX**
   - 各自提供响应式能力和工程化体验

---

### Provider 的思路是什么？

Provider 做的事情：

- 把共享状态放到组件树上层
- 下层组件按需读取
- 状态变化后只更新依赖它的组件

它解决了两个核心问题：

1. 避免层层手动传参
2. 把状态管理从页面 UI 中拆出来

适合：

- 中小型项目
- 团队希望低学习成本快速落地

---

### Bloc 模式适合什么场景？

Bloc 更强调：

- 事件驱动
- 状态不可变
- UI 和业务逻辑分离

排查流程：

```text
UI 触发 Event
  ↓
Bloc 处理业务逻辑
  ↓
输出新的 State
  ↓
UI 根据 State 重建
```

适合用在：

- 业务复杂
- 页面状态多
- 团队要求结构规范
- 需要更强可测试性

---

### InheritedWidget 的作用是什么？

`InheritedWidget` 是 Flutter 原生的跨层级状态共享机制。

当祖先节点中的共享数据变化时，依赖这个数据的子孙节点可以自动收到更新通知。

它是不少状态管理方案的基础，比如：

- Provider
- 主题传递
- 媒体信息传递

---

### Provider、Riverpod、Bloc 应该怎么选？

这是 Flutter 里很常见的选型问题，不适合直接判断“哪个最好”，更重要的是说明“哪个更适合当前场景”。

**Provider：**

- 学习成本低
- 接近 Flutter 原生思路
- 适合中小型项目

**Riverpod：**

- 依赖关系更明确
- 不强依赖 `BuildContext`
- 更利于测试和模块化

**Bloc：**

- 事件和状态边界清晰
- 更适合复杂业务流
- 团队规范化协作更强

**简化选型建议：**

- 小中型项目：`Provider` / `Riverpod`
- 复杂业务项目：`Bloc` / `Cubit`
- 强调可测试性和依赖清晰：优先考虑 `Riverpod`

---

### 为什么说 Riverpod 比 Provider 更容易测试？

原因在于：**Riverpod 的依赖读取不需要强绑定 Widget 树和 `BuildContext`。**

这带来几个好处：

1. 状态提供者更独立
2. 依赖注入和替换更方便
3. 单元测试时不一定要先搭 Widget 树
4. 全局依赖关系更显式

如果被问区别，可以直接说：

> Provider 更轻、更接近 Widget 树；Riverpod 更独立、更适合测试和复杂依赖管理。

---

### Bloc 和 Cubit 有什么区别？

二者都属于 Bloc 体系，但复杂度不同。

**Cubit：**

- 更轻量
- 直接通过方法修改并发出新状态
- 少一层 Event 抽象

**Bloc：**

- 基于 Event -> State 流转
- 更适合复杂交互、异步流程和规范化事件建模

简单看：

- `Cubit`：轻量版 Bloc
- `Bloc`：标准事件驱动架构

---

### 状态管理中常见的问题有哪些？

可以从这几个角度回答：

1. **把所有状态都放全局**
   - 会导致状态边界混乱

2. **UI、业务、网络请求耦合在一个类里**
   - 难测试、难维护

3. **刷新粒度过粗**
   - 一个状态变化导致整个页面重建

4. **异步状态缺少 loading / error / empty 管理**
   - 页面状态不完整

5. **滥用单例和全局变量**
   - 导致依赖不可控

高频总结句：

> 状态管理最大的坑不是“用错框架”，而是状态边界、职责拆分和刷新粒度设计得不好。

---

### Flutter 中如何设计状态分层？

如果把问题落到“状态该怎么拆”，可以这样回答：

1. **局部 UI 状态**
   - 例如 tab 切换、表单输入、展开收起
   - 使用 `setState` 通常即可

2. **页面级状态**
   - 例如列表数据、详情数据、加载状态
   - 适合 `Provider`、`Riverpod`、`Cubit`

3. **全局共享状态**
   - 例如登录态、主题、语言、权限信息
   - 放到应用级状态容器

处理原则：

- 状态尽量靠近使用它的地方
- 不要为了“统一”把所有状态都上提成全局状态

---

### Flutter 中如何处理异步状态？

异步状态通常至少要拆成几类：

1. `loading`
2. `success`
3. `empty`
4. `error`

继续往下拆，重点在这里：

- 不要只维护一份 data
- 要把请求过程状态显式建模出来
- 这样 UI 才能正确展示骨架屏、空态、错误态、重试按钮

在 `Bloc`、`Riverpod`、`Provider` 中，这种思想都一样，只是表达方式不同。

---

## 五、异步与并发

### Future、async/await、Stream 的区别是什么？

**Future：**

- 表示未来某个时间点返回一次结果

**async/await：**

- 编写异步逻辑的语法糖

**Stream：**

- 表示持续不断的数据流

常见场景：

- 获取用户信息：`Future`
- 监听聊天消息：`Stream`
- 编写异步流程控制：`async/await`

---

### Isolate 是什么？为什么 Flutter 需要它？

Flutter 的 UI 线程不能长时间做重计算，否则容易卡顿掉帧。

`Isolate` 可以看成 Dart 的并发执行单元，它的特点是：

- 内存隔离
- 不能直接共享对象
- 通过消息通信

适合放进 `Isolate` 的任务：

- 大 JSON 解析
- 图片处理
- 加解密
- 大量计算任务

---

## 六、工程化与跨端能力

### pubspec.yaml 的作用是什么？

`pubspec.yaml` 是 Flutter 项目的核心配置文件，常见用途有：

1. 声明项目基础信息
2. 管理依赖包
3. 配置静态资源
4. 配置字体
5. 指定环境约束

顺手可以补上这一点：

> 依赖、资源、字体、版本约束，基本都要看 `pubspec.yaml`。

---

### package 和 plugin 的区别是什么？

**package：**

- 纯 Dart 或纯 Flutter 层面的可复用代码

**plugin：**

- 除了 Dart 代码外，还包含原生平台实现
- 用于访问摄像头、定位、蓝牙、推送等系统能力

所以：

- 纯逻辑复用，常用 `package`
- 涉及原生能力接入，通常是 `plugin`

---

### Flutter 如何和原生 Android / iOS 通信？

Flutter 通过 `Platform Channel` 与原生层通信。

常见类型：

1. `MethodChannel`
   - 常见用，请求一个原生方法并返回结果

2. `EventChannel`
   - 原生持续推送事件给 Flutter

3. `BasicMessageChannel`
   - 双向消息通信

常见场景：

- 获取电量
- 调用相机
- 打开蓝牙
- 接入第三方原生 SDK

---

### Flutter 中如何实现页面导航？

常见答案分两类：

1. **Navigator 1.0**
   - 命令式导航
   - `push`、`pop` 易于理解

2. **Navigator 2.0 / Router**
   - 声明式导航
   - 更适合 Web、深链、复杂路由管理

项目里也经常用：

- `go_router`
- `auto_route`

来降低路由维护成本。

---

### Navigator 1.0 和 Navigator 2.0 的主要区别是什么？

**Navigator 1.0：**

- 命令式导航
- 通过 `push`、`pop` 直接操作路由栈
- 上手简单

**Navigator 2.0：**

- 声明式导航
- 由状态决定当前页面栈
- 更适合 Web、深链、复杂嵌套路由

常见的总结就是：

> Navigator 1.0 更像“手动操作栈”，Navigator 2.0 更像“根据状态声明路由结果”。

---

### go_router 为什么在项目里很常用？

`go_router` 流行的主要原因：

1. 官方生态背景更强
2. 封装了 `Navigator 2.0` 的复杂性
3. 更方便处理深链
4. 支持嵌套路由
5. 支持路由守卫和重定向
6. 对 Web 支持更自然

对于大部分团队来说，它的价值不是“更底层”，而是“维护成本更低”。

---

### auto_route 和 go_router 的区别可以怎么回答？

这道题不需要答得太绝对，重点说风格差异：

**go_router：**

- 更偏声明式配置
- 官方生态接受度高
- Web / deep link 场景常见

**auto_route：**

- 更强调代码生成
- 对复杂嵌套路由、大型路由表管理比较友好

选型上可以这样区分：

- 喜欢轻配置、贴近官方生态：`go_router`
- 喜欢代码生成和强类型路由：`auto_route`

---

### Flutter 中什么是深层链接（Deep Link）？

Deep Link 指的是：

- 从外部链接直接打开 App 内某个具体页面

比如：

- 打开商品详情页
- 打开活动落地页
- 打开订单页

它在以下场景很常见：

- 短信跳转
- H5 唤起 App
- 推送通知点击跳转
- Web 路由映射

这个问题可以补充一点：

> Deep Link 通常会和路由管理、登录态校验、参数解析、页面恢复一起出现。

---

### Flutter 路由守卫一般怎么做？

适合用在：

- 未登录不能进入个人中心
- 权限不足不能进入某页面
- 首次安装先进入引导页

常见实现思路：

1. 在路由跳转前统一判断登录态 / 权限 / 初始化状态
2. 不满足条件时重定向到登录页或引导页
3. 登录成功后再跳回目标页

如果用 `go_router`，通常会使用：

- `redirect`

来实现统一路由守卫。

---

### Flutter 中如何做路由传参？

常见方式有三类：

1. **构造参数**
   - 最直接、类型最清晰

2. **命名路由参数**
   - 适合统一路由表管理

3. **URL query / path 参数**
   - 常见于 Web 和 Deep Link 场景

顺手可以补上这一点：

> 如果项目有 Web 或 Deep Link 需求，路由参数设计最好贴近 URL 语义，而不是只依赖内存对象传递。

---

### Flutter 路由设计有哪些常见问题？

高频问题主要有：

1. **页面路径设计混乱**
   - 后期维护困难

2. **登录态和路由耦合不清**
   - 容易出现跳转死循环

3. **参数类型不统一**
   - 页面接收参数时容易出错

4. **返回栈管理混乱**
   - 导致返回行为不符合预期

5. **Deep Link 与普通路由逻辑分裂**
   - 外部拉起和内部跳转行为不一致

标准总结句：

> 路由问题往往不在“跳转 API 怎么写”，而在路径设计、页面栈设计、参数设计和守卫策略设计。

---

## 七、性能优化

### Flutter 常见的性能优化手段有哪些？

可以从下面几类回答：

1. **减少不必要的 rebuild**
   - 拆小 Widget
   - 局部刷新
   - 避免整个页面反复 `setState`

2. **优先使用 const**
   - 减少重复创建对象

3. **长列表使用 builder**
   - 比如 `ListView.builder`

4. **重计算放到 Isolate**
   - 避免阻塞主线程

5. **适当使用 RepaintBoundary**
   - 隔离重绘区域

6. **优化图片加载**
   - 控制分辨率
   - 使用缓存
   - 避免超大图直接解码

7. **减少层级过深的嵌套**
   - 降低布局和绘制成本

---

### const 在 Flutter 中为什么重要？

`const` 表示编译期常量。

在 Flutter 中它的价值主要看：

1. **减少 Widget 重复创建**
2. **帮助框架更高效地复用不可变节点**

虽然它不是性能优化的全部，但它是最便宜、较稳定的一类优化。

---

### RepaintBoundary 的作用是什么？

`RepaintBoundary` 的作用是把某一部分 UI 隔离成独立重绘区域。

这样做的好处是：

- 局部变化时，不必让整棵树一起重绘
- 对动画区、复杂图表区、局部频繁刷新区域比较有价值

但也不能滥用，因为额外的边界也会带来内存和合成成本。

---

## 八、实战场景题

### 如果 Flutter 页面出现卡顿，你会怎么排查？

可以从这几个方向回答：

1. 看是否存在大量不必要 rebuild
2. 看列表是否一次性渲染过多节点
3. 看是否有大图解码或频繁图片重绘
4. 看主线程是否执行了重计算
5. 看动画区域是否重绘范围过大
6. 借助性能工具定位 frame 掉帧点

可以补充治理动作：

- 拆分组件
- 使用 builder
- 加 `const`
- 重计算放 `Isolate`
- 用 `RepaintBoundary`

---

### Flutter 和原生开发应该如何选？

这里不要答得太绝对，重点还是讲权衡：

适合 Flutter 的场景：

- 需要多端统一开发
- UI 自定义程度高
- 需要快速迭代
- 团队可接受 Dart 技术栈

适合原生的场景：

- 深度依赖平台能力
- 大量复杂原生 SDK 集成
- 团队已有成熟原生体系
- 对平台特性体验要求极高

---

### 如何用 1 分钟概括 Flutter？

可以概括为：

> Flutter 是 Google 推出的跨平台 UI 框架，使用 Dart 开发。它和 React Native 最大的区别在于，Flutter 不依赖系统原生控件，而是通过自己的渲染引擎直接绘制 UI，因此多端一致性通常更强。它的核心原理可以拆成 Widget、Element、RenderObject 三层：Widget 负责声明 UI，Element 负责管理和连接，RenderObject 负责布局和绘制。项目里，Flutter 适合复杂 UI、多端统一和高定制化场景，但团队需要掌握 Dart、状态管理和渲染机制。

---

## 九、Widget 与常用组件专题

### FutureBuilder 和 StreamBuilder 有什么区别？

**FutureBuilder：**

- 用于监听一次性异步结果
- 适合接口请求、初始化加载

**StreamBuilder：**

- 用于监听持续变化的数据流
- 适合聊天消息、WebSocket、下载进度、传感器数据

**主要区别：**

- `FutureBuilder` 对应单次异步
- `StreamBuilder` 对应持续异步流

---

### ListView.builder 和 ListView 有什么区别？

**ListView：**

- 适合子节点数量较少、固定的场景
- 会直接接收一组 children

**ListView.builder：**

- 适合长列表
- 按需构建列表项
- 更节省内存和首屏渲染成本

通常可以直接说：

> 短列表用 `ListView`，长列表优先用 `ListView.builder`。

---

### Sliver 是什么？为什么它重要？

`Sliver` 可以看成 Flutter 可滚动区域中的“片段化滚动组件”。

它的重要性在于：

1. 能把不同滚动效果组合到一个滚动容器中
2. 更适合复杂滚动页面
3. 支持吸顶、伸缩头部、分段列表等能力

常见组件有：

- `CustomScrollView`
- `SliverAppBar`
- `SliverList`
- `SliverGrid`

适合用在：

- 电商首页
- 复杂信息流
- 带折叠头部的详情页

---

### Expanded、Flexible 和 Spacer 有什么区别？

这三个组件都常用于 `Row` / `Column` / `Flex` 布局。

**Expanded：**

- 强制子组件占满剩余空间

**Flexible：**

- 允许子组件在剩余空间内弹性布局
- 不一定完全撑满

**Spacer：**

- 本质是一个占位的弹性空白区域

可以这样区分：

- `Expanded` 更强制
- `Flexible` 更灵活
- `Spacer` 专门拿来留空

---

### Flutter 中为什么要避免过深的 Widget 嵌套？

这里不是说“嵌套一定错”，而是过深嵌套可能带来：

1. 可读性下降
2. 布局调试困难
3. rebuild 范围不易控制
4. 局部复杂场景下增加布局和绘制成本

优化方式：

- 抽离子组件
- 复用通用布局
- 分离业务与展示

---

## 十、高级渲染与原生集成

### CustomPaint 的作用是什么？适合什么场景？

`CustomPaint` 用于自定义绘制，它允许开发者直接在画布上绘制图形。

适合用在：

- 图表
- 签名板
- 进度环
- 粒子动画
- 特殊背景效果

它的优势是灵活，但也意味着：

- 需要自己处理绘制逻辑
- 对性能和重绘边界要更敏感

---

### Flutter 中的命中测试是什么？

命中测试可按下面理解：

> 当用户点击屏幕时，Flutter 如何判断事件应该交给哪个组件处理。

它和以下能力有关：

- 手势分发
- 点击区域判断
- 事件冒泡链路
- `RenderObject` 的事件处理

一旦聊到手势系统，命中测试往往就是底层关键字。

---

### Hero 动画是什么？底层思路是什么？

`Hero` 是 Flutter 中用于页面间共享元素过渡动画的机制。

常见场景：

- 列表图进入详情页
- 卡片放大进入详情
- 头像、封面图跨页面转场

它的底层思路可以简单看为：

1. 在前后两个页面中找到相同 `tag` 的 Hero
2. 在路由切换过程中把它们做视觉上的连续过渡

这样用户会感觉是“同一个元素在移动”，而不是“一个元素消失、另一个元素出现”。

---

### PlatformView 是什么？什么时候会用到？

`PlatformView` 允许在 Flutter 页面中嵌入原生平台视图。

适合用在：

- 地图
- WebView
- 原生视频播放器
- 第三方原生 SDK 提供的 View

但这里最好把它的代价也顺手说出来：

- 嵌入原生 View 会增加跨层集成复杂度
- 某些场景下性能、手势和渲染合成会更复杂

---

### Flutter 插件开发的基本思路是什么？

如果问插件开发，可以按这个结构答：

1. Dart 层定义统一 API
2. 通过 `Platform Channel` 调用 Android / iOS 原生实现
3. 各平台分别完成具体能力接入
4. 返回结果给 Flutter 层

适合做插件的能力包括：

- 相机
- 蓝牙
- 文件系统
- 支付
- 推送

---

### Impeller 和 Skia 是什么关系？

这是近几年 Flutter 近几年更常出现的问题。

**Skia：**

- Flutter 传统使用的 2D 图形引擎

**Impeller：**

- Flutter 推进中的新渲染器
- 目标是让渲染更稳定、减少运行时着色器卡顿

不需要展开到底层实现，重点说明这一点：

> Skia 是 Flutter 长期使用的图形引擎，Impeller 是 Flutter 在新阶段推动的渲染器方向，重点是提升渲染稳定性，尤其减少首帧和动画中的 shader 卡顿问题。

---

## 十一、测试与质量保障

### Flutter 中有哪些常见测试方式？

Flutter 常见测试分三类：

1. **Unit Test**
   - 测试纯逻辑
   - 速度快

2. **Widget Test**
   - 测试单个 Widget 的行为和渲染
   - 是 Flutter 很有代表性的测试方式

3. **Integration Test**
   - 测试完整页面流程或端到端行为

这里可以强调：

> Flutter 的 Widget Test 是它相对有特色的一层，因为 UI 本身就是 Widget 组合。

---

### Widget Test 和 Integration Test 有什么区别？

**Widget Test：**

- 更轻量
- 跑得快
- 关注单个或局部组件行为

**Integration Test：**

- 更接近真实用户操作
- 会覆盖更完整的页面和流程
- 成本更高，执行更慢

简单看：

- 组件级验证：`Widget Test`
- 业务流程验证：`Integration Test`

---

### Flutter 中如何避免内存泄漏？

常见泄漏点有：

1. `AnimationController` 没有 `dispose`
2. `TextEditingController` 没有释放
3. `StreamSubscription` 没有取消
4. `Timer` 没有关闭
5. 闭包错误持有大对象或旧页面引用

比较标准的回答通常是：

- 资源谁创建，谁释放
- 所有控制器、订阅、监听器在 `dispose` 中回收
- 异步回调注意 `mounted`

---

### Flutter 中如何做代码质量保障？

一般可以从这几个层面回答：

1. 使用 `flutter analyze`
2. 建立 `lint` 规范
3. 编写单元测试、Widget 测试、集成测试
4. 做 Code Review
5. 关键页面做性能监控
6. 关键异常接入崩溃和日志平台

---

## 十二、补充题

### AppLifecycleState 是什么？有什么实际用途？

`AppLifecycleState` 用来描述应用生命周期状态，例如：

- 前台可交互
- 非活跃
- 后台
- 恢复

它的实际用途包括：

1. 页面切后台时暂停动画或视频
2. 页面恢复时刷新数据
3. 统计应用前后台切换
4. 处理音视频、定位、播放器等资源状态

顺手可以补上这一点：

> Flutter 组件生命周期解决的是 Widget 的创建和销毁，`AppLifecycleState` 解决的是整个应用前后台状态变化。

---

### compute 和 Isolate 有什么关系？

`compute` 可以看成 Flutter 对 `Isolate` 的一个常用封装。

它适合：

- 把单次的耗时计算放到后台执行
- 简化手动创建 `Isolate` 的复杂度

区别可以这样记：

- `Isolate` 更底层、更灵活
- `compute` 更适合简单的一次性后台任务

---

### Flutter 动画为什么常和 TickerProvider 一起出现？

因为不少动画控制器，比如 `AnimationController`，需要帧同步驱动。

`TickerProvider` 的作用就是：

- 为动画提供随屏幕刷新节奏推进的 ticker

常见的是：

- `SingleTickerProviderStateMixin`
- `TickerProviderStateMixin`

一般回答：

- 单个动画控制器常用 `SingleTickerProviderStateMixin`
- 多个动画控制器可以用 `TickerProviderStateMixin`

---

### Mixin 在 Flutter 中常见用法是什么？

Mixin 在 Flutter 中很常见，主要用于能力复用。

典型例子：

- `SingleTickerProviderStateMixin`
- `AutomaticKeepAliveClientMixin`
- `WidgetsBindingObserver`

它的价值在于：

- 不走传统多继承
- 可以把一类横切能力混入到组件中

---

### Flutter 中 InheritedWidget、Provider、Riverpod 的关系如何理解？

这道题适合用“抽象层次”去讲，而不是只讲哪个库更流行。

1. `InheritedWidget`
   - Flutter 原生提供的依赖共享机制
   - 更底层

2. `Provider`
   - 对 `InheritedWidget` 的工程化封装
   - 降低使用门槛

3. `Riverpod`
   - 在 Provider 思路上进一步强化依赖管理、测试性和解耦能力

这条链路重要的原因在于：它体现的是“原生能力 -> 工程封装 -> 更强抽象”的演进关系，而不是几个互不相关的工具。

---

## 十三、进阶高频题与新版本实战

### Impeller 渲染引擎跟 Skia 的本质区别是什么？

Impeller 是 Flutter 团队 22 年开始做的新渲染器，24 年起 iOS 默认启用，Android 也在分阶段切换。它要解决的核心问题是 Skia 时代被诟病已久的 **shader compilation jank**——首次播放某个动画/特效时卡一帧。

**Skia 时代为什么会有 shader 编译卡顿？**

Skia 是个通用 2D 引擎，它的设计假设是「shader 在用到时再编译」。Flutter 用 Skia 时，每个 RenderObject 的绘制路径（圆角、阴影、渐变、模糊）背后都是 shader 程序。第一次跑到某段 shader，Skia 要：

1. 从 SkSL 编译成目标平台 GLSL/Metal
2. GPU 驱动再把它编译成 GPU 可执行的字节码
3. 这一步可能耗费 100-500ms

结果就是「打开新页面、第一次播某个动画」时卡顿——因为 shader 在那一刻才编译。

Skia 的兜底方案是 `SkSL warmup`：先跑一遍把常用 shader 编译好缓存。但维护成本高（要手动收集 SkSL 列表），且不能覆盖所有场景。

**Impeller 的根本改变**：

- 所有 shader 在 **构建时**就预编译好（用 Flutter 的 `impellerc` 工具）
- 运行时只做「数据上传 + 绘制命令」，不再触发 shader 编译
- 直接用平台原生 API：iOS 用 Metal、Android 用 Vulkan / OpenGL ES

代价：

- 包体积略增（要带预编译 shader）
- 跟 Skia 时代相比，某些自定义 shader 的灵活性下降
- Android 端 Vulkan 路径在低端机上有兼容性问题（24 年还在持续修）

**实测收益**：iOS 上 Impeller 比 Skia 性能稳定不少——之前首次播放复杂动画掉到 20fps 的场景，Impeller 下能稳 60fps。

**如何开启 / 关闭**：

```yaml
# Flutter 3.10+ iOS 默认开启
# 强制关闭(老项目兼容):
# Info.plist
<key>FLTEnableImpeller</key>
<false/>

# Android 强制开启(Flutter 3.16+):
# AndroidManifest.xml
<meta-data
  android:name="io.flutter.embedding.android.EnableImpeller"
  android:value="true" />
```

几条归纳：

- Skia 是「运行时编译 shader」，首次播放有 jank
- Impeller 是「构建时预编译 shader」，运行时只跑命令
- iOS 已默认、Android 渐进切换
- 实测「首帧卡顿」类问题解决得彻底

### Dart 3 的 records 和 patterns 实战

Dart 3（23 年发布）的新特性主要是 records + patterns + sealed class，是真正改变写法的更新。

**Records（元组）**：

```dart
// 直接返回多个值,不用定义 class
(String, int) getUser() {
  return ('Alice', 30)
}

// 解构
final (name, age) = getUser()
print('$name $age')   // Alice 30

// 命名 record
({String name, int age}) getUserNamed() {
  return (name: 'Alice', age: 30)
}

final user = getUserNamed()
print(user.name)   // Alice
```

**Patterns（模式匹配）**：

switch 表达式现在可以「按结构匹配」：

```dart
Widget buildState(LoadState state) {
  return switch (state) {
    Loading() => const CircularProgressIndicator(),
    Success(data: var d) => Text(d),
    Error(message: var msg) => Text('Error: $msg'),
  }
}

// 配合 records 解构
String describe(Object obj) {
  return switch (obj) {
    (int x, int y) when x == y => 'equal pair',
    (int x, int y) => 'pair $x, $y',
    String s => 'string: $s',
    _ => 'unknown',
  }
}

// if-case 表达式
final position = (10, 20)
if (position case (int x, int y)) {
  print('x=$x y=$y')
}
```

**Sealed class（密封类）**：

```dart
sealed class LoadState<T> {
  const LoadState()
}

class Loading<T> extends LoadState<T> {
  const Loading()
}

class Success<T> extends LoadState<T> {
  const Success(this.data)
  final T data
}

class Error<T> extends LoadState<T> {
  const Error(this.message)
  final String message
}

// switch 时编译器强制穷尽,漏掉一个 case 编译报错
Widget build(LoadState<User> state) {
  return switch (state) {
    Loading() => const Spinner(),
    Success(:final data) => UserCard(user: data),
    Error(:final message) => ErrorView(message),
  }   // ✅ 编译通过(穷尽了所有 case)
}
```

**对状态管理的影响**：

Dart 3 之前，要表达「Loading / Success / Error」三态，常用 `freezed` 包生成代码：

```dart
@freezed
class LoadState<T> with _$LoadState<T> {
  const factory LoadState.loading() = Loading
  const factory LoadState.success(T data) = Success
  const factory LoadState.error(String message) = Error
}
```

`freezed` 需要 build_runner 跑代码生成。Dart 3 的 sealed class + records，**不少场景能直接替代 freezed**——不用代码生成、写法更简洁。但 freezed 仍有它的价值（自动生成 copyWith、equality、toJson），完全替代它不现实。

具体怎么用：

- 简单状态机用 sealed class + pattern matching
- 复杂模型（要 copyWith、要序列化）继续 freezed
- API 返回值多个值，优先 records 替代「定义临时类」

### Riverpod 2.x 的 code generation 实战

Riverpod 1.x 时代要手写 `StateNotifierProvider`、`FutureProvider`，类型签名经常拗口。Riverpod 2.x 主推 `@riverpod` 注解 + 代码生成，大幅简化。

**老写法（Riverpod 1.x）**：

```dart
// 手写各种 Provider 类型
final userProvider = FutureProvider.family<User, int>((ref, id) async {
  return ref.read(apiProvider).fetchUser(id)
})

final userNotifierProvider = StateNotifierProvider<UserNotifier, AsyncValue<User>>((ref) {
  return UserNotifier(ref)
})

class UserNotifier extends StateNotifier<AsyncValue<User>> {
  UserNotifier(this.ref) : super(const AsyncValue.loading()) {
    _load()
  }
  final Ref ref

  Future<void> _load() async {
    state = const AsyncValue.loading()
    try {
      final user = await ref.read(apiProvider).fetchUser(1)
      state = AsyncValue.data(user)
    } catch (e, st) {
      state = AsyncValue.error(e, st)
    }
  }
}
```

**新写法（2.x + code generation）**：

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart'

part 'user_provider.g.dart'

@riverpod
Future<User> user(UserRef ref, int id) async {
  return ref.watch(apiProvider).fetchUser(id)
}

// 带 mutation 的:
@riverpod
class UserNotifier extends _$UserNotifier {
  @override
  Future<User> build(int id) async {
    return ref.read(apiProvider).fetchUser(id)
  }

  Future<void> refresh() async {
    state = const AsyncValue.loading()
    state = await AsyncValue.guard(() => ref.read(apiProvider).fetchUser(state.value!.id))
  }

  Future<void> updateName(String name) async {
    final user = state.value!
    state = AsyncValue.data(user.copyWith(name: name))
    await ref.read(apiProvider).updateUser(user.id, name: name)
  }
}
```

`@riverpod` 注解 + `dart run build_runner build` 生成 `_$UserNotifier` 基类、`userNotifierProvider` 实例。

**消费 Provider（在 Widget 里）**：

```dart
class UserPage extends ConsumerWidget {
  final int userId
  const UserPage({required this.userId})

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider(userId))

    return userAsync.when(
      loading: () => const Spinner(),
      error: (e, st) => ErrorView(error: e),
      data: (user) => UserCard(user: user),
    )
  }
}
```

**自动 dispose**：

```dart
// 默认 keepAlive: false,Widget 销毁后 Provider 自动 dispose
// 想保活:
@Riverpod(keepAlive: true)
Future<List<Product>> products(ProductsRef ref) async { ... }
```

**Riverpod vs Bloc 怎么选**：

- 团队偏「函数式 + reactive」：Riverpod 自然
- 团队偏「事件驱动 + 严格分层」：Bloc 更结构化
- 团队又想要类型安全又怕样板代码：Riverpod + codegen

24 年新项目里 Riverpod 略占上风（社区活跃度高），但 Bloc 在企业级项目里仍有大量用户。

### Flutter 性能调优实战：用 DevTools 定位卡顿

Flutter 性能问题主要分三类：build 慢、layout/paint 慢、shader 卡顿。每类用不同工具定位。

**第一步：开 DevTools Performance 面板**

```bash
flutter run --profile        # profile 模式(关掉 debug 的 overhead)
# 然后打开 DevTools 的 Performance 面板
```

`--profile` 模式跟 release 接近但保留 profiler 信息。**千万不要用 debug 模式做性能分析**，结论会严重失真。

**第二步：识别问题类型**

DevTools 的 Performance overlay 显示每帧的 UI / Raster 耗时：

```
UI:    16.7ms   ← 蓝色,JS / Dart 逻辑耗时(build + layout)
Raster: 8.3ms   ← 绿色,GPU 光栅化耗时
```

任一超过 16.6ms 就掉帧（60fps 标准）。

- **UI 高 + Raster 正常** → build / layout 慢，看 Widget 树
- **UI 正常 + Raster 高** → 渲染复杂，看 paint / shader
- **第一帧 UI 飙高** → shader compilation jank（升 Impeller 解决）

**第三步：CPU Profiler**

DevTools 里 CPU Profile（Dart）能看到每个函数的耗时分布：

- 看 `BuildContext.build` 是否反复跑
- 看是否有 build 里做了昂贵计算（应该 `useMemoized` 或者前置）
- 看 setState 范围是否过大

**常见性能反模式**：

```dart
// ❌ build 里做昂贵计算
class List extends StatelessWidget {
  final List<Item> items

  @override
  Widget build(BuildContext context) {
    final filtered = items.where((i) => i.active).toList()   // 每次 build 都跑
    final sorted = filtered..sort((a, b) => a.date.compareTo(b.date))
    return ListView(children: sorted.map(...).toList())
  }
}

// ✅ 用 useMemoized 或上层传 props
final filtered = useMemoized(
  () => items.where((i) => i.active).toList()..sort(...),
  [items],
)
```

```dart
// ❌ 整个 List 用 setState 包,改一个 item 全部重建
class _State extends State<List> {
  List<Item> items = []

  void toggleItem(int id) {
    setState(() {
      items = items.map((i) => i.id == id ? i.copyWith(done: !i.done) : i).toList()
    })   // 整个 ListView 重建
  }
}

// ✅ 每个 Item 自己 StatefulWidget,只重建自己
class ItemTile extends StatefulWidget {
  final Item item
  ...
}
```

**长列表用 ListView.builder**：

```dart
// ❌ 一次性 build 1000 个 Widget
ListView(
  children: items.map((i) => ItemTile(item: i)).toList(),
)

// ✅ 只 build 屏幕里的 + 缓存几个
ListView.builder(
  itemCount: items.length,
  itemBuilder: (ctx, i) => ItemTile(item: items[i]),
)
```

**RepaintBoundary 隔离重绘**：

```dart
// 复杂动画区域包 RepaintBoundary,不让外面变化触发它重绘
RepaintBoundary(
  child: ComplexAnimatedWidget(),
)
```

但不要乱用——每个 RepaintBoundary 都开一层独立光栅化，过多反而拖性能。

### Flutter Web 现在能用了吗？

Flutter Web（24 年）已经过了「能跑」阶段，到了「可选」阶段。但选不选要看场景。

**Flutter Web 的两种渲染模式**：

- **HTML renderer**：编译为 HTML + CSS + Canvas，体积小、SEO 友好、加载快
- **CanvasKit renderer**：Skia/Impeller 编译为 WebAssembly，画到 Canvas 上，跟移动端表现一致但首屏要下载 ~2MB WASM

24 年 Flutter Web 默认会**自动选择**：移动端用 HTML、桌面端用 CanvasKit。

**适合 Flutter Web 的场景**：

- **想跟 Flutter App 共享代码的内部工具**：管理后台、内部 dashboard、跟移动端 99% 复用业务代码
- **桌面 Web 应用**（Figma 类、画图工具）：CanvasKit 性能足够、UI 强一致
- **PWA**：跟原生 App 体验相近的渐进 Web 应用

**不适合的场景**：

- **SEO 要求高**：Flutter Web 的 HTML renderer 也只是「文本能爬」，搜索引擎对 Canvas 内容理解差
- **首屏快、轻量**：Flutter Web 即便 HTML 模式，初始包也有 1MB+，比 React 起码大 3 倍
- **内容型网站**（博客、新闻、电商详情页）：传统 Web 技术更合适
- **依赖大量 Web 生态库**（rich text editor、复杂图表）：Flutter Web 还要找替代品

**实际表现**：

- 启动：CanvasKit 模式 3-5 秒（要下 WASM），HTML 模式 1-2 秒
- 体积：HTML 模式约 1.5MB（gzip），CanvasKit 模式约 3MB
- 性能：CanvasKit 下复杂动画跟移动端持平；HTML 模式偶有渲染差异

所以 Flutter Web 适合「Flutter 已有 App + 想共享代码做内部工具或 dashboard」的场景，**不适合做公网产品的 SEO 落地页**。

### Flutter 与原生集成进阶：Platform View 性能成本

不少场景要在 Flutter 里嵌入原生 UI——地图、WebView、相机预览、原生视频播放器。`PlatformView` 是 Flutter 提供的方案。

**两种模式**：

**1. Hybrid Composition**（默认，性能成本高）：

- Android：原生 View 渲染到一个独立的 Surface，Flutter 通过 SurfaceTexture 拿到画面合成
- iOS：UIKit View 直接叠加在 Flutter 渲染层之上
- 优势：完全原生体验、原生 view 的特性（手势、滚动）都正常
- 代价：多一层合成、性能比纯 Flutter 低 30-50%

**2. Virtual Display**（老方案，已废弃）：

- 原生 View 渲染到一个虚拟 display，整个截图传给 Flutter
- 性能更好但**手势事件没法正常传递**——原生 view 收不到 touch 事件
- 24 年起几乎不用

具体怎么取舍：

- 偶尔嵌入一两个原生 view（一个地图、一个 WebView）：性能成本可接受
- 大量复用原生 view、构成主界面：考虑反过来——用原生主导 + 嵌入 Flutter（`FlutterFragment` / `FlutterViewController`）

**性能优化**：

- 减少 PlatformView 的尺寸 / 数量
- PlatformView 区域避免叠 Flutter 动画
- 用 `Texture` widget 替代（针对纯视频流场景）：原生写到 OES texture，Flutter 直接复用，性能接近原生纹理渲染

**嵌入原生主导的项目**：

老 App 想渐进引入 Flutter，不是「整个 App 用 Flutter」：

```kotlin
// Android: 在已有 Activity 里嵌一个 FlutterFragment
class MyActivity : FragmentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val flutterFragment = FlutterFragment.withNewEngine()
      .initialRoute("/profile")
      .build<FlutterFragment>()

    supportFragmentManager.beginTransaction()
      .replace(R.id.flutter_container, flutterFragment)
      .commit()
  }
}
```

```swift
// iOS: 在已有 ViewController 里 push 一个 FlutterViewController
let flutterEngine = (UIApplication.shared.delegate as! AppDelegate).flutterEngine
let flutterVC = FlutterViewController(engine: flutterEngine, nibName: nil, bundle: nil)
navigationController?.pushViewController(flutterVC, animated: true)
```

需要 Flutter Engine 全局复用（`FlutterEngineGroup` 管理多 engine）才能保证启动速度，否则每次 push 都要起 2 秒。

### Flutter 大型项目的工程化

中大型 Flutter 项目要解决「目录结构、路由、依赖注入、多 flavor、CI」这些问题。

**目录结构（feature-based）**：

```
lib/
├── core/                  # 跨 feature 共享
│   ├── network/
│   ├── storage/
│   ├── utils/
│   └── theme/
├── features/
│   ├── auth/
│   │   ├── data/          # API、repository
│   │   ├── domain/        # entity、business logic
│   │   ├── presentation/  # widget、provider
│   │   └── auth.dart      # 模块导出
│   └── product/
│       ├── data/
│       ├── presentation/
│       └── product.dart
├── shared/                # 跨 feature widget 库
│   ├── widgets/
│   └── components/
└── main.dart
```

每个 feature 一个独立目录，对外暴露 `feature.dart` 作为 barrel。

**路由：go_router**：

```dart
final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (ctx, state) => HomePage(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (ctx, state) => ProductPage(
        id: state.pathParameters['id']!,
      ),
    ),
    ShellRoute(
      builder: (ctx, state, child) => MainShell(child: child),
      routes: [
        GoRoute(path: '/home', builder: (...) => ...),
        GoRoute(path: '/profile', builder: (...) => ...),
      ],
    ),
  ],
  redirect: (ctx, state) async {
    final isLoggedIn = ref.read(authProvider).isLoggedIn
    if (!isLoggedIn && state.matchedLocation != '/login') {
      return '/login'
    }
    return null
  },
)
```

替代手写 Navigator，支持嵌套、深链、路由守卫。

**依赖注入：GetIt / Riverpod**：

- 简单项目：GetIt（service locator 模式）
- 中大项目：Riverpod（也能当 DI 用，并且天然 reactive）

```dart
// GetIt
final getIt = GetIt.instance

void setupDI() {
  getIt.registerLazySingleton<ApiClient>(() => ApiClient())
  getIt.registerLazySingleton<UserRepository>(() => UserRepository(getIt()))
}

// Riverpod 当 DI
final apiClientProvider = Provider((ref) => ApiClient())
final userRepositoryProvider = Provider((ref) => UserRepository(ref.read(apiClientProvider)))
```

**多 flavor 配置**：

```bash
# Flutter 内置 flavor 支持
flutter run --flavor dev --dart-define=API_URL=https://dev-api.example.com
flutter run --flavor prod --dart-define=API_URL=https://api.example.com
```

```dart
class Config {
  static const apiUrl = String.fromEnvironment('API_URL', defaultValue: '')
}
```

iOS 用 Xcode scheme + Build Configuration、Android 用 productFlavors + sourceSets。

**CI/CD（GitHub Actions 示例）**：

```yaml
jobs:
  build-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with: { flutter-version: '3.24.x' }
      - run: flutter pub get
      - run: flutter test
      - run: flutter build ios --release --no-codesign

  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
      - run: flutter pub get
      - run: flutter build apk --release
```

**代码生成的管理**：

Riverpod、freezed、json_serializable 都要 `build_runner`。每次改完跑：

```bash
dart run build_runner build --delete-conflicting-outputs
# 或者 watch 模式
dart run build_runner watch --delete-conflicting-outputs
```

把这一步加到 git pre-commit hook 避免漏跑导致编译失败。

### Flutter vs Kotlin Multiplatform 现在怎么选？

24-25 年 Kotlin Multiplatform（KMP）越来越被提到——Google 跟 JetBrains 一起推，定位是「跨端但保留各端原生 UI」。这跟 Flutter 的「跨端用同一套 UI」路子完全不同，团队选型容易问到。

**Flutter 的路线**：

- 一套 Dart 代码 → 一套 UI（用自家渲染引擎画）
- 跨端体验一致性更强——iOS 和 Android 视觉表现高度一致
- 工具链统一：开发、构建、调试都用 Flutter SDK

**KMP 的路线**：

- 共享业务逻辑（Kotlin 写的 model、API、数据库等）
- UI 各端自己写——iOS 用 SwiftUI、Android 用 Compose
- 跨端体验是各端原生的

对比起来：

**Flutter 的优势**：

- 跨端 UI 一致，设计师只画一套图
- 开发效率高——一套代码全端跑
- 工具链一站式（DevTools、Flutter run）

**KMP 的优势**：

- UI 完全原生，每端体验是原生味儿
- 团队可以 iOS / Android 分开做 UI、共享业务层
- 跟原生生态深度集成（不用 Platform Channel 这种桥接）
- Compose Multiplatform（KMP 的 UI 部分）也支持桌面 / Web

**适合 Flutter 的场景**：

- 设计要求跨端完全一致（电商 App、品牌应用）
- 团队前端背景为主，Dart 学起来快
- 想最快速度出多端 MVP

**适合 KMP 的场景**：

- 大型 App 已有 iOS / Android 原生代码，想逐步共享业务逻辑
- 团队 iOS 和 Android 工程师都很强，想保留各自的 UI 工艺
- 对原生体验有强诉求（系统级集成、特定平台特性）

看现在大公司的实际做法：

24-25 年的趋势：

- 创业公司、小团队：仍以 Flutter 为主（一套代码、上手快）
- 大厂：KMP 在崛起。Netflix、Philips、McDonald's 都公开用 KMP 共享业务层
- 国内大公司：Flutter 仍是主流，KMP 国内生态还在追

两者不是完全互斥——可以用 Flutter 做主 App、KMP 共享部分核心模块（加密、复杂业务计算）。但通常一个团队选一条主路。

### Flutter 应用的崩溃监控怎么接？

Flutter 的崩溃监控也分两层——Dart 层崩溃和 Native 层崩溃。

**Dart 层异常**：

```dart
void main() {
  // 同步异常
  FlutterError.onError = (details) {
    FlutterError.presentError(details)
    Sentry.captureException(details.exception, stackTrace: details.stack)
  }

  // 异步 / Isolate 异常
  PlatformDispatcher.instance.onError = (error, stack) {
    Sentry.captureException(error, stackTrace: stack)
    return true
  }

  // 启动应用要用 runZonedGuarded 包(捕获 zone 内未处理异常)
  runZonedGuarded(() {
    runApp(MyApp())
  }, (error, stack) {
    Sentry.captureException(error, stackTrace: stack)
  })
}
```

`FlutterError.onError` 捕获 Widget tree 渲染过程的错误，`PlatformDispatcher.instance.onError` 捕获其他 Dart 异步错误。runZonedGuarded 是兜底。

**Native 层崩溃**：

Flutter 的 Dart 代码异常归 Dart 层处理，但底层 Engine（C++）崩溃、原生 SDK 崩溃、内存崩溃归 Native 层。要单独接 iOS / Android crash reporter：

```dart
// 使用 sentry_flutter 一站式
import 'package:sentry_flutter/sentry_flutter.dart'

Future<void> main() async {
  await SentryFlutter.init(
    (options) {
      options.dsn = 'https://xxx@sentry.io/xxx'
      options.tracesSampleRate = 0.1
    },
    appRunner: () => runApp(MyApp()),
  )
}
```

`sentry_flutter` 内部会自动安装 iOS 和 Android 的 native crash reporter，Dart + Native 一站式覆盖。

**Obfuscation / Symbolication**：

release build 出来的 Dart 代码默认会被 obfuscate（混淆），crash 堆栈里全是 `a.b.c` 看不出业务。要带 `--obfuscate --split-debug-info` 编译：

```bash
flutter build apk --release \
  --obfuscate \
  --split-debug-info=./build/symbols
```

`./build/symbols` 目录里是 debug info 文件，要上传到监控平台用来还原堆栈：

```bash
sentry-cli upload-dif ./build/symbols
```

**崩溃监控建议**：

- **dev 不上报**：用 `kReleaseMode` 判断，dev 模式只 print
- **捕获异常后展示降级 UI**：用 `ErrorWidget.builder` 替换红屏

```dart
ErrorWidget.builder = (details) {
  return Material(
    child: Center(
      child: Text(kDebugMode ? details.toString() : '页面出错了'),
    ),
  )
}
```

- **关联用户**：上报时带 userId、release 版本、设备信息，定位回溯方便
- **采样**：trace 高频，全量上报会爆，加 sampleRate 控制

**24-25 年的工具栈**：

- **Sentry Flutter**：一站式，社区主流
- **Firebase Crashlytics**：Google 出的，跟 Firebase 生态绑定
- **bugsnag / instabug**：商业方案，专注移动端崩溃

国内场景还有 Bugly 等本地化方案，但 Flutter 的支持没有 Sentry / Crashlytics 这种海外工具完善。

### Flutter 编译产物里到底有什么？AOT vs JIT 怎么选？

不少人写了几年 Flutter 也没看过产物里有啥。理解了能解释 release 包大、调试和发布行为差异的根源。

**两种编译模式**：

- **JIT（Just In Time）**：debug 模式用，dart 代码在运行时编译为机器码
- **AOT（Ahead Of Time）**：release 模式用，dart 代码在 build 时已经编译为机器码

JIT 慢但能 hot reload（dart VM 支持运行时替换代码）。AOT 快但不能 hot reload。

**Android release 产物**：

```bash
flutter build apk --release
# 产物在 build/app/outputs/flutter-apk/app-release.apk
```

解压 apk 后能看到：

```
app-release.apk
├── lib/
│   ├── arm64-v8a/
│   │   ├── libflutter.so        # Flutter Engine(C++,约 5MB)
│   │   └── libapp.so            # AOT 编译后的 Dart 代码(业务大小,几 MB)
│   └── armeabi-v7a/             # 老 ARM 架构(老 Android 设备)
├── assets/
│   └── flutter_assets/          # Flutter 资源(图片、字体)
└── classes.dex                  # Android 原生代码
```

`libflutter.so` 是 Flutter Engine（包含 Skia / Impeller + Dart VM + 基础 widget），固定大小约 5MB。`libapp.so` 是你的业务代码编译后的机器码，大小看代码量。

**iOS release 产物**：

```
Runner.app/
├── Frameworks/
│   ├── Flutter.framework        # Flutter Engine
│   └── App.framework            # AOT 编译后的业务代码
├── flutter_assets/
└── ...
```

跟 Android 类似，分 Engine 和 App 两块。

**包大小为什么这么大**：

Flutter 项目默认 release 大小：

- Android APK：约 15-20MB（一个空白项目）
- iOS IPA：约 20-30MB
- Web：约 1.5-3MB（视 renderer）

主要来源：

- Flutter Engine（5MB+）
- Material / Cupertino widget 库
- Skia / Impeller 渲染引擎
- Dart VM 跑时

业务代码只占其中很小一部分。

**怎么瘦身**：

- **Android 用 App Bundle 而不是 APK**：上传 .aab 到 Google Play，Play 会自动按设备分发对应架构包（不带 armeabi-v7a 给只有 arm64 的设备）。能减 30-40%
- **拆分 ABI**：`flutter build apk --split-per-abi` 生成多个 APK
- **Tree-shake icons**：默认开（release 模式）
- **不用的语言包砍掉**：`flutter build --no-tree-shake-icons` 关掉某些不需要的优化

国内场景没 Play Store，自建分发要按设备类型分包，最小化用户下载量。

**调试相关**：

- **DevTools 看到的 isolate**：Dart VM 里实际跑代码的 isolate，有 main、IO、background 等
- **Profile 模式**：介于 debug 和 release 之间——AOT 编译（性能接近 release）但保留 profiler 信息

```bash
flutter run --profile
```

性能优化必须用 profile 模式跑，不能用 debug。

### Flutter 跟原生通信的 Platform Channel 性能瓶颈

不少人用 Platform Channel 用着用着觉得「调用慢」。理解它的传输模型才能避问题。

**Platform Channel 的本质**：

JS 给原生发消息走的也是消息队列模型——Dart 这边 invokeMethod，参数被序列化为 binary message（MethodChannel 用 StandardMessageCodec），通过 BinaryMessenger 发给原生，原生处理完结果再序列化回来。

```dart
const platform = MethodChannel('com.example.battery')
final int level = await platform.invokeMethod('getBatteryLevel')
```

每次 invoke 都是：

1. Dart → 序列化参数（约几十微秒）
2. 跨语言通信（消息队列入队 + 出队，约几十微秒）
3. 原生处理（业务时间）
4. 结果序列化（同 1）
5. 回到 Dart（同 2）

**瓶颈在哪**：

- **序列化开销**：传大对象（图片字节、大数组）每次都要 copy + 序列化，几 MB 数据能慢几百毫秒
- **频繁通信**：每秒上千次 invoke，单 channel 排队等
- **类型限制**：StandardMessageCodec 只支持几种基础类型（数字、字符串、List、Map、Uint8List 等），自定义类要 JSON-ize

**优化方案**：

**1. BasicMessageChannel 替代 MethodChannel**：

更轻量、可双向流式：

```dart
const channel = BasicMessageChannel<String>('chan', StringCodec())
channel.setMessageHandler((message) async {
  return 'reply: $message'
})
```

**2. FFI（Foreign Function Interface）替代 Platform Channel**：

FFI 是 Dart 直接调 C/C++ 函数，绕过消息队列：

```dart
import 'dart:ffi'

final lib = DynamicLibrary.open('mylib.so')
final addFunc = lib.lookupFunction<Int32 Function(Int32, Int32), int Function(int, int)>('add')
print(addFunc(1, 2))   // 直接同步调用,零序列化
```

适合：高频调用、复杂数据传输、性能敏感场景。但只能调 C/C++，调 Java/Swift 还得绕一层。

**3. Pigeon 生成类型安全的 channel 代码**：

`pigeon` 是 Flutter 官方工具，根据 Dart interface 自动生成各端 channel 代码：

```dart
// pigeons/api.dart
@HostApi()
abstract class BatteryApi {
  int getLevel()
  void setMode(int mode)
}
```

跑 pigeon，自动生成 Dart 端 + iOS Swift 端 + Android Kotlin 端的对接代码，类型对齐、不用手写序列化。

**4. 大数据用 Texture 或 PlatformView**：

视频流、相机预览这种持续传大数据的，别走 Channel。Texture 是 GPU 共享内存，原生写、Flutter 读，零拷贝。

项目里：

Platform Channel 适合「偶尔调一下原生 API」（震动、获取设备 ID、调系统弹窗）。如果发现某个功能在大量调 channel，考虑：

- 业务逻辑能不能搬到 Dart（FFI 调 native lib）
- 是不是真的需要每次都通信（缓存原生返回值）
- 用 Texture / FFI 替代

24-25 年的新项目建议一开始就用 Pigeon 写 channel 接口，少手写。类型安全和后期维护差异明显。

### Flutter 包大小优化全套方案

Flutter App 出包默认就比原生大不少（带了 Flutter Engine ~5MB），如果业务还重，最终包容易 30MB+。包大用户下载意愿低、流量贵地区流失明显。优化思路：

**第一步：开 App Bundle（Android）**

Google Play 推荐用 .aab 而不是 .apk。Play 收到 .aab 后会按用户设备特征拆分——只下载对应 CPU 架构（arm64 / armv7）+ 屏幕密度的资源：

```bash
flutter build appbundle --release
# 上传到 Play Store
# 用户实际下载的包通常比 APK 小 30-40%
```

**第二步：拆分 ABI（国内场景，没 Play）**

国内分发不能用 .aab，但可以分别 build 不同架构的 .apk：

```bash
flutter build apk --split-per-abi --release

# 产物:
# app-armeabi-v7a-release.apk   (老机器)
# app-arm64-v8a-release.apk     (现代机器,大部分用户)
# app-x86_64-release.apk        (模拟器)
```

分发渠道根据用户设备下不同的 APK，省 40-50% 体积。

**第三步：tree-shake icons**

`flutter build` 默认开启 icon tree shaking——只把代码里实际用到的 Material Icons 打进包。不要手动关：

```bash
# ❌ 不要这么干
flutter build apk --no-tree-shake-icons

# 一个常见错误:用了 Map<String, IconData> 这种动态查图标的写法,tree shaker 看不出来,会被禁用
```

**第四步：图片资源治理**

- 用 WebP 替代 PNG（体积减 30-50%）
- 用合适的分辨率（不要把 4K 大图当 80px 头像用）
- 移除未引用的 asset（dart 工具能扫）

```yaml
# pubspec.yaml 只列实际需要的
flutter:
  assets:
    - assets/images/   # 整个目录会被打包
    # 比这个更可控:
    - assets/images/logo.png
    - assets/images/banner.webp
```

**第五步：deferred components（Android 动态 feature）**

24 年起 Flutter 支持 deferred components（懒加载模块）。某些不常用的 feature 可以拆出去，用户首次进入时才下载：

```dart
// 标记为 deferred
import 'package:my_app/features/admin.dart' deferred as admin;

// 使用时加载
Future<void> openAdmin() async {
  await admin.loadLibrary()
  Navigator.push(context, MaterialPageRoute(
    builder: (_) => admin.AdminPage(),
  ))
}
```

适合：管理后台模块、AR / 高级编辑器这种大 feature 但不是所有用户都用的。

**第六步：依赖治理**

不是 Flutter 特有但很关键——检查 `pubspec.lock`，看哪些大包能换或者删：

```bash
# 检查每个 pub 包占的体积
flutter pub deps
```

常见胖子：

- `font_awesome_flutter`：图标库，全引入 5MB+，按需用 `flutter_svg` + 单独 SVG 资源更省
- `cached_network_image`：还好，几百 KB
- 视频 / 音频处理库：动辄几 MB，能用 native 就用 native

**第七步：obfuscate + split-debug-info**

```bash
flutter build apk --release \
  --obfuscate \
  --split-debug-info=./build/symbols
```

混淆能让 dart 类名 / 函数名变短，减少 5-10% 业务代码体积。同时把调试信息拆出去（不打进包），出包再减一点。

**项目典型成果**：

- 空白 Flutter 项目 release APK：~15MB
- 普通业务（含图标、几个页面、网络请求）：~20-25MB
- 优化后（App Bundle + obfuscate）：~12-15MB

体积优化做到 15MB 以下基本就到 Flutter 的天花板了。再小就要考虑「业务能不能拆 deferred component」「是不是该砍掉 Flutter Engine 自带的某些 widget」。

### Flutter 大型项目用 Melos 做模块化

中大型 Flutter 项目（多人协作、复杂业务）单一 `lib/` 目录很难长期维护——所有人改同一个仓库、循环依赖增多、build 越来越慢。Melos 是 Flutter 社区的 Monorepo 管理工具，类似 JS 生态的 Turborepo。

**Melos 项目结构**：

```
my_app/
├── melos.yaml
├── pubspec.yaml      # workspace 根配置
├── apps/
│   └── main_app/     # 主 App 工程
└── packages/
    ├── core/         # 跨包共享
    ├── auth/
    ├── product/
    ├── cart/
    └── ui_kit/       # UI 组件库
```

**melos.yaml 配置**：

```yaml
name: my_app

packages:
  - apps/**
  - packages/**

scripts:
  build:
    run: melos exec -- "flutter build apk"
    packageFilters:
      scope: "main_app"

  test:
    run: melos exec -- "flutter test"
    packageFilters:
      dirExists: test

  analyze:
    run: melos exec -- "flutter analyze"
```

```bash
# 安装 melos
dart pub global activate melos

# bootstrap:把 packages 之间的依赖 link 起来
melos bootstrap

# 一条命令在所有包跑 build_runner
melos exec -- "dart run build_runner build --delete-conflicting-outputs"

# 只对 affected packages 跑测试
melos run test --since=origin/main
```

**模块化的关键约束**：

1. **`packages/core` 不能反向依赖业务模块**：core 是叶子包，谁都能依赖它，它不依赖谁
2. **业务模块之间通过 interface 解耦**：`cart` 模块要用 `auth` 的能力，不要直接 `import 'package:auth/'`，而是定义接口，由根 App 注入实现
3. **UI 组件统一在 `ui_kit`**：业务模块不能各自造按钮、输入框

```dart
// packages/auth/lib/auth.dart
abstract class AuthService {
  Future<User?> getCurrentUser()
  Future<void> logout()
}

// packages/cart/lib/cart.dart
class CartService {
  final AuthService _auth   // 接口依赖,不依赖具体实现

  CartService(this._auth)
}

// apps/main_app/lib/main.dart
final auth = AuthServiceImpl()
final cart = CartService(auth)   // 主 App 注入
```

**Melos 的实战收益**：

- **构建时间**：affected-only 跑只测改动模块，大型项目 CI 从 20 分钟 → 5 分钟
- **代码所有权清晰**：每个 package 对应一个团队 / 一组人，PR 改谁的代码一目了然
- **复用方便**：UI Kit / Core 抽出来后，新 App 直接 import 复用，不用重写

**版本管理**：

Melos 集成了 `changesets` 风格的版本管理：

```bash
melos version --no-private
# 自动:
# - 看每个 package 的 commit 历史
# - 算新版本号
# - 更新 CHANGELOG
# - 打 git tag
```

适合「内部 Flutter 包发到私有 pub repo」的场景。

**新项目要不要上 Melos**？

- 5 人以下、单 App 项目：通常不需要，单 `lib/` 配合 feature-based 目录即可
- 5-20 人、多人并行开发：开始考虑，按业务拆 3-5 个 package
- 20+ 人、多个 App 共享组件：必须用，否则代码冲突 / 编译时间都顶不住

阿里、字节、Google 内部的大 Flutter 项目都是 Melos 路子。中等规模团队建议从一开始就按 Melos 结构搭，后期重构成本巨大。

### Flutter vs Compose Multiplatform 怎么选？

Compose Multiplatform（CMP）是 JetBrains 在 KMP 上推的 UI 框架，24 年起逐渐成熟，正面对 Flutter。

**两者的根本路线**：

- **Flutter**：自绘渲染（Skia/Impeller），写一次 Dart，UI 各端完全一致
- **CMP**：UI 用 Compose（一种声明式 UI 语法），编译到各端原生组件（Android 走 View 系统、iOS 走 SwiftUI 桥接、桌面走 Swing）

**对比表**：

| 维度 | Flutter | Compose Multiplatform |
|------|---------|----------------------|
| 语言 | Dart | Kotlin |
| 跨端覆盖 | iOS + Android + Web + 桌面 + 嵌入式 | Android + iOS + 桌面 + Web（Beta） |
| 渲染 | 自绘 | iOS 桥到 SwiftUI / Android 桥到 View |
| 性能 | 接近原生 | 接近原生（iOS 略低于 SwiftUI） |
| 生态 | pub.dev 几年积累 | 起步阶段，Kotlin/Java 生态可复用 |
| 团队背景 | 前端 / 跨端 | Android / 后端 Kotlin |
| 团队投入 | Google | JetBrains |

**Flutter 仍占优的场景**：

- **UI 设计要求各端完全一致**：自绘渲染天然做得到，CMP iOS 桥到 SwiftUI 多少有差异
- **前端 / Web 团队为主**：Dart 跟 JS 接近，迁移成本低
- **生态需求高**：Flutter pub.dev 库丰富、教程多

**CMP 占优的场景**：

- **Android 主导团队**：Compose 已经是 Android UI 的现代选择，CMP 等于「现有 Android 经验扩展到 iOS / 桌面」
- **要跟原生 SDK 深度集成**：CMP 跟 Kotlin / Swift 互操作天然
- **现有 KMP 代码基础**：业务逻辑已经用 KMP 共享了，UI 自然选 CMP
- **桌面应用**：IntelliJ IDEA / Android Studio 自己就是 CMP 写的，桌面成熟度高

**24-25 年生态情况**：

- Flutter：成熟，全球大量生产应用
- CMP：iOS 24 年稳定，桌面已稳定 2+ 年，Web 还在 Beta

看现在大公司的选型：

- 创业公司 / 中小团队 / 跨端体验一致：Flutter 仍是第一选择
- 大公司 / 已有 KMP 投资 / Android 重度团队：CMP 是趋势
- 国内：Flutter 仍占绝对多数，CMP 几乎没人用（语言、生态、文档都是门槛）

两者不是非此即彼。一些团队的做法：业务逻辑用 KMP 共享、UI 在 iOS 写 SwiftUI、Android 写 Compose、Web 用 Flutter Web。把 Flutter 当「Web / 桌面端的 UI 框架」、把 Compose 当「移动端 Android UI 框架」，KMP 把业务逻辑统一——这是 24-25 年某些大型项目的新尝试。

### Dart 3.5+ 新特性带来了什么实际改变

Dart 一直在演进，3.5（24 年）/ 3.6 / 3.7 几个版本加的东西落到业务里有实际价值。

**Dart 3.5：digit separators**

```dart
// 旧
const oneMillion = 1000000
const bytes = 1073741824

// 新:数字里可以加下划线分隔
const oneMillion = 1_000_000
const bytes = 1_073_741_824   // 1GB,清晰
const phoneNumber = 0xFF_FF_FF
```

不影响行为，纯粹可读性。处理金额、时间戳、字节数时特别有用。

**Dart 3.5：`base` / `interface` / `final` 修饰符稳定**

类的可见性 / 扩展性控制更精细：

```dart
// base 类:只能被同库继承,跨库只能 implement
base class Animal {}

// interface 类:只能被 implement,不能 extend
interface class Logger {}

// final 类:既不能 extend 也不能 implement
final class User {}

// sealed:封闭继承,所有子类必须在同一文件,switch 自动穷尽
sealed class Result {}
```

库作者用这套修饰符精确控制 API 边界，使用方不能随意继承或实现——这是大型项目里特别需要的。

**Dart 3.7：wildcard variables**

```dart
// 旧:循环里没用的变量也困难名
for (final ignored in list) { /* 不关心 ignored */ }

// 新:用 _ 表示「不关心」
for (final _ in list) { /* 跑 N 次 */ }

// 解构里也能用
final (_, age) = getUser()   // 不要 name
```

C# / Rust 早就有的语法糖,Dart 终于补上。

**Dart 3.6+:扩展类型(Extension Types)**

类似 TS 的 branded types,零运行时开销:

```dart
extension type UserId(int value) {}
extension type OrderId(int value) {}

void getUser(UserId id) { /* ... */ }

final orderId = OrderId(123)
getUser(orderId)   // ❌ 编译报错:OrderId 不是 UserId
```

编译后实际就是 int(零开销),但类型系统区分两者。处理「都是 int / String 但语义不同」的 ID 场景超有用。

**实战影响**:

- 老项目升 Dart 3.x 时,新的严格规则会暴露出很多历史问题(空安全的严格、sealed 强制穷尽 switch)
- `pubspec.yaml` 的 `environment.sdk: ^3.5.0` 是新项目最低基线
- 配合 `lints` 包的 `recommended` / `strict` 规则集开启,代码质量自动起来

### Flutter Web 的输入交互 + 多平台键鼠

Flutter Web 24 年起越来越成熟,「桌面应用 + Web 应用」场景越来越多。也就是说原本「触摸优先」的 Flutter,要处理键盘、鼠标、触控板这些桌面级输入。

**鼠标 hover 状态**:

```dart
MouseRegion(
  onEnter: (_) => setState(() => isHovered = true),
  onExit: (_) => setState(() => isHovered = false),
  cursor: SystemMouseCursors.click,   // 鼠标光标变手型
  child: Container(
    color: isHovered ? Colors.blue : Colors.grey,
    child: Text('Hover me'),
  ),
)
```

`InkWell` / `IconButton` 等 Material 组件已经内置 hover 效果,自定义 widget 要自己加。

**键盘事件**:

```dart
KeyboardListener(
  focusNode: focusNode,
  onKeyEvent: (event) {
    if (event is KeyDownEvent && event.logicalKey == LogicalKeyboardKey.escape) {
      Navigator.pop(context)
    }
  },
  child: ...,
)
```

更高级的:`Shortcuts` + `Actions` 做应用级快捷键(Ctrl+S 保存、Cmd+K 搜索)。

**滚动 + 触控板**:

桌面用户用滚轮 / 触控板滚动,跟手机触摸滚动行为完全不同——精确度、惯性、方向都不一样。`ScrollConfiguration` 控制不同平台行为:

```dart
ScrollConfiguration(
  behavior: ScrollConfiguration.of(context).copyWith(
    dragDevices: {
      PointerDeviceKind.touch,
      PointerDeviceKind.mouse,
      PointerDeviceKind.trackpad,   // 触控板
    },
  ),
  child: ListView(...),
)
```

**右键菜单**(桌面 / Web 必备):

```dart
GestureDetector(
  onSecondaryTapDown: (details) {
    showMenu(
      context: context,
      position: RelativeRect.fromLTRB(
        details.globalPosition.dx, details.globalPosition.dy, 0, 0,
      ),
      items: [
        PopupMenuItem(child: Text('复制')),
        PopupMenuItem(child: Text('粘贴')),
      ],
    )
  },
  child: ...,
)
```

**文本选择 + 复制粘贴**:

Web 上用户期望长按文本能选 + 复制。Flutter 默认行为是「点击聚焦」,要 SelectionArea 包起来:

```dart
SelectionArea(
  child: Column(
    children: [
      Text('用户名: Alice'),
      Text('邮箱: alice@example.com'),
    ],
  ),
)
```

包了之后用户能像 Web 一样跨多行选文本 + 复制。

项目里:

Flutter Web 在「桌面级输入」上做得还可以,但跟原生 Web 体验仍有差距:

- **SEO**:HTML renderer 模式爬虫能爬到文本,但页面 DOM 结构跟普通 Web 完全不一样(Flutter 自己画的 DIV 套 DIV)
- **可访问性**:有 `Semantics` widget 帮屏幕阅读器读,但效果比原生 ARIA 差
- **跟浏览器原生功能集成**:浏览器搜索(Ctrl+F)、剪贴板自动识别等功能在 Flutter Web 里几乎不存在

所以 Flutter Web 的甜区还是「内部工具 / dashboard / 桌面级应用」,不是「公网内容站」。

### Flutter 状态恢复:从后台进程被终止到完整恢复

iOS / Android 系统在内存紧张时会终止后台 App 进程。用户切回来时如果 App 整个重置,会明显影响体验——刚填一半的表单、看到一半的视频,都需要重新开始。

Flutter 的 RestorationScope / RestorableProperty 是解决这个的。

```dart
class TodoEditorPage extends StatefulWidget {
  @override
  State<TodoEditorPage> createState() => _TodoEditorPageState()
}

class _TodoEditorPageState extends State<TodoEditorPage> with RestorationMixin {
  final _title = RestorableString('')
  final _isImportant = RestorableBool(false)

  @override
  String get restorationId => 'todo_editor'

  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    registerForRestoration(_title, 'title')
    registerForRestoration(_isImportant, 'isImportant')
  }

  @override
  void dispose() {
    _title.dispose()
    _isImportant.dispose()
    super.dispose()
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          onChanged: (v) => _title.value = v,
          controller: TextEditingController(text: _title.value),
        ),
        SwitchListTile(
          value: _isImportant.value,
          onChanged: (v) => setState(() => _isImportant.value = v),
        ),
      ],
    )
  }
}
```

App 启动时加 `restorationScopeId`:

```dart
MaterialApp(
  restorationScopeId: 'app',   // 启用整个 app 的状态恢复
  // ...
)
```

效果:

- 用户打开 todo 编辑页,输入了「买菜」,点 home 切到别的 App
- 系统在后台终止了 Flutter 进程
- 用户切回来,Flutter App 重新启动
- 路由自动恢复到 todo 编辑页
- 「买菜」自动填回输入框

**适用场景**:

- 编辑类页面(表单、富文本)
- 列表筛选状态
- 媒体播放进度
- 复杂导航栈深度

**不适合的**:

- 敏感信息(密码、token) - 不要走状态恢复,有安全风险
- 服务器数据 - 重启时重新拉接口更安全

**注意**:

- iOS 默认开启 state restoration,Android 要在 AndroidManifest 加配置
- 不是所有数据类型都能恢复(只能存基本类型 + 简单容器),复杂对象要序列化成 JSON
- Web / Desktop 不支持(系统通常不会终止进程,没有必要)

实战中不少 App 不做这个——因为开发成本不低,QA 也要专门测「进程被终止后的恢复」。但医疗、金融、内容创作类 App 对这类体验要求很高,缺失时会明显影响用户体验。

### Flutter 测试金字塔实战

Flutter 测试分三层(跟 web 框架一致):unit test、widget test、integration test。但每层的写法跟 web 不一样,Flutter 有自己的工具。

**Unit Test:测纯 Dart 逻辑**

```dart
import 'package:test/test.dart'

void main() {
  group('User', () {
    test('full name 拼接', () {
      final user = User(firstName: 'Alice', lastName: 'Smith')
      expect(user.fullName, equals('Alice Smith'))
    })
  })
}
```

适合:业务逻辑、工具函数、Model 转换。完全不依赖 widget。

**Widget Test:测单个组件**

```dart
import 'package:flutter_test/flutter_test.dart'

void main() {
  testWidgets('Counter 点击 +1', (tester) async {
    await tester.pumpWidget(MaterialApp(home: CounterPage()))

    expect(find.text('0'), findsOneWidget)

    await tester.tap(find.byIcon(Icons.add))
    await tester.pump()

    expect(find.text('1'), findsOneWidget)
  })
}
```

Widget test 跑在「假的 Flutter 环境」里,不需要模拟器/设备,速度跟 unit test 一样快。这是 Flutter 测试金字塔的重头戏。

**Integration Test:端到端测试**

```dart
import 'package:integration_test/integration_test.dart'
import 'package:flutter_test/flutter_test.dart'

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized()

  testWidgets('登录流程', (tester) async {
    app.main()
    await tester.pumpAndSettle()

    await tester.enterText(find.byKey(Key('email')), 'test@example.com')
    await tester.enterText(find.byKey(Key('password')), 'pwd')
    await tester.tap(find.byKey(Key('submit')))
    await tester.pumpAndSettle()

    expect(find.text('欢迎,Alice'), findsOneWidget)
  })
}
```

跑在真实设备/模拟器上,启动整个 App 测端到端流程。慢、但跟用户体验最接近。

**实战配比**:

- 70% unit test - 业务逻辑/utils
- 25% widget test - 组件交互
- 5% integration test - 关键业务流程

写满 100% 覆盖率不切实际,优先测「容易回归」「业务关键」「重构频繁」的部分。

**Mock 依赖**:

`mocktail` 是 Flutter 主流 mock 库:

```dart
class MockApiClient extends Mock implements ApiClient {}

void main() {
  test('登录失败时显示错误', () async {
    final mockApi = MockApiClient()
    when(() => mockApi.login(any(), any()))
      .thenThrow(Exception('密码错误'))

    final viewModel = LoginViewModel(api: mockApi)
    await viewModel.login('test', 'wrong')

    expect(viewModel.error, equals('密码错误'))
  })
}
```

**Golden test(视觉回归)**:

Flutter 内置「golden test」——截组件图片当 baseline,后续 PR 跟 baseline 比对,差异时报错。Web 端 Chromatic 那种能力 Flutter 内置:

```dart
testWidgets('Card 视觉对比', (tester) async {
  await tester.pumpWidget(MaterialApp(home: MyCard(title: 'Hello')))
  await expectLater(find.byType(MyCard), matchesGoldenFile('my_card.png'))
})
```

第一次跑生成 `my_card.png`,以后 PR 改了 Card 样式截图不一样就 fail。

**CI 集成**:

```yaml
# .github/workflows/test.yml
- run: flutter test --coverage
- uses: codecov/codecov-action@v3   # 上报覆盖率
- run: flutter test integration_test/   # 集成测试
```

Flutter 测试生态比想象中成熟,实战中不少团队跑测试的纪律比 Web 还好——因为 Flutter 的 widget test 太便宜了,几乎没理由不写。
