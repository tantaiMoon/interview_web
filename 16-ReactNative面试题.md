[toc]

## 零、RN 和 Flutter 跨平台的实现方式

### React Native 和 Flutter 分别是如何实现跨端的？

这是一个很高频的对比题，核心抓住这两句：

- **React Native**：`JavaScript/TypeScript + React` 描述 UI，通过**桥接/原生模块调用原生控件和系统能力**
- **Flutter**：`Dart + Flutter Engine` 自己负责**渲染 UI**，不依赖系统原生控件

两者都能跨平台，但实现思路差异明显。

---

### 1. React Native 如何实现跨平台？

React Native 的思路：**业务逻辑写一套，渲染时映射到不同平台的原生组件**。

比如：

- 前端写的是 `<View>`、`<Text>`、`<Image>`
- 在 iOS 上会映射成 `UIView`、`UILabel` 等原生控件
- 在 Android 上会映射成 `View`、`TextView` 等原生控件

所以 React Native 不是自己画界面，而是**调用各平台已有的原生 UI 组件**。

它的典型架构可按下面理解：

```text
JS/TS 业务代码
   ↓
React 组件描述 UI
   ↓
Bridge / JSI / Native Module
   ↓
iOS / Android 原生组件和原生能力
```

**它为什么能跨端？**

原因在于分层复用：

1. **业务逻辑层复用**
   - 状态管理、接口请求、数据处理、路由思想大部分可以复用
2. **组件抽象层复用**
   - RN 提供统一的跨平台组件，如 `View`、`Text`、`ScrollView`
3. **平台适配层做差异处理**
   - 对 iOS 和 Android 的差异，通过 `Platform`、原生模块、平台文件分别处理

**它如何访问原生能力？**

React Native 需要通过桥接机制调用原生代码，例如：

- 调用摄像头
- 调用定位
- 调用蓝牙
- 调用文件系统
- 调用系统通知

如果 RN 官方或社区没有提供现成能力，开发者也可以自己写：

- iOS 原生模块（Objective-C / Swift）
- Android 原生模块（Java / Kotlin）

然后暴露给 JavaScript 调用。

**React Native 跨端的优问题：**

好处：

- 复用前端技术栈，Web/React 开发者上手快
- UI 接近原生，体验比 WebView 方案好
- 可以灵活接入原生代码

问题：

- JS 和原生通信有成本，复杂交互或高频通信场景可能有性能损耗
- 不同平台下仍然需要处理样式、组件行为、系统能力差异
- 对原生环境依赖较强，复杂问题往往要懂 iOS/Android 才能查到底

---

### 2. Flutter 如何实现跨平台？

Flutter 的实现思路和 React Native 差异明显：

**Flutter 不依赖系统原生控件，而是自己用渲染引擎把界面画出来。**

它的典型架构可按下面理解：

```text
Dart 业务代码
   ↓
Flutter Framework（Widget）
   ↓
Flutter Engine
   ↓
Skia / Impeller 渲染
   ↓
在屏幕上直接绘制 UI
```

换个说法：

- React Native 是“**调用原生控件**”
- Flutter 是“**自绘控件**”

Flutter 中的按钮、文本、列表、动画等，大多不是系统直接提供的原生控件，而是 Flutter 框架自己实现的一套 Widget 体系，然后交给渲染引擎统一绘制。

**它为什么能跨端？**

因为 Flutter 做了两层统一：

1. **统一开发语言**
   - 使用 Dart 编写业务逻辑和 UI
2. **统一渲染体系**
   - 不依赖 Android/iOS 原生控件，避免了不同平台控件行为不一致的问题

所以 Flutter 的跨端一致性通常更强，尤其在：

- UI 表现一致性
- 动画效果一致性
- 自定义复杂界面

这些方面往往更有优势。

**Flutter 如何访问原生能力？**

Flutter 通过 `Platform Channel` 与原生通信：

- Dart 发起调用
- iOS/Android 原生代码处理
- 再把结果回传给 Flutter

因此 Flutter 虽然自己负责渲染 UI，但在调用摄像头、定位、蓝牙、推送等系统能力时，仍然需要和原生平台交互。

**Flutter 跨端的优问题：**

好处：

- UI 一致性强，跨平台效果更统一
- 自绘能力强，适合高定制化界面和复杂动画
- 渲染链路统一，不少场景性能表现稳定

问题：

- 不使用原生控件，某些场景下平台原生体验需要额外适配
- 包体积通常相对更大
- 团队需要学习 Dart 和 Flutter 体系，前端迁移成本高于 RN

---

### 3. React Native 和 Flutter 跨端实现的主要区别

可概括为：

1. **渲染方式不同**
   - React Native：调用原生控件
   - Flutter：自己绘制控件

2. **跨端一致性不同**
   - React Native：依赖原生控件，不同平台可能存在差异
   - Flutter：统一渲染，一致性更强

3. **技术栈不同**
   - React Native：JavaScript/TypeScript + React
   - Flutter：Dart + Flutter Framework

4. **与原生交互方式不同**
   - React Native：Bridge / JSI / Native Module
   - Flutter：Platform Channel

5. **适用场景不同**
   - React Native：适合已有 React/前端团队，希望快速复用前端能力
   - Flutter：适合追求高度统一 UI、复杂动画、强定制化跨端产品

---

### 4. 浓缩到一段的回答模板

如果把问题落到“RN 和 Flutter 分别如何实现跨平台”：

可回答为：

> React Native 的跨平台思路是用 JavaScript/React 写业务和组件描述，再把组件映射为 iOS 和 Android 的原生控件，逻辑复用、渲染走原生；Flutter 则是用 Dart 编写业务和 UI，通过 Flutter Engine 自己把界面绘制出来，不依赖系统原生控件，所以它的 UI 一致性通常更强。两者在调用摄像头、定位等系统能力时，都需要通过各自的桥接机制和原生平台通信。

## 一、React Native 基础认知

### 什么是 React Native？它的优势是什么？

React Native 是 Facebook（现 Meta）于 2015 年开源的跨平台移动应用开发框架，它允许开发者使用 JavaScript 和 React 的开发模式来构建更接近原生移动应用。

**为什么会有 React Native？**

在 React Native 出现之前，移动应用开发面临几个痛点：
1. **原生开发成本高**：iOS 需要 Swift/Objective-C，Android 需要 Java/Kotlin，需要维护两套代码
2. **Hybrid 方案性能差**：Cordova/PhoneGap 使用 WebView，性能和体验都不如原生
3. **技术栈割裂**：前端开发者无法直接参与移动端开发，团队协作困难
4. **迭代速度慢**：原生应用每次更新都需要发版和审核，无法快速响应

React Native 的出现就是为了解决这些问题：既要有接近原生的性能，又要有跨平台的开发效率，还要能热更新。

**核心理念：**

"Learn once, write anywhere"（一次学习，到处编写）—— 不同于 "Write once, run anywhere"，React Native 强调的是学习一套技术栈后，可以针对不同平台编写适配代码，而不是完全的一次编写到处运行。

**为什么不是 "Write once, run anywhere"？**

因为 iOS 和 Android 在交互规范、UI 设计、系统 API 等方面存在差异，强行统一会导致：
- 用户体验不符合平台规范（iOS 用户习惯右滑返回，Android 用户习惯按返回键）
- 无法充分利用平台特性（iOS 的 3D Touch，Android 的 Material Design）
- 性能优化受限（不同平台的性能瓶颈不同）

所以 React Native 提供了 `Platform` API 来区分平台，方便按平台写不同逻辑。

**技术本质：**

React Native 并不是将 Web 应用简单地包装成移动应用（如 Cordova/PhoneGap），而是通过 JavaScript 调用原生组件来渲染更接近原生 UI。JavaScript 代码运行在独立的 JavaScript 引擎中（iOS 上是 JavaScriptCore，Android 上是 Hermes 或 V8），通过 Bridge（桥接层）与原生代码通信。

**关键区别：**

```
Cordova/PhoneGap 架构：
JavaScript → WebView → HTML/CSS 渲染
结果：渲染的是网页，性能差

React Native 架构：
JavaScript → Bridge → 原生组件
结果：渲染的是更接近原生 UI，性能好
```

**优势：**

1. **跨平台复用**：
   - 一套代码同时运行在 iOS 和 Android 上
   - 代码复用率通常可达 70-90%
   - 平台特定代码可通过 Platform API 区分处理
   - 降低开发和维护成本

   **为什么能跨平台？**

   React Native 的跨平台能力来自于它的分层架构：
   - **业务逻辑层**（100% 复用）：状态管理、数据处理、网络请求等
   - **UI 组件层**（80-90% 复用）：大部分组件在两个平台上表现一致
   - **平台适配层**（10-20% 差异）：使用 Platform API 处理平台特定逻辑

   ```javascript
   // 业务逻辑完全复用
   const fetchUserData = async (userId) => {
     const response = await fetch(`/api/users/${userId}`)
     return response.json()
   }

   // UI 组件大部分复用
   <View style={styles.container}>
     <Text>Hello World</Text>
   </View>

   // 平台特定代码
   const styles = StyleSheet.create({
     container: {
       paddingTop: Platform.OS === 'ios' ? 20 : 0
     }
   })
   ```

2. **原生性能**：
   - 使用原生组件渲染，不是 WebView
   - UI 性能接近原生应用
   - 可以调用原生 API 和模块
   - 支持原生动画和手势

   **为什么性能接近原生？**

   React Native 的渲染流程：
   ```
   JavaScript 代码 → 计算虚拟 DOM → 通过 Bridge 发送指令 → 原生层渲染更接近原生组件
   ```

   最终用户看到的是 `UIView`（iOS）或 `android.view.View`（Android），而不是 WebView 中的 HTML 元素。所以 React Native 的性能远超 Hybrid 方案。

   **性能瓶颈在哪？**

   主要瓶颈在 Bridge 的异步通信：
   - JS 和原生之间的数据传输需要序列化/反序列化
   - 频繁的跨 Bridge 通信会影响性能
   - 这也是新架构（JSI）要解决的核心问题

3. **热更新能力**：
   - 支持 CodePush 等热更新方案
   - 无需重新发布应用即可修复 bug 和更新功能
   - 绕过应用商店审核流程（仅限 JS 代码更新）
   - 快速迭代和灰度发布

   **为什么能热更新？**

   React Native 应用的代码分为两部分：
   - **原生壳**：包含原生代码和 React Native 框架，需要发版更新
   - **JS Bundle**：包含业务逻辑和 UI 代码，可以动态下载和替换

   热更新的原理：
   ```
   1. 应用启动时检查服务器是否有新的 JS Bundle
   2. 如果有，下载新的 Bundle 到本地
   3. 下次启动时加载新的 Bundle
   4. 更新过程不打断用户当前操作
   ```

   **热更新的限制：**
   - 只能更新 JS 代码，无法更新原生代码
   - 无法添加新的原生模块或权限
   - 苹果应用商店对热更新有限制（无法改变应用的核心功能）

4. **开发效率高**：
   - 使用 React 的声明式编程模式
   - 支持热重载（Hot Reload）和快速刷新（Fast Refresh）
   - 组件化开发，代码复用性强
   - 丰富的调试工具（React DevTools、Flipper）

   **Fast Refresh 有多快？**

   传统原生开发：修改代码 → 重新编译（1-3分钟）→ 重新安装 → 重新打开应用
   React Native：修改代码 → 自动刷新（< 1秒）→ 保持应用状态

   也就是改完代码马上能看到效果，调 UI 会快不少。

5. **生态丰富**：
   - 可以使用 npm 生态中的大量 JavaScript 库
   - 社区活跃，第三方组件库丰富
   - 可以编写或使用原生模块扩展功能
   - 大公司背书（Facebook、Microsoft、Shopify 等）

   **常用的第三方库：**
   - 导航：React Navigation
   - 状态管理：Redux、MobX、Zustand
   - UI 组件：React Native Elements、NativeBase、React Native Paper
   - 网络请求：Axios、React Query
   - 动画：Reanimated、Lottie
   - 图表：Victory Native、React Native Chart Kit

6. **学习成本低**：
   - 前端同学上手快
   - 使用熟悉的 JavaScript/TypeScript
   - React 开发经验可以直接迁移
   - 不需要学习 Swift/Kotlin 等原生语言

   **学习路径：**
   ```
   已有 React 经验 → 1-2 周掌握 React Native 基础
   只有 JavaScript 经验 → 2-4 周掌握 React + React Native
   完全零基础 → 2-3 个月掌握完整技术栈
   ```

**与其他方案对比：**

| 方案 | 性能 | 开发效率 | 跨平台能力 | 学习成本 | 生态成熟度 | 热更新 |
|------|------|---------|-----------|---------|-----------|--------|
| **React Native** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ |
| **Flutter** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ❌ |
| **原生开发** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ❌ |
| **H5/Hybrid** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ |
| **uni-app** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ |

**适合用在：**

1. **适合使用 React Native：**
   - 需要跨平台开发的中小型应用
   - 团队有 React/JavaScript 技术栈
   - 需要快速迭代和热更新
   - 对性能要求不是极致（非游戏、非复杂动画）
   - 预算和时间有限

   **常见场景举例：**
   - 电商应用（商品列表、详情页、购物车）
   - 社交应用（动态流、聊天、个人主页）
   - 内容类应用（新闻、视频、音乐）
   - 工具类应用（待办事项、记账、健康管理）
   - 企业内部应用（OA、CRM、数据看板）

2. **不适合使用 React Native：**
   - 对性能要求极高的应用（3D 游戏、复杂图形处理）
   - 需要大量使用平台特定功能
   - 团队完全没有 JavaScript 经验
   - 需要极致的用户体验和平台一致性

   **为什么不适合？**

   - **3D 游戏**：需要操作 GPU，Bridge 的通信开销太大
   - **复杂动画**：60fps 的流畅动画需要原生实现（但可以用 Reanimated 库改善）
   - **AR/VR 应用**：需要深度集成硬件和传感器
   - **相机类应用**：需要实时处理视频流，JS 性能不够

   **但也有例外：**

   即使是"不适合"的场景，也可通过混合开发解决：
   - 核心功能用原生实现（如相机、视频处理）
   - 其他页面用 React Native 实现（如设置、个人中心）
   - 这样能保证性能，也能享受跨平台的便利

**知名应用案例：**

- Facebook、Instagram（部分页面）
- Discord、Shopify
- 京东、携程、美团（部分模块）
- Coinbase、Bloomberg

**补充要点：**

1. **能说明 React Native 不是简单的 WebView 包装**，而是通过 Bridge 调用原生组件渲染更接近原生 UI，这是与 Cordova/Ionic 等方案的本质区别。Cordova 在 WebView 中渲染 HTML/CSS，最终产出的是网页元素；RN 的 `<View>` 映射到 iOS 的 UIView 或 Android 的 ViewGroup，`<Text>` 映射到 UILabel 或 TextView——渲染产物是原生控件，因此可以享受原生的辅助功能、字体渲染、布局引擎等能力，性能和体验远超 WebView 方案。

2. **了解新架构（JSI、Fabric、TurboModules）的改进方向**：
   - JSI 解决了 Bridge 的性能瓶颈：用 C++ 层的直接引用替代了异步 JSON 消息队列，支持同步调用，通信延迟从 5-10ms 降到 < 1ms
   - Fabric 支持并发渲染和优先级调度：可以中断低优先级渲染来优先响应用户输入，与 React 18 的并发特性配合
   - TurboModules 实现了按需加载，提升启动速度：原生模块不再启动时全量初始化，而是首次访问时才加载，启动时间减少约 50%

3. **能说清 React Native 适合什么项目**，不盲目推崇或贬低，而是根据项目需求做出合理选择。RN 适合中大型内容型应用（电商、社交、IM、工具类），优势是跨平台复用 + 热更新；不适合 3D 游戏、AR/VR、实时视频流等对 GPU/传感器有硬性要求的场景——这些场景即使强用 RN，核心模块也需要原生实现，跨平台的意义有限。

4. **了解热更新的原理和限制**，知道什么能更新、什么无法更新，以及各平台的政策限制。能更新的是 JS Bundle（业务逻辑 + JS 资源），无法更新的是原生代码（新增原生模块、修改 AndroidManifest/Info.plist、更换启动图等都需要发版）。苹果审核条款禁止通过热更新改变应用核心功能，否则会被下架。

5. **能够说出实际的性能优化经验**，如列表优化、动画优化、包体积优化等。例如：FlatList 配合 `getItemLayout` + `React.memo` 可将长列表滚动帧率从 30fps 提升到 60fps；开启 Hermes 引擎的预编译字节码可减少 30-40% 启动时间；`useNativeDriver: true` 将动画下沉到原生层运行，彻底避免 JS 线程阻塞导致的掉帧。

**追问：**

Q: React Native 的性能真的能接近原生吗？
A: 在大多数场景下可以接近原生，但在以下情况会有性能差距：
- 频繁的 JS 和原生通信（旧架构）
- 复杂的列表滚动（可通过优化改善）
- 复杂的动画（可以使用 Reanimated 库）
- 大量的图片渲染（需要做好缓存和优化）

Q: 为什么有些大公司放弃了 React Native？
A: 主要原因包括：
- 团队技术栈不匹配（如 Airbnb 主要是原生开发者）
- 对性能和用户体验要求极高
- 需要深度定制和优化，维护成本高
- 版本升级成本大（早期版本不稳定）

但也有不少公司持续使用并投入（如 Microsoft、Shopify、Discord），关键是看是否适合自己的场景。

Q: React Native 和 Flutter 该选哪个？
A: 取决于团队和项目：
- 团队有 React/JS 经验 → React Native
- 团队有移动端经验或从零开始 → Flutter
- 需要热更新 → React Native
- 追求极致性能 → Flutter
- 需要 Web 端复用 → React Native（可以用 React Native Web）

### React Native 的工作原理是什么？

React Native 的核心是通过 **JavaScript Bridge**（桥接层）连接 JavaScript 代码和原生代码，实现跨语言通信。

**为什么需要 Bridge？**

JavaScript 和原生代码（Objective-C/Swift/Java/Kotlin）运行在不同的环境中：
- JavaScript 运行在 JavaScript 引擎（JavaScriptCore/Hermes）
- 原生代码运行在原生运行时

它们无法直接互相调用，需要一个"翻译官"来传递消息，这就是 Bridge 的作用。

**三层架构：**

1. **JavaScript 层（JS Thread）**：
   - 运行 React 代码和业务逻辑
   - 使用 JavaScript 引擎（iOS 的 JavaScriptCore、Android 的 Hermes/V8）
   - 处理用户交互、状态管理、数据处理
   - 生成虚拟 DOM 树

   **为什么用 JavaScriptCore/Hermes？**

   - **JavaScriptCore**：iOS 系统自带，无需额外打包，但性能一般
   - **Hermes**：Facebook 专为 React Native 优化的 JS 引擎，特点大概是：
     - 启动速度快（预编译字节码）
     - 内存占用低（优化了垃圾回收）
     - 包体积小（字节码比 JS 源码小）
     - Android 上默认使用，iOS 从 0.64 开始支持

2. **Bridge 层（桥接层）**：
   - 负责 JS 和原生之间的通信
   - **异步通信**：所有通信都是异步的，不会阻塞 UI 线程
   - **批量传输**：多个调用会被批量处理，减少通信次数
   - **JSON 序列化**：数据通过 JSON 格式序列化传输
   - 这是旧架构的性能瓶颈所在

   **为什么是异步的？**

   如果是同步通信，JS 调用原生方法时会阻塞 JS 线程，导致：
   - UI 无法响应用户操作
   - 动画卡顿
   - 应用假死

   异步通信虽然增加了复杂度，但保证了 UI 的流畅性。

   **批量传输的好处：**

   ```javascript
   // 不批量：每次调用都要跨 Bridge
   setColor('red')    // Bridge 调用 1
   setSize(100)       // Bridge 调用 2
   setPosition(0, 0)  // Bridge 调用 3

   // 批量：一次性传输多个调用
   batchUpdates([
     { method: 'setColor', args: ['red'] },
     { method: 'setSize', args: [100] },
     { method: 'setPosition', args: [0, 0] }
   ]) // 只需要 1 次 Bridge 调用
   ```

3. **Native 层（UI Thread + Native Modules）**：
   - 原生模块和组件
   - 负责实际的 UI 渲染
   - 处理系统 API 调用（相机、定位、存储等）
   - 响应用户手势和交互

   **为什么要分 UI Thread 和其他线程？**

   - **UI Thread**：负责渲染界面，需要保持流畅（60fps）
   - **Shadow Thread**：负责布局计算，避免阻塞 UI 线程
   - **Native Modules Thread**：执行耗时的原生操作

**工作流程详解：**

```
1. 用户触发事件（如点击按钮）
   ↓
2. 原生层捕获事件，通过 Bridge 发送给 JS 层
   ↓
3. JS 层处理事件，更新状态，计算新的虚拟 DOM
   ↓
4. React 进行 diff 算法，找出需要更新的部分
   ↓
5. 将更新指令通过 Bridge 序列化为 JSON 发送给原生层
   ↓
6. 原生层解析指令，更新对应的原生组件
   ↓
7. 原生层重新渲染 UI
```

**Bridge 的工作机制：**

```javascript
// JavaScript 调用原生方法
NativeModules.CalendarModule.createEvent('Party', 'My House')

// Bridge 将调用序列化为 JSON
{
  "moduleId": 1,
  "methodId": 2,
  "args": ["Party", "My House"]
}

// 原生层接收并执行
// 执行完成后，结果通过 Bridge 返回给 JS
```

**旧架构的性能瓶颈：**

1. **Bridge 是异步的**：所有通信都需要序列化/反序列化，有性能开销
2. **无法同步调用**：某些场景需要同步获取数据时会很麻烦
3. **JSON 序列化开销**：大量数据传输时性能下降明显
4. **单线程 Bridge**：Bridge 本身是单线程的，可能成为瓶颈

**实际案例：为什么 Bridge 会成为瓶颈？**

假设你在做一个滚动列表，每个 item 都有复杂的交互：

```javascript
// 用户快速滚动列表
<FlatList
  data={items}
  renderItem={({ item }) => (
    <TouchableOpacity onPress={() => handlePress(item)}>
      <Image source={{ uri: item.image }} />
      <Text>{item.title}</Text>
    </TouchableOpacity>
  )}
/>
```

滚动时发生的事情：
1. 用户滑动 → 原生层捕获手势 → 通过 Bridge 发送给 JS
2. JS 计算新的可见 items → 通过 Bridge 发送渲染指令给原生
3. 原生层渲染新的 items → 如果有图片加载，又要通过 Bridge 通知 JS
4. 如果用户点击了某个 item，又要通过 Bridge 传递事件

在快速滚动时，这些通信会很频繁，Bridge 就成了瓶颈，导致：
- 滚动不流畅
- 图片加载延迟
- 点击响应慢

**如何缓解？**

1. 使用 `useNativeDriver: true` 让动画在原生层执行
2. 使用 `removeClippedSubviews` 减少渲染的组件数量
3. 优化列表配置（`initialNumToRender`、`maxToRenderPerBatch`）
4. 使用新架构（JSI）彻底解决

**新架构（JSI - JavaScript Interface）：**

从 React Native 0.68 开始引入新架构，主要改进：

**为什么要推出新架构？**

旧架构的 Bridge 虽然实现了跨平台，但存在根本性的性能问题：
1. 所有通信都要序列化成 JSON，开销大
2. 无法同步调用，某些场景很难实现
3. 启动时要加载所有原生模块，启动慢
4. 无法利用 React 18 的并发特性

新架构就是为了从根本上解决这些问题。

**1. JSI（JavaScript Interface）**

- **移除 Bridge**：使用 JSI 调用原生方法，无需序列化
- **同步通信**：支持同步调用原生方法
- **直接访问**：JavaScript 可以直接持有原生对象的引用
- **性能提升**：减少了序列化/反序列化的开销

**JSI 的工作原理：**

```
旧架构：
JS 调用 → JSON 序列化 → Bridge 传输 → JSON 反序列化 → 原生执行
原生返回 → JSON 序列化 → Bridge 传输 → JSON 反序列化 → JS 接收

新架构：
JS 调用 → JSI 调用 → 原生执行 → 直接返回 → JS 接收
```

**代码对比：**

```javascript
// 旧架构：异步调用
NativeModules.MyModule.getValue((value) => {
  console.log(value)
})

// 新架构：同步调用
const value = global.MyModule.getValue()
console.log(value)
```

**为什么同步调用重要？**

某些场景需要同步获取数据：
- 获取设备信息（屏幕尺寸、系统版本）
- 读取本地存储（用户配置、缓存数据）
- 计算布局信息（组件位置、尺寸）

旧架构中这些都要异步处理，代码复杂且容易出错。

**2. Fabric（新渲染器）**

- **并发渲染**：支持 React 18 的并发特性
- **优先级调度**：可以中断低优先级渲染，优先处理用户交互
- **同步布局计算**：可以同步获取布局信息
- **更好的类型安全**：使用 C++ 实现，类型更安全

**为什么需要并发渲染？**

传统渲染是同步的，一旦开始就无法中断：
```
用户输入 → 等待渲染完成 → 响应输入
如果渲染耗时 500ms，用户就要等 500ms
```

并发渲染可以中断低优先级任务：
```
用户输入 → 立即响应 → 后台继续渲染
用户感觉应用很流畅
```

**实际场景：**

```javascript
// 用户在搜索框输入
<TextInput
  value={searchText}
  onChangeText={setSearchText} // 高优先级
/>

// 同时在渲染大量搜索结果
<FlatList
  data={searchResults} // 低优先级
  renderItem={renderItem}
/>
```

旧架构：输入会卡顿，因为在等待列表渲染
新架构：输入流畅，列表渲染可以被中断

**3. TurboModules（新原生模块系统）**

- **按需加载**：原生模块只在使用时才加载，启动更快
- **类型安全**：自动生成类型定义
- **更好的性能**：减少了初始化开销

**为什么按需加载重要？**

旧架构启动流程：
```
1. 加载所有原生模块（即使不用）
2. 初始化所有模块
3. 注册到 JS 环境
4. 应用才能启动

如果有 50 个原生模块，都要初始化，启动慢
```

新架构启动流程：
```
1. 只注册模块名称
2. 应用立即启动
3. 使用时才加载和初始化模块

启动速度提升 50%+
```

**4. Codegen（代码生成器）**

- **自动生成接口代码**：根据 JavaScript 定义自动生成原生接口
- **类型安全**：编译时检查类型错误
- **减少手写模板代码**：少写重复胶水代码

**Codegen 的作用：**

旧架构：手动编写 JS 和原生的接口代码，容易出错
```javascript
// JS 端
NativeModules.MyModule.doSomething(arg1, arg2)

// 原生端（需要手动保持一致）
RCT_EXPORT_METHOD(doSomething:(NSString *)arg1 arg2:(NSNumber *)arg2)
```

新架构：只需要定义一次，自动生成
```typescript
// 定义接口
interface Spec extends TurboModule {
  doSomething(arg1: string, arg2: number): void;
}

// Codegen 自动生成 JS 和原生代码
```

**新旧架构对比：**

| 特性 | 旧架构（Bridge） | 新架构（JSI） |
|------|----------------|--------------|
| 通信方式 | 异步、JSON 序列化 | 同步/异步、调用 |
| 性能 | 有序列化开销 | 性能更好 |
| 类型安全 | 运行时检查 | 编译时检查 |
| 启动速度 | 较慢（加载所有模块） | 更快（按需加载） |
| 并发渲染 | 不支持 | 支持 |
| 同步调用 | 不支持 | 支持 |

**新架构的优势：**

1. **启动速度提升**：TurboModules 按需加载，减少启动时间
   - 旧架构：启动时间 2-3 秒
   - 新架构：启动时间 1-1.5 秒
   - 提升约 50%

2. **运行性能提升**：JSI 调用，减少通信开销
   - 旧架构：Bridge 通信延迟 5-10ms
   - 新架构：JSI 调用延迟 < 1ms
   - 提升约 10 倍

3. **更好的用户体验**：Fabric 支持并发渲染，交互更流畅
   - 输入响应更快
   - 滚动更流畅
   - 动画更顺滑

4. **开发体验提升**：Codegen 自动生成代码，类型更安全
   - 减少手动编写代码
   - 编译时发现类型错误
   - 更好的 IDE 支持

**新架构的迁移成本：**

虽然新架构带来不少好处，但迁移也有成本：
1. 需要升级到 0.68+ 版本
2. 第三方库可能不兼容（需要等待更新）
3. 自定义原生模块需要重写
4. 可能遇到新的 bug（新架构还在完善中）

**建议：**
- 新项目：使用新架构
- 老项目：等待生态成熟后再迁移（2024 年后）
- 关键业务：谨慎升级，充分测试

**补充要点：**

1. **能说清 Bridge 的工作机制和性能瓶颈**，以及新架构如何解决这些问题。Bridge 的问题出在：所有 JS 与原生之间的通信需要经过 JSON 序列化→异步消息队列→反序列化三个步骤，每次跨 Bridge 调用延迟约 5-10ms。在快速滚动的列表中，每秒可能产生数十次 Bridge 调用（手势事件→JS 计算→渲染指令），队列堵塞就会导致掉帧。JSI 通过 C++ 层直接持有 JS 引擎对象的引用，省去了序列化和排队，将通信延迟降到 < 1ms。

2. **了解 JSI、Fabric、TurboModules 的作用和优势**，说明 React Native 正在向更高性能的方向演进。JSI 解决通信层的序列化开销；Fabric 解决渲染层的并发能力（支持 React 18 Concurrent Mode，可中断低优先级渲染优先响应用户输入）；TurboModules 解决启动层的全量初始化问题（模块按需加载而非启动时全部注册）。三者分别从通信、渲染、加载三个维度优化了旧架构的瓶颈。

3. **能说清新架构什么时候该迁**，不要只看版本更新，要结合项目成本。新架构从 0.68 开始支持、0.74 成为默认选项，但迁移需要：升级 RN 版本、确认第三方库是否兼容 TurboModules、重写自定义原生模块的 Spec 定义。存量项目可以渐进式迁移——新旧架构可以共存，旧的 NativeModules 继续走 Bridge，新增模块走 TurboModules，无需一次性改完。

4. **了解 Hermes 引擎的优势**，知道为什么 Android 默认使用 Hermes。Hermes 的核心优化是预编译字节码：构建期就将 JS 源码编译成字节码，运行时直接加载执行，跳过了运行时解析和编译的开销。这使得 Android 端启动速度提升 30-40%，内存占用降低（优化了 GC 策略），包体积也更小（字节码比 JS 源码紧凑）。iOS 从 RN 0.64 开始也支持 Hermes。

5. **能够说出实际的性能数据**，如启动速度提升 50%，通信延迟降低 10 倍等。旧架构 Bridge 通信延迟约 5-10ms/次，新架构 JSI 同步调用 < 1ms；旧架构冷启动约 2-3 秒（全量初始化原生模块），新架构约 1-1.5 秒（TurboModules 按需加载）；Hermes 字节码预编译使 TTI（Time to Interactive）减少 30-40%。

**追问：**

Q: 新架构什么时候能稳定？
A:
- 0.68 版本（2022 年）开始支持新架构
- 0.70 版本（2022 年）新架构趋于稳定
- 0.74 版本（2024 年）新架构成为默认选项
- 预计 2024-2025 年生态完全成熟

Q: 旧架构的应用需要立即升级吗？
A: 不一定，取决于：
- 如果性能满足需求，可以继续使用旧架构
- 如果遇到性能瓶颈，可以考虑升级
- 如果是新项目，建议使用新架构
- 如果依赖不少第三方库，等待生态成熟

Q: JSI 和 Bridge 能共存吗？
A: 可以，新架构支持渐进式迁移：
- 核心框架使用 JSI
- 旧的原生模块继续使用 Bridge
- 逐步将原生模块迁移到 TurboModules
- 最终完全移除 Bridge

### React Native 与 React Web 有什么区别？

虽然都使用 React，但两者有明显差异：

**1. 组件不同**

```javascript
// React Web
import React from 'react'

function App() {
  return (
    <div className="container">
      <h1>标题</h1>
      <p>段落</p>
      <button onClick={handleClick}>按钮</button>
    </div>
  )
}

// React Native
import React from 'react'
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native'

function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>标题</Text>
      <Text>段落</Text>
      <TouchableOpacity onPress={handleClick}>
        <Text>按钮</Text>
      </TouchableOpacity>
    </View>
  )
}
```

**2. 样式处理不同**

```javascript
// React Web - 使用 CSS
<div style={{ display: 'flex', flexDirection: 'row' }}>

// React Native - 使用 StyleSheet
<View style={styles.container}>

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row'
  }
})
```

**3. 事件处理不同**

```javascript
// React Web
<button onClick={handleClick}>点击</button>

// React Native
<TouchableOpacity onPress={handleClick}>
  <Text>点击</Text>
</TouchableOpacity>
```

**4. 路由导航不同**

```javascript
// React Web - React Router
import { BrowserRouter, Route } from 'react-router-dom'

// React Native - React Navigation
import { NavigationContainer } from '@react-navigation/native'
import { createStackNavigator } from '@react-navigation/stack'
```

**为什么会有这些差异？**

React Web 的底层是浏览器 DOM，React Native 的底层是原生平台的渲染引擎（iOS 的 UIKit / Android 的 Views）。同一个 React 组件树的渲染结果，Web 侧生成 DOM 节点，Native 侧生成原生视图树。这个底层差异向上传导，导致了组件、样式、事件、路由四个层面的分叉：

1. **组件差异的根源**：浏览器提供 `<div>`、`<span>` 等通用容器，原生平台没有这些概念——iOS 只有 UIView，Android 只有 ViewGroup。React Native 需要用 `<View>`/`<Text>` 等抽象组件映射到对应平台的原生视图，无法复用 HTML 标签。此外 `<Text>` 在 RN 中是强制性的：原生平台要求文字需要放在专门的文本控件内，无法像 Web 那样直接写在 `<div>` 里。

2. **样式差异的根源**：浏览器有完整 CSS 引擎，支持级联、伪类、媒体查询、百分比布局等；原生平台只有各自的声明式样式 API（iOS 的 NSLayoutConstraint / Android 的 XML Layout）。React Native 选择 Flexbox 作为统一布局模型并内嵌到 StyleSheet 中，牺牲了 CSS 的全量能力，换来跨平台一致性。StyleSheet.create 在注册时做校验能捕获无效属性名，也方便原生层做样式预处理——这是 inline style 做不到的。

3. **事件差异的根源**：浏览器的事件模型基于 DOM 冒泡/捕获，原生平台的手势体系差异明显（iOS 的 UIGestureRecognizer / Android 的 MotionEvent）。`onClick` 在移动端不适用——手指触摸的"点击"需要区分 tap、long press、swipe、pinch 等，所以 RN 提供 `onPress`、`onLongPress` 等更细粒度的事件，以及 `GestureResponderSystem` 来处理手势竞争。

4. **路由差异的根源**：Web 的 URL 是全局唯一的，浏览器自带前进/后退栈和 `window.history`；原生 App 没有 URL 概念，页面导航靠显式维护的栈或 Tab 容器。React Navigation 不得不自己实现整个导航状态机，这也是它比 React Router 复杂得多的原因——它要同时管理栈、Tab、Drawer 等多种导航容器的嵌套状态。

**选型时的核心考量**：如果应用以内容展示为主、交互复杂度不高，React Web（甚至 React Native Web）的代码复用率可以很高；如果应用深度依赖平台能力（摄像头、传感器、后台任务、精细手势），React Native 的组件/样式/事件分叉几乎是不可避免的，需要在设计阶段就规划好平台抽象层。

## 二、核心组件与 API

### React Native 常用的核心组件有哪些？

**1. 基础组件**

```javascript
import {
  View,        // 容器组件，类似 div
  Text,        // 文本组件
  Image,       // 图片组件
  ScrollView,  // 滚动容器
  TextInput,   // 输入框
  Button,      // 按钮
  TouchableOpacity,  // 可点击的透明度变化组件
  TouchableHighlight, // 可点击的高亮组件
  FlatList,    // 高性能列表
  SectionList, // 分组列表
  Modal,       // 模态框
  ActivityIndicator, // 加载指示器
  Switch,      // 开关
  StatusBar    // 状态栏
} from 'react-native'
```

**2. 使用示例**

代码见 [实践：LoginScreen 登录页](#practices-login)

### FlatList 和 ScrollView 有什么区别？什么时候用哪个？

这是 React Native 中常见的性能问题之一，选错组件可能导致严重的性能问题。

**为什么需要两种滚动组件？**

因为它们的使用场景差异明显：
- ScrollView 适合内容固定、数量少的场景
- FlatList 适合内容动态、数量多的场景

**ScrollView：**

- 一次性渲染所有子组件
- 适合少量数据（< 100 条）
- 支持横向和纵向滚动
- 可以包含任意组件

**工作原理：**

```javascript
<ScrollView>
  <Component1 />  // 立即渲染
  <Component2 />  // 立即渲染
  <Component3 />  // 立即渲染
  // ... 所有组件都会立即渲染
</ScrollView>
```

**为什么一次性渲染所有组件？**

因为 ScrollView 不知道子组件的高度，无法做虚拟化。如果只渲染可见部分，滚动条的位置就无法计算。

**ScrollView 的性能问题：**

```javascript
// ❌ 错误：渲染 1000 条数据
<ScrollView>
  {data.map(item => (
    <ItemCard key={item.id} data={item} />
  ))}
</ScrollView>

// 问题：
// 1. 首次渲染要创建 1000 个组件，耗时 5-10 秒
// 2. 内存占用巨大，可能导致应用崩溃
// 3. 滚动时卡顿，因为要维护 1000 个组件
```

**FlatList：**

- 懒加载，只渲染可见区域的项
- 适合大量数据（> 100 条）
- 内置优化（复用、分页加载）
- 性能更好

**工作原理（虚拟化）：**

```javascript
<FlatList
  data={[1, 2, 3, ..., 1000]}
  renderItem={({ item }) => <Item data={item} />}
/>

// 实际渲染：
// 屏幕可见 10 条 → 只渲染 10 + 缓冲区（5 条）= 15 条
// 用户向下滚动 → 回收顶部不可见的，渲染底部新出现的
// 始终只维护 15-20 个组件，而不是 1000 个
```

**为什么 FlatList 性能好？**

1. **虚拟化**：只渲染可见部分
2. **组件复用**：滚动出屏幕的组件会被复用
3. **分批渲染**：不会一次性渲染所有数据
4. **懒加载**：支持滚动到底部时加载更多

**使用场景对比：**

```javascript
// ✅ ScrollView - 适合少量固定内容
<ScrollView>
  <Header />
  <Banner />
  <ProductList items={products.slice(0, 10)} />
  <Footer />
</ScrollView>

// ✅ FlatList - 适合大量列表数据
<FlatList
  data={products}
  renderItem={({ item }) => <ProductCard product={item} />}
  keyExtractor={item => item.id}
  // 性能优化配置
  initialNumToRender={10}        // 首次渲染 10 条
  maxToRenderPerBatch={10}       // 每批最多渲染 10 条
  windowSize={5}                 // 维护 5 个屏幕高度的内容
  removeClippedSubviews={true}   // 移除不可见的视图
  onEndReached={loadMore}        // 滚动到底部加载更多
  onEndReachedThreshold={0.5}    // 距离底部 50% 时触发
/>
```

**常见错误：**

```javascript
// ❌ 错误 1：用 ScrollView 渲染大量数据
<ScrollView>
  {bigData.map(item => <Item key={item.id} />)}
</ScrollView>

// ❌ 错误 2：在 FlatList 外层套 ScrollView
<ScrollView>
  <FlatList data={data} />  // FlatList 的虚拟化失效
</ScrollView>

// ❌ 错误 3：FlatList 嵌套 FlatList
<FlatList
  data={categories}
  renderItem={({ item }) => (
    <FlatList data={item.products} />  // 性能很差
  )}
/>
```

**处理方式：**

```javascript
// ✅ 正确 1：大量数据用 FlatList
<FlatList data={bigData} renderItem={renderItem} />

// ✅ 正确 2：需要混合内容时用 FlatList 的 ListHeaderComponent
<FlatList
  data={products}
  ListHeaderComponent={() => (
    <>
      <Header />
      <Banner />
    </>
  )}
  ListFooterComponent={Footer}
  renderItem={renderItem}
/>

// ✅ 正确 3：嵌套列表用 SectionList
<SectionList
  sections={[
    { title: '分类1', data: [...] },
    { title: '分类2', data: [...] }
  ]}
  renderItem={renderItem}
  renderSectionHeader={({ section }) => <Text>{section.title}</Text>}
/>
```

**性能对比（1000 条数据）：**

| 指标 | ScrollView | FlatList |
|------|-----------|----------|
| 首次渲染时间 | 5-10 秒 | < 1 秒 |
| 内存占用 | 500+ MB | 50-100 MB |
| 滚动帧率 | 20-30 fps | 55-60 fps |
| 是否会崩溃 | 可能 | 不会 |

**补充要点：**

1. **能解释虚拟化的原理**：只渲染可见部分，复用组件，减少内存占用。FlatList 在内部维护一个"可视窗口"，只渲染窗口内的 item + 上下缓冲区的少量 item。滚动出窗口的 item 会被卸载回收，新进入窗口的 item 被创建复用，因此无论数据总量多大，实际挂载的组件数量始终稳定在十几到几十个，内存占用可控。

2. **知道 FlatList 的性能优化参数**：
   - `initialNumToRender`：首次渲染数量——设小加快首屏，设大减少空白闪屏
   - `maxToRenderPerBatch`：每批增量渲染数量——设大滚动时填充快但可能卡帧，设小流畅但空白多
   - `windowSize`：维护的窗口大小（默认 21 屏）——调到 5-7 足够多数场景，减少内存占用
   - `removeClippedSubviews`：移除不可见视图——Android 上效果显著，iOS 提升有限
   - `getItemLayout`：提前告知 item 高度，跳过 onLayout 测量——定高列表必用，可跳过昂贵的异步测量步骤

3. **了解常见的性能陷阱**：
   - 避免在 ScrollView 中渲染大量数据——ScrollView 一次性渲染所有子组件，1000 条数据会创建 1000 个组件，内存溢出且卡顿
   - 避免在 FlatList 外层套 ScrollView——FlatList 高度变为无限，虚拟化失效，退化为全量渲染
   - 避免嵌套 FlatList——内层 FlatList 无法确定自身高度，同样导致虚拟化失效；应用 SectionList 或 FlatList + ListHeaderComponent 替代

4. **能够根据场景选择合适的组件**：
   - 少量固定内容（< 100 条） → ScrollView：简单直接，无需虚拟化开销
   - 大量列表数据（> 100 条） → FlatList：虚拟化保证性能
   - 分组列表 → SectionList：内置分组头支持和按 section 虚拟化
   - 混合内容 → FlatList + ListHeaderComponent：头部的静态内容和列表的虚拟化兼得

**追问：**

Q: FlatList 的 windowSize 是什么意思？
A: windowSize 决定了维护多少个屏幕高度的内容：
- windowSize={5} 表示维护 5 个屏幕高度
- 当前屏幕上方 2 个屏幕高度 + 当前屏幕 + 下方 2 个屏幕高度
- 超出这个范围的组件会被卸载
- 值越大，滚动越流畅，但内存占用越高
- 默认值是 21（比较保守）

Q: 为什么无法在 FlatList 外层套 ScrollView？
A: 因为 FlatList 需要知道自己的高度才能计算虚拟化：
- 套在 ScrollView 里，FlatList 的高度是无限的
- FlatList 会认为所有内容都可见
- 虚拟化失效，所有 item 都会被渲染
- 性能退化到和 ScrollView 一样

Q: 如何优化 FlatList 的性能？
A: 多个方面：
1. 使用 `keyExtractor` 提供稳定的 key
2. 使用 `getItemLayout` 避免动态测量高度
3. 使用 `React.memo` 避免 item 不必要的重渲染
4. 优化 `renderItem`，避免创建新的函数和对象
5. 使用 `removeClippedSubviews` 移除不可见视图
6. 图片使用合适的尺寸，避免过大

### React Native 中如何处理图片？

**1. 本地图片**

```javascript
// 静态引入
<Image source={require('./assets/logo.png')} />

// 根据条件选择图片
const icon = isActive
  ? require('./assets/active.png')
  : require('./assets/inactive.png')
<Image source={icon} />
```

**2. 网络图片**

```javascript
<Image
  source={{ uri: 'https://example.com/image.jpg' }}
  style={{ width: 200, height: 200 }}
/>

// 带请求头的网络图片
<Image
  source={{
    uri: 'https://example.com/image.jpg',
    headers: {
      Authorization: 'Bearer token'
    }
  }}
/>
```

**3. Base64 图片**

```javascript
<Image
  source={{ uri: 'data:image/png;base64,iVBORw0KGgo...' }}
  style={{ width: 100, height: 100 }}
/>
```

**4. 图片优化**

```javascript
<Image
  source={{ uri: imageUrl }}
  style={styles.image}
  // 调整大小模式
  resizeMode="cover" // cover | contain | stretch | repeat | center
  // 加载指示器
  loadingIndicatorSource={require('./loading.gif')}
  // 默认图片
  defaultSource={require('./placeholder.png')}
  // 淡入效果
  fadeDuration={300}
  // 事件回调
  onLoad={() => console.log('加载完成')}
  onError={(error) => console.log('加载失败', error)}
/>
```

## 三、样式与布局

### React Native 中的样式系统是怎样的？

React Native 使用类似 CSS 的样式系统，但有一些重要区别。

**1. 基本用法**

```javascript
import { StyleSheet } from 'react-native'

// 推荐：使用 StyleSheet.create
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    padding: 20
  },
  text: {
    fontSize: 16,
    color: '#333',
    fontWeight: 'bold'
  }
})

// 使用
<View style={styles.container}>
  <Text style={styles.text}>Hello</Text>
</View>
```

**2. 样式组合**

```javascript
// 数组方式组合样式（后面的覆盖前面的）
<View style={[styles.base, styles.active, { marginTop: 10 }]} />

// 条件样式
<View style={[
  styles.button,
  isActive && styles.activeButton,
  isDisabled && styles.disabledButton
]} />
```

**3. 与 CSS 的主要区别**

```javascript
// ❌ CSS 写法（不支持）
.container {
  display: flex;
  flex-direction: row;
  background-color: #fff;
}

// ✅ React Native 写法
const styles = StyleSheet.create({
  container: {
    // 默认就是 flex 布局，不需要写 display: 'flex'
    flexDirection: 'row',
    backgroundColor: '#fff', // 驼峰命名
    // 没有单位，数字默认是 dp/pt
    padding: 20,
    // 不支持简写
    // padding: '10px 20px' ❌
    paddingVertical: 10,
    paddingHorizontal: 20
  }
})
```

**4. 不支持的 CSS 特性**

```javascript
// ❌ 不支持
- 级联选择器（.parent .child）
- 伪类（:hover, :active）
- 伪元素（::before, ::after）
- 百分比宽高（部分支持）
- calc() 函数
- CSS 变量
- 动画（需要用 Animated API）
- 媒体查询（需要用 Dimensions API）
```

### React Native 中如何实现响应式布局？

**1. 使用 Flexbox**

```javascript
const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center'
  },
  left: {
    flex: 1 // 占据剩余空间
  },
  right: {
    width: 100 // 固定宽度
  }
})
```

**2. 使用 Dimensions API**

```javascript
import { Dimensions } from 'react-native'

const { width, height } = Dimensions.get('window')

const styles = StyleSheet.create({
  container: {
    width: width * 0.9, // 屏幕宽度的 90%
    height: height / 2   // 屏幕高度的一半
  }
})

// 监听屏幕尺寸变化（横竖屏切换）
useEffect(() => {
  const subscription = Dimensions.addEventListener('change', ({ window }) => {
    setScreenWidth(window.width)
    setScreenHeight(window.height)
  })

  return () => subscription?.remove()
}, [])
```

**3. 使用 PixelRatio 适配不同分辨率**

```javascript
import { PixelRatio } from 'react-native'

// 获取设备像素比
const pixelRatio = PixelRatio.get()

// 将设计稿尺寸转换为实际尺寸
function normalize(size) {
  const scale = width / 375 // 假设设计稿宽度为 375
  return Math.round(size * scale)
}

const styles = StyleSheet.create({
  text: {
    fontSize: normalize(16)
  }
})
```

**4. 使用 Platform API 区分平台**

```javascript
import { Platform, StyleSheet } from 'react-native'

const styles = StyleSheet.create({
  container: {
    ...Platform.select({
      ios: {
        paddingTop: 20,
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 3.84
      },
      android: {
        paddingTop: 0,
        elevation: 5
      }
    })
  },
  text: {
    fontSize: Platform.OS === 'ios' ? 16 : 14
  }
})
```

**5. 封装响应式工具**

代码见 [实践：响应式工具模块](#practices-responsive)

### React Native 中如何实现阴影效果？

iOS 和 Android 的阴影实现方式不同，需要分别处理。

```javascript
import { Platform, StyleSheet } from 'react-native'

const styles = StyleSheet.create({
  // 方案1: 使用 Platform.select
  card: {
    backgroundColor: '#fff',
    borderRadius: 8,
    padding: 20,
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 3.84
      },
      android: {
        elevation: 5
      }
    })
  },

  // 方案2: 封装通用阴影函数
  cardWithShadow: {
    backgroundColor: '#fff',
    borderRadius: 8,
    padding: 20,
    ...getShadow(5)
  }
})

// 封装阴影函数
function getShadow(elevation) {
  return {
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: {
          width: 0,
          height: elevation / 2
        },
        shadowOpacity: 0.25,
        shadowRadius: elevation
      },
      android: {
        elevation: elevation
      }
    })
  }
}
```

## 四、导航与路由

### React Native 中如何实现页面导航？

React Native 官方推荐使用 **React Navigation** 库。

**1. 安装和基本配置**

```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context
npm install @react-navigation/stack
```

**2. 栈导航（Stack Navigator）**

代码见 [实践：Stack Navigator](#practices-stack-nav)

**3. 底部标签导航（Tab Navigator）**

代码见 [实践：Tab Navigator](#practices-tab-nav)

**栈导航与 Tab 导航的主要区别与选型**

| 维度 | Stack Navigator | Tab Navigator |
| --- | --- | --- |
| 导航语义 | 层级递进（进入详情→返回列表） | 平级切换（首页/发现/我的） |
| 页面生命周期 | push 时新页面 mount，pop 时旧页面恢复 | 各 Tab 首次 mount 后常驻，切换不卸载 |
| 返回行为 | 系统返回键回到上一页 | 系统返回键退出 App（需额外处理） |
| 状态保持 | 默认不保持（pop 后状态丢失，需 persist） | 默认保持，切换回来状态还在 |
| 内存占用 | 栈越深占用越高（每个页面独立视图树） | 固定，Tab 数量有限且通常 3~5 个 |

**为什么需要两种导航？**

移动端 App 的信息架构几乎都由两种模式组合而成：横向平级切换（Tab）+ 纵向深度探索（Stack）。它们分别对应不同的用户理解模型——Tab 是"我去另一个地方"，Stack 是"我钻进去看看再出来"。React Navigation 提供这两种基础容器，就是为了组合出任意嵌套结构，例如：Tab 内嵌 Stack（每个 Tab 有自己的深度栈）、Stack 内嵌 Tab（详情页底部有子 Tab）。

**实际选型考量：**

1. **Tab 数量控制在 3~5 个**。超过 5 个应考虑 Drawer 导航或分组 Tab。
2. **Stack 层级不宜超过 5 层**。深层嵌套不仅内存压力大，返回路径也难以理解。深层页面应考虑用 `replace` 或 `reset` 截断导航栈（如登录成功后 reset 到首页）。
3. **嵌套导航的性能**：每个嵌套的 Navigator 都会维护独立的路由状态，深层嵌套时 `navigation.navigate` 可能需要指定 target，否则会找错容器——这是 React Navigation 嵌套场景常见的问题。
4. **替代方案**：对于需要原生级导航性能和视觉的场景，可考虑 `react-native-navigation`（Wix 出品）——它使用原生导航控制器，不走 JS 渲染，但 API 设计和社区生态不如 React Navigation 活跃。

**4. 导航方法总结**

```javascript
// 跳转到指定页面
navigation.navigate('Details', { id: 1 })

// 返回上一页
navigation.goBack()

// 返回到栈顶
navigation.popToTop()

// 替换当前页面
navigation.replace('Login')

// 重置导航栈
navigation.reset({
  index: 0,
  routes: [{ name: 'Home' }]
})
```

## 五、性能优化

### React Native 常见的性能问题有哪些？如何优化？

React Native 的性能问题通常集中在三个层面：**渲染层**（组件重渲染、列表卡顿）、**通信层**（Bridge 拥堵、JSON 序列化开销）、**资源层**（图片、字体、Bundle 体积）。理解这些瓶颈来源，才能对症下药。

**为什么 React Native 容易出现性能问题？**

相比纯原生开发，RN 多了 JavaScript 引擎和 Bridge 这两层：

1. JS 单线程，长任务会阻塞 UI 响应。
2. Bridge 通信需要 JSON 序列化，数据量大时延迟明显。
3. 列表、动画、手势这类需要高频更新的场景，容易暴露问题。
4. 第三方库质量参差，可能引入意外开销。

下面按优化层级展开。

#### 1. 列表性能优化（最高频痛点）

列表是 RN 应用中性能问题的"重灾区"，90% 的卡顿都发生在列表上。

```javascript
<FlatList
  data={data}
  renderItem={({ item }) => <Item data={item} />}
  keyExtractor={item => item.id}
  initialNumToRender={10}        // 首屏渲染条数,越小首屏越快
  maxToRenderPerBatch={10}       // 每批增量渲染条数
  windowSize={5}                 // 维护几个屏幕高度的内容(默认 21,过大浪费内存)
  removeClippedSubviews={true}   // 卸载屏幕外不可见的子视图(Android 显著)
  updateCellsBatchingPeriod={50} // 批处理周期(ms),越大越省 CPU 但更新越慢
  getItemLayout={(data, index) => ({ // 提前告知每项尺寸,跳过测量
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index
  })}
/>
```

**每个参数的作用与权衡：**

1. **`initialNumToRender`**：首屏先渲染多少条。设小一点 TTI 更短，但用户滚动到下方时可能看到空白闪屏。
2. **`windowSize`**：决定内存占用和滚动流畅度的平衡。值越大越流畅但占内存,默认 21 偏保守,大多数场景可以调到 5~7。
3. **`removeClippedSubviews`**：让屏幕外的视图从原生层卸载。Android 上效果显著，iOS 上提升有限，且对受控组件（如输入框）可能造成状态丢失。
4. **`getItemLayout`**：跳过动态测量步骤，是长列表性能优化的关键。**只有定高 item 才能用**，但凡有动态高度就用不上。

**为什么这些参数能提升性能？**

FlatList 的虚拟化机制依赖测量和回收,但默认配置偏保守(为了兼容动态高度场景)。当你的 item 高度固定时,`getItemLayout` 能跳过昂贵的 `onLayout` 测量,效果立竿见影——长列表滚动从 30fps 提升到 60fps 是常见结果。

**列表项本身的优化（容易被忽略）：**

```javascript
// ❌ 反面：每次父组件 render，所有 item 都重新渲染
const Item = ({ data }) => {
  return (
    <View style={{ padding: 10 }}>  // 行内对象，每次新引用
      <Text>{data.name}</Text>
      <Button onPress={() => handleClick(data.id)} />  // 行内函数
    </View>
  )
}

// ✅ 正面：用 memo + 提取样式 + 稳定回调
const styles = StyleSheet.create({
  item: { padding: 10 }
})

const Item = React.memo(({ data, onClick }) => {
  return (
    <View style={styles.item}>
      <Text>{data.name}</Text>
      <Button onPress={() => onClick(data.id)} />
    </View>
  )
})

// 父组件用 useCallback 稳定 onClick 引用
const handleClick = useCallback((id) => { ... }, [])
```

**为什么 `React.memo` 在列表里特别重要？**

FlatList 滚动时会频繁触发 render（每次新 item 进入可视区都可能触发整个列表重渲染）。如果 item 没 memo,即使 props 没变也会重渲;1000 项列表中只要 5% 的 item 重渲,就是 50 次无意义的 reconcile 工作。

#### 2. 避免不必要的重渲染

```javascript
// 子组件用 React.memo 阻止 props 未变时的重渲染
const ExpensiveItem = React.memo(({ data }) => {
  return <Text>{data.name}</Text>
}, (prev, next) => {
  // 自定义比较：只比关心的字段
  return prev.data.id === next.data.id && prev.data.name === next.data.name
})
```

**RN 中的常见重渲染陷阱：**

| 反模式 | 问题 | 处理方式 |
|--------|------|---------|
| `style={{ flex: 1 }}` 行内写 | 每次新对象，触发 memo 失效 | `StyleSheet.create` 抽出 |
| `onPress={() => doSomething(id)}` | 每次新函数引用 | `useCallback` 稳定引用 |
| 把 store 整个塞进 Context | 任何字段变都触发所有消费者 | 拆 Context 或用 Zustand |
| 父组件 state 频繁变（如键盘高度） | 子树跟着重渲 | 把这里 state 下沉到独立组件 |

**经验法则：先用 React DevTools Profiler 定位更接近瓶颈，再针对性 memo**——盲目加 memo 会把维护成本和比较开销都拉上来，得不偿失。

#### 3. 使用原生驱动的动画（useNativeDriver）

```javascript
// ❌ JS 驱动：动画运行在 JS 线程，长任务会卡顿
Animated.timing(fadeAnim, {
  toValue: 1,
  duration: 1000,
  useNativeDriver: false  // 默认值
}).start()

// ✅ 原生驱动：动画完全在原生层运行，不受 JS 阻塞
Animated.timing(fadeAnim, {
  toValue: 1,
  duration: 1000,
  useNativeDriver: true  // 关键
}).start()
```

**为什么 `useNativeDriver: true` 能大幅提升动画流畅度？**

不开原生驱动时,动画的每一帧都要走这个流程:

```
JS 线程计算下一帧值 → Bridge 序列化 → 原生层应用样式 → 浏览器/系统绘制
```

每帧都要跨 Bridge,只要 JS 线程在做别的事(比如解析数据、响应触摸),动画就会掉帧。

开了原生驱动后:

```
JS 一次性把动画曲线告诉原生 → 原生自己计算每帧并渲染
```

整个动画过程 JS 线程完全空闲,所以即使 JS 在跑长任务,动画也不会卡。

**`useNativeDriver` 的限制：** 只支持 `transform`（translate/scale/rotate）、`opacity` 这类**非布局属性**。`width`、`height`、`backgroundColor`、`flex` 这些会触发布局计算的属性无法用——这是绝大多数 RN 动画问题的根源。需要这些属性时改用 `react-native-reanimated`，它能把更多动画下沉到原生层。

#### 4. 图片与资源优化

```javascript
// ❌ 大图渲染
<Image source={{ uri: 'https://example.com/4k-photo.jpg' }} />

// ✅ 服务端裁剪 + 占位 + 缓存
<FastImage
  source={{
    uri: 'https://cdn.example.com/photo.jpg?w=375&h=200',
    priority: FastImage.priority.normal,
    cache: FastImage.cacheControl.immutable
  }}
  style={styles.image}
  resizeMode={FastImage.resizeMode.cover}
/>
```

**优化要点：**

1. **服务端裁剪**：列表里的缩略图按真实显示尺寸（375×200）请求，避免拉 4K 原图——RN 会在内存里 decode 整张图,4K 图能占 50MB+ 内存。
2. **使用 `react-native-fast-image`**：基于原生 SDWebImage / Glide 实现，缓存和优先级管理远胜内置 `Image`。
3. **`resizeMode` 选对**：`cover` 比 `contain` 性能好（不需要计算空白区域），`stretch` 几乎不用。
4. **本地资源走 `require()`**：编译期就处理好，不走运行时加载。

#### 5. 减少 Bridge 通信

```javascript
// ❌ 频繁过 Bridge：滚动时每帧都把位置同步到 JS
<ScrollView onScroll={(e) => {
  setScrollY(e.nativeEvent.contentOffset.y)  // 每帧 setState
}}>

// ✅ 用 Animated.event + useNativeDriver 在原生层处理
const scrollY = useRef(new Animated.Value(0)).current
<Animated.ScrollView
  onScroll={Animated.event(
    [{ nativeEvent: { contentOffset: { y: scrollY } } }],
    { useNativeDriver: true }
  )}
>
```

**为什么这能大幅提升性能？**

`onScroll` 在原生层每秒触发 60 次,如果走 setState,就有 60 次跨 Bridge 调用 + 60 次组件重渲。用 `Animated.Value` 后,所有滚动数据全在原生层流转,JS 线程零消耗。

#### 6. 启动性能优化

启动慢通常出在三个地方:**JS Bundle 加载、原生模块初始化、首屏渲染**。优化手段:

1. **开 Hermes 引擎**：预编译字节码,启动速度提升 30%~40%。
2. **代码分割 + 懒加载**：Splash 页只加载首屏需要的代码,其他页面动态 import。
3. **新架构开 TurboModules**：原生模块按需加载，避免启动时全量初始化。
4. **首屏组件层级压平**：嵌套越深布局越慢，能用 `flexDirection` 解决就别套额外 View。

#### 7. 内存优化

```javascript
// ✅ 组件卸载时清理订阅、定时器、监听
useEffect(() => {
  const sub = DeviceEventEmitter.addListener('event', handler)
  const timer = setInterval(tick, 1000)
  return () => {
    sub.remove()
    clearInterval(timer)
  }
}, [])

// ✅ 大对象用完就置 null,帮助 GC
useEffect(() => {
  const heavyData = parseHugeJson(raw)
  process(heavyData)
  // 函数结束 heavyData 自然超出作用域
}, [])
```

**补充要点：**

1. **能从"渲染/通信/资源"三个维度系统讲优化**，而不是只会说"加 memo"。渲染层关注组件重渲染（React.memo、useCallback、useMemo）和列表虚拟化（FlatList 配置调优）；通信层关注 Bridge 拥堵（Animated.event + useNativeDriver 在原生层处理滚动联动，避免每帧 setState）；资源层关注图片裁剪（按实际显示尺寸请求缩略图）、Bundle 分割（动态 import 减小首屏体积）、Hermes 字节码预编译。

2. **能说出 `useNativeDriver` 的限制**——只支持非布局属性，体现对原生动画系统的理解。`useNativeDriver: true` 只能驱动 `transform`（translateX/Y、scale、rotate）和 `opacity`，因为这些属性的修改不影响布局，原生层可以直接在 UI 线程独立完成。`width`、`height`、`backgroundColor`、`margin` 等会触发布局重算（layout pass），需要 JS 线程的 Yoga 引擎参与，所以无法用原生驱动。需要这些属性的动画应改用 `react-native-reanimated` 的 layout animations 或用 `transform: scale` 近似替代。

3. **知道 Reanimated 的优势**：worklet 机制把更多动画逻辑下沉到原生层，能做手势驱动的复杂动画。Reanimated 的 worklet 是一种特殊的 JS 函数，在编译时被提取出来注册到原生 UI 线程独立执行。与 `useNativeDriver` 只能声明动画曲线不同，worklet 里可以写条件判断、循环、读写多个 SharedValue，实现手势跟随、弹性动画、共享元素转场等 `Animated API` 无法完成的交互。

4. **知道用 React DevTools Profiler 和 Flipper Performance 定位瓶颈**，而不是凭感觉优化。React DevTools Profiler 可以看到每个组件的 render 次数和耗时，快速定位哪些组件在不必要地重渲染；Flipper 的 Performance 插件可以看到 JS 线程和 UI 线程的帧率、Bridge 通信频率、内存占用趋势。优化前先测量，避免盲目加 memo 带来多余的比较开销。

5. **知道 Hermes 的字节码预编译**，能解释为什么它启动快。传统 JS 引擎（JSC/V8）在运行时才解析和编译 JS 源码，这是冷启动的主要耗时。Hermes 在构建阶段就将 JS 源码编译成字节码，打包进 apk/ipa 的是编译产物而非源码，运行时直接加载执行，省掉了 parse + compile 的开销，冷启动速度提升 30-40%。代价是字节码与 Hermes 版本绑定，热更新包需要与原生壳的 Hermes 版本匹配。

**追问：**

Q: FlatList 已经做了虚拟化，为什么列表还是卡？
A: 多数原因不在 FlatList 本身：
- item 没用 `React.memo`，父组件 render 时全量重渲
- `renderItem` 里有内联函数/对象，破坏 memo
- 单个 item 渲染本身就很重（多张图、多个嵌套组件）
- item 高度动态，没法用 `getItemLayout`
- 滚动时频繁 setState 引发联动重渲

Q: `useNativeDriver: true` 为什么无法给 `width` 用？
A: 因为修改 `width` 会触发布局重新计算（layout pass），而布局信息需要在 JS 线程的 Yoga 引擎里计算后回传到原生层。`transform` 和 `opacity` 不影响布局，原生层可以独立修改。要做宽高动画用 Reanimated 的 layout animations 或退而求其次用 transform scale 模拟。

Q: 用 Redux 后界面变卡，怎么办？
A: 通常是 selector 写得太粗：
- 避免在 `useSelector` 里返回新对象（每次都判不等）
- 用 Reselect 做 memoized selector
- selector 粒度细化，组件只订阅自己需要的字段
- 服务端数据用 React Query 替代 Redux，减少 Redux store 体积

## 六、原生模块与桥接

### 如何在 React Native 中调用原生代码？

当 React Native 提供的 API 无法满足需求时（比如调用蓝牙、传感器、第三方原生 SDK），就需要通过**原生模块（Native Modules）**让 JavaScript 调用原生代码。

**什么时候需要写原生模块？**

1. **使用平台特定 API**：比如 iOS 的 HealthKit、Android 的 BiometricPrompt。
2. **集成第三方原生 SDK**：支付、IM、地图、推送等通常只提供原生 SDK。
3. **性能瓶颈优化**：图像处理、加密计算等 CPU 密集任务，原生执行比 JS 快几个数量级。
4. **复用已有原生代码**：团队已有原生组件库，没必要重写。

**如果有现成的社区库，优先用社区库**——原生模块的开发和维护成本远高于 JS 代码，能不写就不写。

#### 1. 调用已有的原生模块

```javascript
import { NativeModules } from 'react-native'

const { CalendarModule } = NativeModules

// 异步调用（旧架构 Bridge 默认是异步）
CalendarModule.createCalendarEvent('Party', 'My House')
  .then(eventId => console.log(`创建成功，ID: ${eventId}`))
  .catch(error => console.error(error))
```

`NativeModules` 是 RN 注入到 JS 全局的原生模块集合,每个原生模块在这里都是一个对象,方法名和原生侧定义一致。

#### 2. 编写一个原生模块（iOS 示例）

**Objective-C 实现：**

代码见 [实践：iOS 原生模块](#practices-ios-module)

**关键宏的作用：**

1. **`RCT_EXPORT_MODULE()`**：把当前类注册为 RN 模块。可以传参数指定 JS 端的名字，不传则默认用类名（去掉 RCT 前缀）。
2. **`RCT_EXPORT_METHOD`**：暴露方法给 JS。注意这是宏，不是普通函数声明，第一个参数特殊（带方法名），其余参数才是实际入参。
3. **`RCTPromiseResolveBlock` / `RCTPromiseRejectBlock`**：标准的 Promise 回调签名，让 JS 端能用 `await` 和 `try/catch`。

#### 3. 编写一个原生模块（Android 示例）

**Java 实现：**

代码见 [实践：Android 原生模块](#practices-android-module)

还需要在 Application 里注册 Package，这里是 Android 平台的样板代码。

#### 4. JS 与原生之间的数据类型映射

跨 Bridge 通信时,数据要序列化成 JSON,所以**只能传可序列化的类型**:

| JS 类型 | iOS (Objective-C) | Android (Java) |
| --- | --- | --- |
| string | NSString | String |
| number | NSNumber | int / float / double |
| boolean | NSNumber (bool) | Boolean |
| Array | NSArray | ReadableArray |
| Object | NSDictionary | ReadableMap |
| null/undefined | nil | null |
| function | RCTResponseSenderBlock | Callback |
| Promise | RCTPromiseResolveBlock / RCTPromiseRejectBlock | Promise |

**无法直接传：** 函数引用（除回调）、循环引用对象、Symbol、Map/Set、Date（要转字符串）、自定义类实例（要序列化成普通对象）。

#### 5. 三种方法返回方式对比

```javascript
// 方式1：Callback（回调函数，老风格）
NativeModules.MyModule.doSomething((result) => {
  console.log(result)
})

// 方式2：Promise（推荐）
const result = await NativeModules.MyModule.doSomething()

// 方式3：Event（原生主动通知 JS）
import { NativeEventEmitter } from 'react-native'
const emitter = new NativeEventEmitter(NativeModules.MyModule)
const sub = emitter.addListener('onUpdate', (data) => { ... })
sub.remove()  // 卸载时清理
```

**三种方式的主要区别**

Callback 和 Promise 实际都属于"请求-响应"模式——JS 先问，原生后答。区别主要在写法和能力边界：

- **Callback**：最原始的方式，一个方法最多接受两个回调（成功/失败），不支持链式调用，无法用 `await`。它的存在主要是历史遗留——早期 RN 没有 Promise 支持，新代码不应使用。
- **Promise**：Callback 的现代替代。支持 `async/await`，配合 `try/catch` 做错误处理更自然。一个方法只能返回一次结果（resolve 或 reject）。适合一次性操作：读取设备信息、调用支付 SDK、请求权限。
- **Event**：唯一的"原生主动推送给 JS"的模式。不遵循请求-响应范式——原生随时可以发事件，JS 随时可以收到。适合持续数据流：位置更新、蓝牙状态变化、推送通知到达、播放器进度回调。

**为什么 Event 无法用 Promise 替代？**

Promise 的语义是"一个请求对应一个结果"。如果原生需要连续推送多次数据（比如每秒推送一次传感器读数），用 Promise 意味着 JS 每秒都要调一次原生方法，每次都走一遍 Bridge 序列化——既浪费又延迟。Event 让原生侧自己控制推送频率，JS 只需注册一次监听，数据沿 Bridge 单向流动，效率高得多。

**选型建议：**

1. **请求-响应模式**：用 Promise（最自然，配合 await/try/catch）。
2. **持续推送模式**（如位置更新、传感器数据、原生事件）：用 Event。
3. **遗留代码兼容**：用 Callback。

#### 6. 新架构下的 TurboModules

新架构（JSI）下原生模块称为 **TurboModule**，主要差异：

```typescript
// 1. 用 TypeScript 定义 Spec
import { TurboModule, TurboModuleRegistry } from 'react-native'

export interface Spec extends TurboModule {
  createCalendarEvent(title: string, location: string): Promise<string>
  getDeviceName(): string  // ✨ 同步方法,旧架构做不到
}

export default TurboModuleRegistry.getEnforcing<Spec>('CalendarModule')
```

```typescript
// 2. Codegen 自动生成 iOS/Android 接口骨架
// 开发者只需在原生侧实现接口
```

**TurboModules 的核心收益：**

1. **同步调用**：可以直接 `const name = MyModule.getDeviceName()`，旧架构需要异步。
2. **按需加载**：模块在第一次调用时才初始化，启动更快。
3. **类型安全**：JS Spec 和原生实现由 Codegen 校对，编译期就能发现签名不匹配。
4. **零序列化**：通过 JSI 直接读写 JS 引擎的内存对象，无 JSON 开销。

**TurboModules 与旧架构 Bridge 的核心对比**

| 维度 | 旧架构 Bridge | 新架构 TurboModules (JSI) |
| --- | --- | --- |
| 通信方式 | 异步消息队列，JSON 序列化 | JSI 直接引用，同步或异步均可 |
| 调用延迟 | 每次跨 Bridge 约 5~10ms（含序列化） | 同步调用 < 0.1ms，接近函数调用 |
| 数据传递 | 全量 JSON 序列化/反序列化 | 直接读写 JS 引擎的 Heap 对象 |
| 模块加载 | 启动时全量初始化所有模块 | 首次访问时按需加载 |
| 类型安全 | 无——方法名拼写错误运行时才暴露 | Spec + Codegen 编译期校验签名一致性 |
| 同步方法 | 不支持——Bridge 队列天然异步 | 支持——JSI 允许原生函数同步返回值 |

**为什么旧架构需要用异步 Bridge？**

旧架构的设计选择是 JS 线程和原生 UI 线程完全隔离，中间用异步消息队列通信。好处是 JS 执行不会阻塞原生 UI 线程，保证了两条线程的流畅性。代价就是所有跨线程调用都需要走序列化→排队→反序列化，天然存在延迟，且无法同步返回结果。

**TurboModules 怎么消除这些开销？**

JSI 让 C++ 代码直接持有 JS 引擎中对象的引用——相当于原生侧拿到了一个指向 JS 堆内存的指针。读写属性不再需要 JSON 中转，也不需要消息队列排队。这也解释了为什么 TurboModules 需要 Spec——因为 JSI 操作内存，类型不匹配会导致引擎崩溃，Spec + Codegen 在编译期就把类型错误拦住了。

#### 7. 实战中的注意事项

```javascript
// ❌ 反面：每次调用都创建 EventEmitter
function MyComponent() {
  useEffect(() => {
    const emitter = new NativeEventEmitter(NativeModules.LocationModule)
    const sub = emitter.addListener('onLocationUpdate', handler)
    // 没清理!
  })
}

// ✅ 正面：卸载时清理订阅
function MyComponent() {
  useEffect(() => {
    const emitter = new NativeEventEmitter(NativeModules.LocationModule)
    const sub = emitter.addListener('onLocationUpdate', handler)
    return () => sub.remove()
  }, [])
}
```

**其他注意事项：**

1. **线程问题**：原生方法默认运行在自己的线程,如果要操作 UI 需要切回主线程(iOS 用 `dispatch_get_main_queue()`,Android 用 `runOnUiThread`)。
2. **错误处理**：`promise.reject` 的第一个参数 code 一定要给,否则 JS 端拿不到错误信息。
3. **频繁通信开销**：旧架构下 Bridge 是异步的，1ms 内调用 1000 次方法会塞满队列。批量传输是常见优化。
4. **iOS 命名转换**：Objective-C 方法 `createEvent:location:` 在 JS 里就是 `createEvent`，第一个冒号前的部分作为 JS 方法名，其余冒号被忽略。

**补充要点：**

1. **能讲清 Bridge 的异步本质和 JSON 序列化开销**，并说明为什么 TurboModule 能消除这些开销。Bridge 通信的完整路径是：JS 调用 → 参数 JSON.stringify → 进入异步消息队列 → 原生侧 JSON.parse → 执行 → 结果 JSON.stringify → 回传队列 → JS 侧 JSON.parse。每一对跨 Bridge 调用至少两次序列化，大量数据传输时（如传递长列表给原生模块）性能下降明显。TurboModule 通过 JSI 直接持有 JS 引擎对象的 C++ 引用，读写属性无需 JSON 中转，同步调用成为可能。

2. **知道 Callback/Promise/Event 分别该用在哪**，不局限于 Promise。Callback 是旧风格的请求-响应，不支持 await，新代码不应使用；Promise 是现代的请求-响应，支持 async/await，适合一次性操作（读取设备信息、调用支付 SDK）；Event 是唯一的原生主动推送模式，适合持续数据流（位置更新、传感器数据、播放进度），JS 只需注册一次监听，原生侧控制推送频率。

3. **知道线程切换的必要性**——这是容易出现的问题（在子线程改 UI 导致崩溃）。RN 的原生方法默认运行在 NativeModules Thread（非主线程），如果方法内需要操作 UI（如弹出 Alert、更新视图），需要切回主线程：iOS 用 `dispatch_async(dispatch_get_main_queue(), ...)`，Android 用 `runOnUiThread(...)`。在子线程操作 UI 在 iOS 上会触发断言崩溃，Android 上会抛出 `CalledFromWrongThreadException`。

4. **能解释 Codegen 的价值**：从手写两边代码到从 Spec 单源生成，类型一致性自动保证。旧架构下 JS 端和原生端的方法签名需要手动保持一致，方法名拼写错误或参数类型不匹配只在运行时才暴露。Codegen 从一份 TypeScript Spec 定义（`interface Spec extends TurboModule`）自动生成 iOS/Android 的接口骨架，编译期就能校验签名一致性，避免了双端手写代码的维护负担。

5. **了解原生 UI 组件（Native Components）和原生模块的区别**：模块是方法调用，组件是渲染节点，两者注册和使用方式不同。Native Module（`RCT_EXPORT_MODULE`）暴露方法供 JS 调用，不参与 UI 渲染，适合功能性操作（文件读写、网络请求、传感器数据）；Native Component（`RCT_EXPORT_VIEW_MODULE`）暴露一个原生视图给 RN 使用，JS 端像普通组件一样使用（`<MyNativeView>`），适合 RN 内置组件无法满足的 UI 需求（地图、视频播放器、图表）。

**追问：**

Q: 写好了原生模块为什么 JS 端 `NativeModules.X` 拿到的是 undefined？
A: 几个可能原因：
- iOS 没在 `RCT_EXPORT_MODULE()` 里指定名字，且类名带特殊前缀
- Android 没在 ReactPackage 的 `createNativeModules` 里返回该模块
- 原生代码改了但没重新 build（JS 改动不需要 build，原生改动需要 build）
- 模块是 Lazy 加载（新架构下）但 Spec 注册有误

Q: 原生模块抛异常 JS 端怎么收到？
A: 需要用 Promise 或 Callback 主动传错误：
- iOS: `reject(@"code", @"message", error)`
- Android: `promise.reject("code", "message", e)`
- 直接抛 Objective-C / Java 异常 JS 端拿不到，会变成"unhandled exception"

Q: 怎么从原生主动调 JS 函数？
A: 通过 EventEmitter 发事件，JS 用 `addListener` 接收。注意：旧架构下原生无法直接同步调用 JS 函数，因为 Bridge 是异步的；新架构 JSI 可以做到。

## 七、常见问题与解决方案

### React Native 中如何处理键盘遮挡输入框的问题？

键盘遮挡是移动端表单的经典难题：用户点击底部的输入框时，弹出的键盘会把输入框完全挡住，看不到自己输入了什么。

**为什么需要专门处理？**

iOS 和 Android 的默认行为不同：

1. **iOS**：键盘弹出时不会自动调整布局，输入框会被键盘完全遮挡。
2. **Android**：可通过 `android:windowSoftInputMode` 配置，但跨页面行为不一致，仍需要 RN 层面统一处理。

#### 方案 1：KeyboardAvoidingView（官方推荐）

```javascript
import { KeyboardAvoidingView, Platform, ScrollView } from 'react-native'

function LoginScreen() {
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      style={{ flex: 1 }}
      keyboardVerticalOffset={Platform.OS === 'ios' ? 64 : 0}  // 顶部 navbar 高度
    >
      <ScrollView contentContainerStyle={{ flexGrow: 1 }}>
        <TextInput placeholder="用户名" />
        <TextInput placeholder="密码" secureTextEntry />
        <Button title="登录" />
      </ScrollView>
    </KeyboardAvoidingView>
  )
}
```

**`behavior` 参数的三种模式：**

| 值 | 行为 | 适用场景 |
| --- | --- | --- |
| `padding` | 给容器底部加 padding，把内容顶上去 | iOS 推荐 |
| `height` | 调整容器高度 | Android（部分场景） |
| `position` | 用 absolute 定位调整 | 复杂布局，少用 |

**为什么 iOS 用 `padding`、Android 用 `height`？**

iOS 的视图层级中,系统不会自动 resize 窗口,用 padding 把内容向上推；Android 上 `windowSoftInputMode=adjustResize` 已经让窗口 resize,padding 会双重叠加导致布局错位,所以用 `height` 让 RN 自己接管。

#### 方案 2：手动监听键盘事件（精细控制）

代码见 [实践：键盘适配](#practices-keyboard)

**适合用在：**

- 聊天页面（输入框需要紧贴键盘）
- 自定义键盘动画
- 需要根据键盘高度联动多个元素

**注意事项：**

1. iOS 用 `keyboardWillShow/Hide`（动画开始时触发），Android 只有 `keyboardDidShow/Hide`（动画结束时触发）。
2. `e.endCoordinates.height` 在 Android 上可能为 0 或不准（部分机型），要做兜底。
3. 卸载时一定要 `remove`，否则会内存泄漏。

#### 方案 3：使用 react-native-keyboard-aware-scroll-view（社区方案）

```javascript
import { KeyboardAwareScrollView } from 'react-native-keyboard-aware-scroll-view'

<KeyboardAwareScrollView
  enableOnAndroid={true}
  extraScrollHeight={20}
  keyboardShouldPersistTaps="handled"
>
  <TextInput placeholder="姓名" />
  <TextInput placeholder="邮箱" />
  <TextInput placeholder="电话" />
</KeyboardAwareScrollView>
```

它会**自动滚动到当前聚焦的输入框**，比 `KeyboardAvoidingView` 更智能，是表单页面的常见选择。代价是引入第三方库 + 更多边界 case。

#### 方案选型决策

| 场景 | 推荐方案 |
| --- | --- |
| 简单登录/注册页 | `KeyboardAvoidingView` + `ScrollView` |
| 多输入框的长表单 | `react-native-keyboard-aware-scroll-view` |
| 聊天/评论框（紧贴键盘） | 手动监听 `keyboardWillShow/Hide` |
| 含原生组件的复杂页面 | 通常需要混合方案 |

#### 容易出现的问题

1. **`keyboardShouldPersistTaps`**：默认点击键盘外区域会先关闭键盘再处理点击，要"点击按钮直接生效"需要设为 `'handled'` 或 `'always'`。
2. **iOS 顶部留白**：`keyboardVerticalOffset` 要算上 SafeArea + Header 高度，否则会有空白。
3. **Modal 内的键盘问题**：Modal 内部的 `KeyboardAvoidingView` 可能不生效（取决于 Modal 实现），通常需要包一层 SafeAreaView。
4. **TextInput 在 Animated.View 里失焦**：动画过程中如果布局变化导致 TextInput 重新挂载，会丢失焦点。要避免在父级用条件渲染。

### 如何实现下拉刷新和上拉加载？

下拉刷新和上拉加载更多是列表页的标配交互。FlatList 内置了完整支持，正确用法如下：

代码见 [实践：下拉刷新与上拉加载](#practices-pull-refresh)

**为什么要这么写？常见问题点：**

1. **需要防重复触发**：`onEndReached` 在快速滚动时可能触发多次，没有 `loadingMore` 锁会发出多次重复请求。
2. **`hasMore` 字段无法少**：没数据后还触发加载会一直 loading，体验差。
3. **`onEndReachedThreshold` 单位是相对值**（0~1），不是像素。0.5 = 距底部还有半屏时触发。
4. **页码用 ref 不用 state**：state 闭包会导致并发请求时拿到旧值，ref 始终读最新。
5. **错误处理要补齐**：网络失败要把 `refreshing/loadingMore` 重置回 false，否则 loading 会卡住。

**自定义下拉效果：**

`RefreshControl` 是原生组件，样式定制有限。要实现"下拉到一半显示自定义动画"这类效果，需要用 `ScrollView` + `onScroll` + `Animated` 自己实现，或者用 `react-native-pull-to-refresh` 这类社区库。

### React Native 如何实现热更新？

热更新（Hot Update / OTA Update）让应用无需走应用商店审核就能更新代码，是 RN 相对原生开发的一大优势，但也是常被滥用、问题最多的功能。

**为什么 RN 能热更新？**

RN 应用结构分两部分：

```
应用包（ipa / apk）
├── 原生壳：原生代码 + RN 框架（不可热更）
└── JS Bundle：业务代码（可热更）
```

启动时优先加载本地 JS Bundle 渲染 UI。如果热更新服务有新版 Bundle，下载到本地、下次启动加载新版本——这就是热更新的全部原理。

**常见用方案：CodePush（微软出品）**

```javascript
import codePush from 'react-native-code-push'

// 配置策略
const codePushOptions = {
  checkFrequency: codePush.CheckFrequency.ON_APP_RESUME,  // 何时检查更新
  installMode: codePush.InstallMode.ON_NEXT_RESTART,      // 何时安装
  mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,    // 强制更新立即生效
}

export default codePush(codePushOptions)(App)
```

**`checkFrequency` 的三种策略：**

| 值 | 含义 | 适用场景 |
| --- | --- | --- |
| `ON_APP_START` | 每次冷启动检查 | 大多数应用 |
| `ON_APP_RESUME` | 每次回到前台检查 | 强时效（金融、IM） |
| `MANUAL` | 手动检查 | 想精细控制（如只在 WiFi 下） |

**`installMode` 的三种策略：**

| 值 | 含义 | 用户感知 |
| --- | --- | --- |
| `ON_NEXT_RESTART` | 下次启动生效 | 对当前使用影响小（推荐） |
| `ON_NEXT_RESUME` | 下次回到前台生效 | 短暂闪屏 |
| `IMMEDIATE` | 立即重启应用 | 应用突然重启（仅强制更新用） |

**手动检查更新示例：**

代码见 [实践：CodePush 手动更新](#practices-codepush)

**热更新的限制（要知道）：**

1. **只能更新 JS 代码 + JS 资源**（图片、字体）。
2. **无法更新原生代码**：新增/修改原生模块、改 AndroidManifest、改 Info.plist 都不行——需要发版。
3. **无法修改应用核心功能**：苹果审核条款明确禁止热更新改变应用主要用途（违反会下架）。
4. **包体积有上限**：CodePush 单个包通常限制在 50MB 内，超出要走分包。

**常见误区：**

```javascript
// ❌ 错误：以为升级了 RN 版本能热更新
// RN 版本升级 = 原生代码变化 = 需要发版

// ❌ 错误：以为新增第三方库能热更新
// 大多数库需要 link 原生代码 = 需要发版

// ❌ 错误：以为修改 app icon、启动图能热更新
// 这些都是原生资源 = 需要发版
```

**热更新工作流：**

```
开发完成 → CI 打包 JS Bundle → 上传 CodePush 服务器
   ↓
按渠道发布（staging / production）
   ↓
按比例灰度（如先发 10% 用户）
   ↓
监控错误率（Sentry 等）
   ↓
全量发布 / 紧急回滚
```

**补充要点：**

1. **理解热更新的边界**：能清晰说出"什么能更"和"什么无法更"。能更新的是 JS Bundle（业务逻辑、JS 资源），无法更新的是原生代码——新增原生模块、修改 AndroidManifest/Info.plist、更换 App Icon 或启动图都需要走应用商店发版。RN 大版本升级（如 0.68 → 0.72）也涉及原生框架代码变化，需要发版，无法热更。

2. **知道灰度发布怎么控风险**：CodePush 支持按 `rollout` 百分比逐步发布，控制风险。例如先发布给 10% 用户，观察 24 小时错误率无异常后再扩到 50%、100%。如果出现问题，CodePush 可以一键回滚到上一个版本，回滚同样是热更新完成，无需重新发版。

3. **能讲苹果审核的合规点**：无法用热更新规避审核（如发版后改成游戏内付费功能），否则会被下架。苹果 App Store Review Guidelines 第 2.5.2 条明确要求：应用在审核后不得改变核心功能和行为。合规的热更新仅限于修复 bug 和优化已有功能，无法添加新的付费入口、改变应用主要用途、绕过审核引入新特性。

4. **知道替代方案**：自研热更新（pushy、阿里云 OTA）、使用 EAS Update（Expo 提供）等，各有取舍。CodePush 依赖微软的全球 CDN，国内访问可能较慢；pushy 是国内社区方案，速度快但生态较小；EAS Update 与 Expo 生态深度绑定，适合 Expo 管理的项目。自建 OTA 服务可以完全控制分发策略和数据安全，但需要自建 CDN 和回滚机制。

5. **知道版本兼容**：热更包需要和原生壳的 RN 版本匹配，跨大版本（如 0.68 → 0.72）需要重新发版。因为 RN 大版本升级会改变 JS 和原生之间的通信协议、组件接口甚至 Hermes 字节码格式，旧壳加载新 Bundle 会导致崩溃。Hermes 字节码尤其敏感——不同 Hermes 版本编译的字节码不兼容，新架构早期这是热更新的常见故障源。

**追问：**

Q: 用户没联网怎么办？
A: 默认行为是用本地缓存的最新版本运行,等下次有网时再检查更新。CodePush 会保留上一个工作版本作为回滚点,新版本启动失败会自动回滚到上个版本。

Q: 热更新会让用户安装恶意代码吗？
A: CodePush 服务端有签名校验，包需要由开发者账号发布。但如果服务端被攻破，按理论说可能推恶意代码——所以企业版应用要加二次签名校验，或者用自建服务器。

Q: 热更新和 React Native 新架构兼容吗？
A: 兼容，但要保证打包工具链版本一致。新架构下 Hermes 字节码格式如果不匹配，热更包加载会失败——这是新架构早期的常见问题，0.72+ 已稳定。

## 八、状态管理

### React Native 中有哪些状态管理方案？

**1. React 内置状态管理**

```javascript
// useState - 组件内部状态
function Counter() {
  const [count, setCount] = useState(0)

  return (
    <View>
      <Text>{count}</Text>
      <Button title="+" onPress={() => setCount(count + 1)} />
    </View>
  )
}

// useReducer - 复杂状态逻辑
function TodoList() {
  const [state, dispatch] = useReducer(reducer, initialState)

  return (
    <View>
      <Button
        title="添加"
        onPress={() => dispatch({ type: 'ADD_TODO', payload: 'New' })}
      />
    </View>
  )
}

// Context - 跨组件状态共享
const ThemeContext = React.createContext()

function App() {
  const [theme, setTheme] = useState('light')

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <HomeScreen />
    </ThemeContext.Provider>
  )
}

function HomeScreen() {
  const { theme } = useContext(ThemeContext)
  return <View style={{ backgroundColor: theme === 'dark' ? '#000' : '#fff' }} />
}
```

**2. Redux**

```javascript
// store.js
import { createStore } from 'redux'
import { Provider } from 'react-redux'

const store = createStore(rootReducer)

function App() {
  return (
    <Provider store={store}>
      <Navigation />
    </Provider>
  )
}

// 使用
import { useSelector, useDispatch } from 'react-redux'

function Counter() {
  const count = useSelector(state => state.count)
  const dispatch = useDispatch()

  return (
    <View>
      <Text>{count}</Text>
      <Button
        title="+"
        onPress={() => dispatch({ type: 'INCREMENT' })}
      />
    </View>
  )
}
```

## 十四、React Native 新架构补充

### React Native 新架构里的 JSI、TurboModules、Fabric 分别是什么？

1. **JSI**
   - JavaScript Interface
   - 让 JS 和原生之间可以更直接通信，不再完全依赖旧 Bridge 的串行消息传递模型

2. **TurboModules**
   - 新一代原生模块体系
   - 更强调按需加载和更高效调用

3. **Fabric**
   - 新一代渲染系统
   - 让 React 和原生 UI 更新链路更统一、更可并发

新架构重要的原因在于：旧 Bridge 模型在高频通信、大型应用和复杂交互场景下，通信成本和调度成本会逐渐暴露出来，而新架构就是在重做这条底层链路。

### Hermes 在 React Native 中解决了什么问题？

Hermes 是 React Native 常用的 JavaScript 引擎。

价值主要在：

1. 更适合移动端资源受限环境
2. 通常能改善启动性能和内存表现
3. 对 RN 场景做了更有针对性的优化

所以它不是所有场景都绝对更强，而是更贴合 React Native 在移动端的运行特征。

### Metro 是什么？它在 React Native 里负责什么？

Metro 是 React Native 默认使用的打包器。

它主要负责：

1. 模块解析
2. 依赖图构建
3. 代码转换
4. 开发期增量打包和 HMR 支持

### 动画里为什么经常提到 `useNativeDriver`？

因为某些动画如果完全跑在 JS 线程上，一旦 JS 线程忙起来，就容易掉帧。

`useNativeDriver` 的意义，就是把部分动画计算和执行尽量下沉到原生侧，减少对 JS 线程实时参与的依赖。

### React Native 为什么容易在长列表和高频手势场景里暴露性能问题？

因为这类场景通常同时具备：

1. 频繁渲染
2. 高频事件
3. 大量布局计算
4. 可能存在 JS 和原生之间高频通信

**3. Redux Toolkit（推荐）**

```javascript
// store.js
import { configureStore, createSlice } from '@reduxjs/toolkit'

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: state => { state.value += 1 },
    decrement: state => { state.value -= 1 }
  }
})

export const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
})

// 使用
function Counter() {
  const count = useSelector(state => state.counter.value)
  const dispatch = useDispatch()

  return (
    <Button
      title="+"
      onPress={() => dispatch(counterSlice.actions.increment())}
    />
  )
}
```

**4. MobX**

```javascript
import { makeAutoObservable } from 'mobx'
import { observer } from 'mobx-react-lite'

class CounterStore {
  count = 0

  constructor() {
    makeAutoObservable(this)
  }

  increment() {
    this.count++
  }
}

const counterStore = new CounterStore()

const Counter = observer(() => {
  return (
    <View>
      <Text>{counterStore.count}</Text>
      <Button title="+" onPress={() => counterStore.increment()} />
    </View>
  )
})
```

**5. Zustand（轻量级）**

```javascript
import create from 'zustand'

const useStore = create(set => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 }))
}))

function Counter() {
  const { count, increment } = useStore()

  return (
    <View>
      <Text>{count}</Text>
      <Button title="+" onPress={increment} />
    </View>
  )
}
```

**6. Recoil**

```javascript
import { atom, useRecoilState } from 'recoil'

const countState = atom({
  key: 'countState',
  default: 0
})

function Counter() {
  const [count, setCount] = useRecoilState(countState)

  return (
    <Button title="+" onPress={() => setCount(count + 1)} />
  )
}
```

**状态管理方案对比：**

| 方案 | 学习成本 | 性能 | 生态 | 适用场景 |
|------|---------|------|------|---------|
| useState/Context | ⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 小型应用 |
| Redux Toolkit | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 中大型应用 |
| MobX | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 复杂状态逻辑 |
| Zustand | ⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | 中小型应用 |
| Recoil | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | 复杂依赖关系 |

### 如何在 React Native 中实现数据持久化？

**1. AsyncStorage（官方推荐）**

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage'

// 存储数据
const storeData = async (key, value) => {
  try {
    await AsyncStorage.setItem(key, JSON.stringify(value))
  } catch (e) {
    console.error('存储失败', e)
  }
}

// 读取数据
const getData = async (key) => {
  try {
    const value = await AsyncStorage.getItem(key)
    return value != null ? JSON.parse(value) : null
  } catch (e) {
    console.error('读取失败', e)
  }
}

// 删除数据
const removeData = async (key) => {
  try {
    await AsyncStorage.removeItem(key)
  } catch (e) {
    console.error('删除失败', e)
  }
}

// 清空所有数据
const clearAll = async () => {
  try {
    await AsyncStorage.clear()
  } catch (e) {
    console.error('清空失败', e)
  }
}
```

**2. MMKV（高性能）**

```javascript
import { MMKV } from 'react-native-mmkv'

const storage = new MMKV()

// 存储
storage.set('user.name', 'John')
storage.set('user.age', 25)
storage.set('user.data', JSON.stringify({ id: 1 }))

// 读取
const name = storage.getString('user.name')
const age = storage.getNumber('user.age')

// 删除
storage.delete('user.name')

// 性能对比：MMKV 比 AsyncStorage 快 30 倍
```

**3. Realm（数据库）**

```javascript
import Realm from 'realm'

// 定义模型
const TaskSchema = {
  name: 'Task',
  properties: {
    _id: 'int',
    name: 'string',
    done: { type: 'bool', default: false }
  },
  primaryKey: '_id'
}

// 打开数据库
const realm = await Realm.open({
  schema: [TaskSchema]
})

// 写入数据
realm.write(() => {
  realm.create('Task', {
    _id: 1,
    name: '学习 React Native',
    done: false
  })
})

// 查询数据
const tasks = realm.objects('Task')
const doneTasks = tasks.filtered('done = true')
```

**4. SQLite**

```javascript
import SQLite from 'react-native-sqlite-storage'

const db = SQLite.openDatabase({ name: 'mydb.db' })

// 创建表
db.transaction(tx => {
  tx.executeSql(
    'CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)'
  )
})

// 插入数据
db.transaction(tx => {
  tx.executeSql('INSERT INTO users (name) VALUES (?)', ['John'])
})

// 查询数据
db.transaction(tx => {
  tx.executeSql('SELECT * FROM users', [], (tx, results) => {
    console.log(results.rows.item(0))
  })
})
```

**持久化方案对比：**

| 方案 | 性能 | 容量 | 复杂度 | 适用场景 |
|------|------|------|--------|---------|
| AsyncStorage | ⭐⭐⭐ | 小 | ⭐ | 简单键值对 |
| MMKV | ⭐⭐⭐⭐⭐ | 中 | ⭐ | 高频读写 |
| Realm | ⭐⭐⭐⭐ | 大 | ⭐⭐⭐ | 复杂数据结构 |
| SQLite | ⭐⭐⭐⭐ | 大 | ⭐⭐⭐⭐ | 关系型数据 |

## 九、调试与测试

### React Native 有哪些调试方法？

**1. 开发者菜单**

```javascript
// 摇一摇设备或按 Cmd+D (iOS) / Cmd+M (Android) 打开菜单

// 常用功能：
// - Reload：重新加载应用
// - Debug：打开调试器
// - Show Inspector：显示元素检查器
// - Show Perf Monitor：显示性能监控
```

**2. Console 调试**

```javascript
console.log('普通日志')
console.warn('警告信息')
console.error('错误信息')
console.table([{ name: 'John', age: 25 }])

// 使用 console.time 测量性能
console.time('fetchData')
await fetchData()
console.timeEnd('fetchData')
```

**3. Flipper 调试工具**

```javascript
// Flipper 是 Facebook 推出的移动应用调试平台

// 功能：
// - 查看网络请求
// - 查看 AsyncStorage 数据
// - 查看 Redux 状态
// - 性能分析
// - 崩溃报告
```

**4. React DevTools**

```javascript
// 安装
npm install -g react-devtools

// 启动
react-devtools

// 功能：
// - 查看组件树
// - 查看 props 和 state
// - 性能分析
// - Hooks 调试
```

**5. 断点调试**

```javascript
// 在代码中添加 debugger
function handlePress() {
  debugger // 程序会在这里暂停
  console.log('继续执行')
}

// 或使用 Chrome DevTools
// 1. 打开开发者菜单
// 2. 选择 "Debug"
// 3. 在 Chrome 中打开 Sources 面板
// 4. 设置断点
```

### React Native 如何进行单元测试？

```javascript
// 使用 Jest + React Native Testing Library

// 安装
npm install --save-dev @testing-library/react-native

// Counter.test.js
import { render, fireEvent } from '@testing-library/react-native'
import Counter from './Counter'

describe('Counter', () => {
  it('应该正确渲染初始值', () => {
    const { getByText } = render(<Counter />)
    expect(getByText('0')).toBeTruthy()
  })

  it('点击按钮应该增加计数', () => {
    const { getByText, getByTestId } = render(<Counter />)
    const button = getByTestId('increment-button')

    fireEvent.press(button)
    expect(getByText('1')).toBeTruthy()
  })
})
```

以上是 React Native 的核心面试题，涵盖了基础概念、组件使用、样式布局、导航路由、性能优化、原生模块、状态管理、调试测试等关键知识点。

## 十、生命周期与 Hooks

### React Native 中的生命周期方法有哪些？

**类组件生命周期（传统方式）：**

```javascript
class MyComponent extends React.Component {
  // 1. 挂载阶段
  constructor(props) {
    super(props)
    this.state = { count: 0 }
    console.log('1. constructor - 组件初始化')
  }

  static getDerivedStateFromProps(props, state) {
    console.log('2. getDerivedStateFromProps - props 变化时调用')
    return null // 返回新的 state 或 null
  }

  componentDidMount() {
    console.log('3. componentDidMount - 组件挂载完成')
    // 适合：数据请求、订阅、定时器
    this.fetchData()
  }

  // 2. 更新阶段
  shouldComponentUpdate(nextProps, nextState) {
    console.log('4. shouldComponentUpdate - 是否需要更新')
    // 返回 false 可以阻止更新，优化性能
    return true
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('5. getSnapshotBeforeUpdate - 更新前获取快照')
    // 返回值会传给 componentDidUpdate
    return null
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('6. componentDidUpdate - 组件更新完成')
    // 适合：根据 props 变化重新请求数据
    if (prevProps.userId !== this.props.userId) {
      this.fetchData()
    }
  }

  // 3. 卸载阶段
  componentWillUnmount() {
    console.log('7. componentWillUnmount - 组件即将卸载')
    // 适合：清理定时器、取消订阅、取消网络请求
    clearInterval(this.timer)
  }

  // 4. 错误处理
  componentDidCatch(error, info) {
    console.log('8. componentDidCatch - 捕获子组件错误')
    this.setState({ hasError: true })
  }

  render() {
    return <View><Text>{this.state.count}</Text></View>
  }
}
```

**函数组件 + Hooks（现代方式）：**

```javascript
function MyComponent({ userId }) {
  const [count, setCount] = useState(0)
  const [data, setData] = useState(null)

  // 相当于 componentDidMount
  useEffect(() => {
    console.log('组件挂载完成')
    fetchData()
  }, []) // 空依赖数组，只执行一次

  // 相当于 componentDidUpdate（监听特定依赖）
  useEffect(() => {
    console.log('userId 变化了')
    fetchData()
  }, [userId]) // 依赖 userId

  // 相当于 componentWillUnmount
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('定时器执行')
    }, 1000)

    // 返回清理函数
    return () => {
      console.log('组件卸载，清理定时器')
      clearInterval(timer)
    }
  }, [])

  // 组合使用：挂载 + 更新 + 卸载
  useEffect(() => {
    console.log('组件挂载或 count 变化')

    return () => {
      console.log('清理上一次的副作用')
    }
  }, [count])

  return <View><Text>{count}</Text></View>
}
```

**生命周期对比表：**

| 类组件生命周期 | Hooks 等价写法 | 用途 |
|--------------|--------------|------|
| constructor | useState | 初始化状态 |
| componentDidMount | useEffect(() => {}, []) | 挂载后执行 |
| componentDidUpdate | useEffect(() => {}, [deps]) | 依赖变化时执行 |
| componentWillUnmount | useEffect 返回函数 | 卸载前清理 |
| shouldComponentUpdate | React.memo | 优化渲染 |
| getDerivedStateFromProps | 直接在渲染中计算 | 从 props 派生 state |

### useEffect 的常见陷阱怎么处理？

**陷阱1：无限循环**

```javascript
// ❌ 错误：会导致无限循环
function BadComponent() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    setCount(count + 1) // 每次更新都会触发 effect
  }) // 没有依赖数组

  return <Text>{count}</Text>
}

// ✅ 正确：添加依赖数组
function GoodComponent() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    // 只在挂载时执行一次
    console.log('mounted')
  }, [])

  return <Text>{count}</Text>
}
```

**陷阱2：依赖数组不完整**

```javascript
// ❌ 错误：缺少依赖
function BadComponent({ userId }) {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetchData(userId) // 使用了 userId 但没有声明依赖
  }, []) // ESLint 会警告

  return <Text>{data}</Text>
}

// ✅ 正确：依赖写全
function GoodComponent({ userId }) {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetchData(userId)
  }, [userId]) // 声明所有依赖

  return <Text>{data}</Text>
}
```

**陷阱3：清理函数使用不当**

```javascript
// ❌ 错误：没有清理订阅
function BadComponent() {
  useEffect(() => {
    const subscription = API.subscribe()
    // 没有清理，会导致内存泄漏
  }, [])
}

// ✅ 正确：返回清理函数
function GoodComponent() {
  useEffect(() => {
    const subscription = API.subscribe()

    return () => {
      subscription.unsubscribe() // 清理订阅
    }
  }, [])
}
```

**陷阱4：在 effect 中使用过期的闭包**

```javascript
// ❌ 错误：使用过期的 count 值
function BadComponent() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count + 1) // count 永远是初始值 0
    }, 1000)

    return () => clearInterval(timer)
  }, []) // 空依赖，count 不会更新
}

// ✅ 正确：使用函数式更新
function GoodComponent() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + 1) // 使用最新的 count
    }, 1000)

    return () => clearInterval(timer)
  }, [])
}
```

**推荐写法：**

```javascript
// 1. 拆分多个 useEffect，每个关注单一职责
function MyComponent({ userId, theme }) {
  // Effect 1: 处理数据获取
  useEffect(() => {
    fetchUserData(userId)
  }, [userId])

  // Effect 2: 处理主题变化
  useEffect(() => {
    applyTheme(theme)
  }, [theme])

  // Effect 3: 处理订阅
  useEffect(() => {
    const subscription = subscribe()
    return () => subscription.unsubscribe()
  }, [])
}

// 2. 使用自定义 Hook 封装复杂逻辑
function useUserData(userId) {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    setLoading(true)
    fetchUserData(userId)
      .then(setData)
      .finally(() => setLoading(false))
  }, [userId])

  return { data, loading }
}

// 使用
function MyComponent({ userId }) {
  const { data, loading } = useUserData(userId)

  if (loading) return <ActivityIndicator />
  return <Text>{data?.name}</Text>
}

// 3. 处理异步操作的取消
function MyComponent({ query }) {
  const [results, setResults] = useState([])

  useEffect(() => {
    let cancelled = false

    async function search() {
      const data = await searchAPI(query)
      if (!cancelled) {
        setResults(data)
      }
    }

    search()

    return () => {
      cancelled = true // 组件卸载时取消更新
    }
  }, [query])
}

// 4. 使用 AbortController 取消网络请求
function MyComponent({ userId }) {
  const [data, setData] = useState(null)

  useEffect(() => {
    const controller = new AbortController()

    fetch(`/api/users/${userId}`, {
      signal: controller.signal
    })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err)
        }
      })

    return () => controller.abort()
  }, [userId])
}
```

### 常用的 React Hooks 有哪些？如何使用？

**1. useState - 状态管理**

```javascript
function Counter() {
  const [count, setCount] = useState(0)

  // 函数式更新
  const increment = () => setCount(c => c + 1)

  // 惰性初始化（只在首次渲染时执行）
  const [data, setData] = useState(() => {
    return expensiveComputation()
  })

  return (
    <View>
      <Text>{count}</Text>
      <Button title="+" onPress={increment} />
    </View>
  )
}
```

**2. useEffect - 副作用处理**

```javascript
function DataFetcher({ userId }) {
  const [data, setData] = useState(null)

  useEffect(() => {
    // 副作用：数据获取
    fetchData(userId).then(setData)

    // 清理函数
    return () => {
      // 取消请求、清理订阅等
    }
  }, [userId]) // 依赖数组

  return <Text>{data?.name}</Text>
}
```

**3. useContext - 上下文**

```javascript
const ThemeContext = React.createContext()

function App() {
  const [theme, setTheme] = useState('light')

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <HomeScreen />
    </ThemeContext.Provider>
  )
}

function HomeScreen() {
  const { theme, setTheme } = useContext(ThemeContext)

  return (
    <View style={{ backgroundColor: theme === 'dark' ? '#000' : '#fff' }}>
      <Button title="切换主题" onPress={() => setTheme(t => t === 'dark' ? 'light' : 'dark')} />
    </View>
  )
}
```

**4. useReducer - 复杂状态逻辑**

```javascript
const initialState = { count: 0, step: 1 }

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step }
    case 'decrement':
      return { ...state, count: state.count - state.step }
    case 'setStep':
      return { ...state, step: action.payload }
    case 'reset':
      return initialState
    default:
      throw new Error()
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState)

  return (
    <View>
      <Text>Count: {state.count}</Text>
      <Text>Step: {state.step}</Text>
      <Button title="+" onPress={() => dispatch({ type: 'increment' })} />
      <Button title="-" onPress={() => dispatch({ type: 'decrement' })} />
      <Button title="Reset" onPress={() => dispatch({ type: 'reset' })} />
    </View>
  )
}
```

**5. useCallback - 缓存函数**

```javascript
function Parent() {
  const [count, setCount] = useState(0)
  const [text, setText] = useState('')

  // ❌ 每次渲染都创建新函数，导致子组件重渲染
  const handleClick = () => {
    console.log('clicked')
  }

  // ✅ 使用 useCallback 缓存函数
  const handleClickMemo = useCallback(() => {
    console.log('clicked')
  }, []) // 依赖为空，函数永远不变

  return (
    <View>
      <Text>{count}</Text>
      <TextInput value={text} onChangeText={setText} />
      <Child onClick={handleClickMemo} />
    </View>
  )
}

const Child = React.memo(({ onClick }) => {
  console.log('Child rendered')
  return <Button title="Click" onPress={onClick} />
})
```

**6. useMemo - 缓存计算结果**

```javascript
function ExpensiveComponent({ items, filter }) {
  // ❌ 每次渲染都重新计算
  const filteredItems = items.filter(item => item.includes(filter))

  // ✅ 使用 useMemo 缓存计算结果
  const filteredItemsMemo = useMemo(() => {
    console.log('重新计算')
    return items.filter(item => item.includes(filter))
  }, [items, filter]) // 只在依赖变化时重新计算

  return (
    <FlatList
      data={filteredItemsMemo}
      renderItem={({ item }) => <Text>{item}</Text>}
    />
  )
}
```

**7. useRef - 引用值**

```javascript
function TextInputWithFocus() {
  const inputRef = useRef(null)
  const countRef = useRef(0)

  useEffect(() => {
    // 自动聚焦
    inputRef.current?.focus()
  }, [])

  const handlePress = () => {
    // useRef 的值变化不会触发重渲染
    countRef.current += 1
    console.log('点击次数:', countRef.current)
  }

  return (
    <View>
      <TextInput ref={inputRef} />
      <Button title="点击" onPress={handlePress} />
    </View>
  )
}
```

**8. useLayoutEffect - 同步副作用**

```javascript
function MeasureComponent() {
  const [height, setHeight] = useState(0)
  const ref = useRef(null)

  // useLayoutEffect 在 DOM 更新后、浏览器绘制前同步执行
  useLayoutEffect(() => {
    if (ref.current) {
      ref.current.measure((x, y, width, height) => {
        setHeight(height)
      })
    }
  }, [])

  return (
    <View ref={ref}>
      <Text>高度: {height}</Text>
    </View>
  )
}
```

**9. 自定义 Hook**

```javascript
// 自定义 Hook: 网络状态
function useNetworkStatus() {
  const [isOnline, setIsOnline] = useState(true)

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      setIsOnline(state.isConnected)
    })

    return () => unsubscribe()
  }, [])

  return isOnline
}

// 使用
function MyComponent() {
  const isOnline = useNetworkStatus()

  return (
    <View>
      <Text>{isOnline ? '在线' : '离线'}</Text>
    </View>
  )
}

// 自定义 Hook: 防抖
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}

// 使用
function SearchComponent() {
  const [searchText, setSearchText] = useState('')
  const debouncedSearchText = useDebounce(searchText, 500)

  useEffect(() => {
    if (debouncedSearchText) {
      // 执行搜索
      search(debouncedSearchText)
    }
  }, [debouncedSearchText])

  return (
    <TextInput
      value={searchText}
      onChangeText={setSearchText}
      placeholder="搜索..."
    />
  )
}
```

**Hooks 使用规则：**

1. **只在顶层调用 Hooks**：避免在循环、条件或嵌套函数中调用
2. **只在 React 函数中调用 Hooks**：函数组件或自定义 Hook
3. **自定义 Hook 需要以 use 开头**：如 useCustomHook

```javascript
// ❌ 错误：在条件中使用 Hook
function BadComponent({ condition }) {
  if (condition) {
    const [count, setCount] = useState(0) // 违反规则
  }
}

// ✅ 正确：在顶层使用 Hook
function GoodComponent({ condition }) {
  const [count, setCount] = useState(0)

  if (condition) {
    // 使用 count
  }
}
```

## 十一、高级主题

### React Native 中如何处理深层链接（Deep Linking）？

**深层链接（Deep Linking）** 是指通过 URL 直接打开应用内的某个页面或功能，而不是从应用首页一路点进去。它是移动应用中"可被分享、可被检索、可被外部唤起"的关键能力。

**为什么深层链接重要？**

1. **从浏览器/微信跳转到 App**：用户点击一个链接，能进入对应的商品详情页，而不是 App 首页。
2. **App 之间互相唤起**：第三方 SDK（支付、登录、分享）回调到自己的 App。
3. **通知点击跳转**：推送通知点击后跳转到对应页面，而不是只能打开 App 首页。
4. **SEO / ASO**：搜索引擎收录的 App 内容，能直接深链到对应页面。

**两种方案对比：**

| 类型 | URL 形式 | 优势 | 劣势 |
| --- | --- | --- | --- |
| **URL Scheme** | `myapp://product/123` | 简单、跨平台一致 | 任何 App 都能注册，可能被劫持 |
| **Universal Links（iOS）/ App Links（Android）** | `https://example.com/product/123` | 安全（域名验证）、未装 App 时可降级到网页 | 配置复杂（需服务端 + 应用商店签名） |

**实战代码：**

代码见 [实践：Deep Linking](#practices-deeplink)

**为什么需要分两种场景处理？**

1. **冷启动**：App 进程不存在,系统通过 URL 启动 App,RN 启动后用 `getInitialURL()` 拿到这个 URL。
2. **热启动**：App 在后台,系统不会重新启动 RN,而是触发 `url` 事件——这时只能靠 `addEventListener` 接收。

如果只处理一种场景，会出现"杀死 App 后再点击链接没反应"或"App 在后台时点击链接打不开正确页面"的问题。

#### 配合 React Navigation 的官方支持

代码见 [实践：Navigation Linking 配置](#practices-linking-config)

**优势：** React Navigation 自动处理 URL 解析、参数提取、路由跳转，比手写更稳。更常见的是用这种声明式配置。

**配置 URL Scheme 的原生步骤：**

1. **iOS**：`Info.plist` 加 `CFBundleURLTypes`，指定 scheme。
2. **Android**：`AndroidManifest.xml` 加 `intent-filter`，指定 `action` 和 `data`。

这里一次配置好即可，更多细节看官方文档即可。

### React Native 中如何实现推送通知？

推送通知是 App 维持用户活跃度的核心手段。RN 中实现推送通常依赖**第三方服务**（FCM、APNs、极光、个推等），而不是自己搭服务器。

**推送的两种类型：**

1. **本地通知（Local Notification）**：App 自己触发，比如定时提醒。
2. **远程通知（Remote / Push Notification）**：服务端推送，需要经过 APNs（iOS）或 FCM（Android）通道。

**为什么需要经过 APNs/FCM？**

操作系统出于功耗考虑，不允许 App 在后台保持长连接。所有 App 共用 APNs/FCM 一条系统级长连接，由系统统一处理推送，App 可以完全不在前台运行也能收到。

**主流方案：Firebase Cloud Messaging（FCM）+ APNs**

代码见 [实践：推送通知](#practices-push)

**为什么前台收到推送系统不自动展示？**

系统认为用户已经在用 App 了，不需要再用通知打扰。RN 开发者要自己决定怎么展示——通常是 in-app 横幅（顶部下拉的一条 toast），点击后跳转到对应页面。

**推送的主要难点：**

1. **iOS 通知权限**：用户拒绝后只能引导去设置里手动开，无法二次弹窗。
2. **Token 管理**：Token 会变（重装、设备恢复），需要监听 `onTokenRefresh` 并同步到服务端，否则推送无效。
3. **静默推送**：iOS 用 `content-available: 1`，Android 用 data-only message，可以在后台触发代码执行（限制不少，如 iOS 1 小时只允许几次）。
4. **角标管理**：iOS 角标数字需要 App 自己维护并主动设置，FCM 不会自动加。
5. **去重和点击统计**：服务端推送时带上 messageId，App 端做去重和上报。

**国内的特殊问题：**

国内 Android 厂商（华为、小米、OPPO、vivo）有自己的推送通道，FCM 在国内不可用：

1. **后台保活机制不同**：各厂商对后台进程的杀进策略不同，靠 FCM 一旦进程被杀就收不到推送。
2. **解决方案**：接入厂商通道（华为 HMS Push、小米 Push、OPPO Push、vivo Push）+ 离线 Push 兜底（极光、个推、友盟）。
3. **统一封装**：用极光、个推、友盟 U-Push 这类聚合方案，一套代码对接多个厂商通道。

### React Native 中如何处理应用权限？

RN 应用经常需要使用相机、位置、通讯录、麦克风等敏感能力，这些都需要**运行时权限**（Runtime Permission）。

**为什么需要运行时权限？**

iOS 6 起、Android 6.0 起，敏感权限不再是安装时一次性授予，而是需要在使用时弹窗向用户请求。这是隐私保护的硬要求，处理不好直接导致功能不可用。

代码见 [实践：权限请求](#practices-permission)

**为什么用 `react-native-permissions` 而不是官方 `PermissionsAndroid`？**

1. **跨平台一致**：官方 `PermissionsAndroid` 只能用在 Android，iOS 要走另一套 API。
2. **状态更细**：能区分"用户拒绝"和"用户选了不再询问"，业务逻辑可以分别处理。
3. **覆盖所有平台权限**：包括相机、位置、通讯录、日历、健康等几十种。

**权限请求建议：**

```javascript
// ❌ 反面：在 App 启动时集中弹出多个权限
useEffect(() => {
  requestCameraPermission()
  requestLocationPermission()
  requestContactsPermission()
  // 用户可能直接卸载
}, [])

// ✅ 正面：用到时再请求,并先做"前置说明"
async function takePhoto() {
  // 第一次用相机时先弹个温和的解释
  if (!hasShownExplanation) {
    Alert.alert(
      '使用相机',
      '我们需要相机权限来拍摄商品照片',
      [
        { text: '取消' },
        {
          text: '继续',
          onPress: async () => {
            const granted = await requestCameraPermission()
            if (granted) openCamera()
          }
        }
      ]
    )
  } else {
    openCamera()
  }
}
```

**iOS 需要配置的 Info.plist 字段：**

iOS 不光要请求权限,还要在 `Info.plist` 写**用途说明**(`Usage Description`),比如:

| Key | 说明 |
| --- | --- |
| `NSCameraUsageDescription` | 相机用途 |
| `NSPhotoLibraryUsageDescription` | 相册用途 |
| `NSLocationWhenInUseUsageDescription` | 使用期间定位 |
| `NSMicrophoneUsageDescription` | 麦克风用途 |

不写或写得过于简略会导致**应用商店审核被拒**。说明文案要诚实、具体——"获取您的位置以推荐附近商家"比"获取位置"通过率高得多。

### React Native 中如何实现国际化（i18n）？

国际化是国际化产品的基础需求,涉及**文案翻译、日期格式、货币、复数变化、RTL 布局**等多个方面,远不止"翻译几个字符串"。

**为什么需要专门的 i18n 方案？**

```javascript
// ❌ 反面：硬编码 + 简单 if-else
function Welcome() {
  const lang = getLang()
  return <Text>{lang === 'zh' ? '欢迎' : 'Welcome'}</Text>
}

// 问题：
// 1. 文案散落在各组件,无法集中管理和翻译
// 2. 增加新语言要改所有组件
// 3. 复数、占位符、日期格式都处理不了
// 4. 翻译人员看不到完整上下文
```

#### 主流方案：i18next + react-i18next

代码见 [实践：国际化配置](#practices-i18n)

**i18next 的核心能力：**

1. **占位符插值**：`{{name}}` 自动替换为实际值。
2. **复数变化**：英文 1 item / 2 items，俄语有三种复数形态——i18next 内置 CLDR 复数规则。
3. **嵌套和命名空间**：大型项目按模块拆 namespace（如 `common.json` / `product.json`）。
4. **延迟加载**：按需加载语言包，减少初始 bundle 体积。

#### 自动检测系统语言

```javascript
import * as RNLocalize from 'react-native-localize'

const locales = RNLocalize.getLocales()
// [{ languageCode: 'zh', countryCode: 'CN', languageTag: 'zh-CN', isRTL: false }]

const fallback = { languageTag: 'en' }
const { languageTag } = RNLocalize.findBestAvailableLanguage(
  ['en', 'zh', 'ja', 'fr'],
  fallback.languageTag
) || fallback

i18n.changeLanguage(languageTag)
```

**为什么不直接用 `Intl.DateTimeFormat`？**

RN 的 JS 引擎默认不带ICU 数据(Hermes 旧版本完全不支持 Intl,iOS 的 JSC 支持但缺部分 locale)。`react-native-localize` 走原生 API,能拿到准确的系统设置(包括用户在系统设置里手动改过的偏好)。

#### RTL 布局支持（阿拉伯语、希伯来语）

```javascript
import { I18nManager } from 'react-native'

// 启用 RTL 后,所有 marginLeft 自动变 marginRight,flexDirection: 'row' 自动反向
if (RNLocalize.isRTL) {
  I18nManager.forceRTL(true)
} else {
  I18nManager.forceRTL(false)
}

// 切换 RTL 后需要重启 App 才能生效
RNRestart.Restart()
```

**为什么 RTL 需要重启 App？**

`I18nManager.forceRTL` 是**全局静态配置**，影响整个布局系统。已经渲染的组件不会动态翻转，只有重新启动后所有组件按新方向布局才正确。

**RTL 开发的注意事项：**

1. **避免 `left` / `right`**：用 `start` / `end` 代替，自动适配方向。
2. **图标方向**：箭头、返回按钮等方向性图标需要镜像或换图。
3. **数字和日期**：数字仍是从左到右（除非用阿拉伯-印度数字）。

**补充要点：**

1. **理解 i18n 不只是翻译**：复数、日期、货币、RTL 都是主要问题。英文有单复数变化（1 item / 2 items），俄语和阿拉伯语有三四种复数形态，i18next 内置 CLDR 复数规则自动处理；日期格式因地区而异（MM/DD/YYYY vs DD/MM/YYYY），需要 Intl 或原生 API 配合；货币符号和位置不同（$10 vs 10€）；RTL 语言（阿拉伯语、希伯来语）需要整个布局方向翻转，RN 的 `I18nManager.forceRTL` 可以自动将 `marginLeft` 转为 `marginRight`，但切换后需要重启 App。

2. **能讲清动态切换语言的实现**：监听 `i18n.changeLanguage`，配合 React 重渲染。调用 `i18n.changeLanguage('en')` 后，所有使用 `useTranslation()` 的组件会自动重渲染，因为 i18next 内部维护了一个语言状态的发布-订阅机制。注意：切换语言后组件的文本会更新，但 StyleSheet.create 中的静态值不会变化，语言相关的样式差异需要在组件内部通过 `useMemo` 动态计算。

3. **了解服务端翻译方案**：大型应用文案通常存在 CMS，运行时动态加载,而不是打包进 bundle。打包所有语言到 JS Bundle 会显著增加包体积（每多一种语言增加数十 KB），且每次文案修改都需要发版。服务端方案（如 Crowdin + API 拉取）可以在 App 启动时下载当前语言包，文案更新无需热更或发版，但需要做好缓存和降级策略（网络不通时用本地兜底文案）。

4. **知道翻译协作流程**：用 Crowdin、Lokalise 这类平台让翻译人员在线协作，导出 JSON 再集成到代码。开发者在代码中用 key 引用文案（`t('welcome')`），提取所有 key 上传到协作平台，翻译人员在平台上看到上下文截图和备注后翻译，平台导出 JSON 文件直接替换项目中的语言资源文件。整个流程可以接入 CI 自动化：代码合并后自动提取新 key 上传，翻译完成后自动拉取并生成 PR。

5. **了解 ICU MessageFormat**：i18next 默认语法的扩展，能表达更复杂的语言变化（如选择不同语句）。ICU MessageFormat 支持 `select`（根据性别等选择不同措辞：`{gender, select, male{他} female{她} other{其}}`）、`plural`（比默认复数更精细的控制）和嵌套变量，是处理斯拉夫语系、闪米特语系等复杂语法变化的行业标准。

## 十二、实战场景题

### 如何优化包含 10000 条数据的列表？

10000 条数据的列表是 RN 性能优化的高频问题。一个未优化的实现可以让 App 直接阻塞或崩溃，处理方式需要从**渲染策略、数据加载、组件设计**三个层面同时入手。

**问题在哪？**

1. **首屏渲染压力**：默认情况下 FlatList 会挂载 21 个屏幕高度的内容，对应可能 200~500 个 item，初始渲染就要几秒。
2. **滚动卡顿**：item 一旦渲染开销大（如多张图片），滑动时新 item 进入可视区会触发卡顿。
3. **内存占用**：item 不及时回收，10000 项全保留会很快 OOM。
4. **重渲染**：父组件状态变化触发整个列表重新 reconcile。

#### 优化方案

代码见 [实践：万级数据列表优化](#practices-biglist)

**每个优化点的收益对比（10000 项实测，仅供参考）：**

| 优化项 | 不优化 | 优化后 | 提升 |
| --- | --- | --- | --- |
| 首屏渲染时间 | 3.5 秒 | 0.4 秒 | 8.7x |
| 滚动 FPS | 25-30 | 55-60 | 2x |
| 内存占用 | 250+ MB | 80 MB | 3x |
| 滚动到底部时间 | 卡顿明显 | 流畅 | - |

#### 进阶方案：分页加载 + 服务端排序

实际生产中**绝不应该**一次性加载 10000 条到客户端：

```javascript
// ✅ 正确:分页 + 无限滚动
const [data, setData] = useState([])
const [page, setPage] = useState(1)

const loadMore = async () => {
  const next = await api.fetchList({ page, pageSize: 20 })
  setData(prev => [...prev, ...next])
  setPage(p => p + 1)
}
```

10000 条数据的真实业务一般是搜索/排行榜——绝大多数用户只看前 100 条，剩余数据通常不会滚到。**更合理的优化是不提前加载**。

#### 终极方案：用 RecyclerListView 或 FlashList

```javascript
import { FlashList } from '@shopify/flash-list'

<FlashList
  data={data}
  renderItem={renderItem}
  estimatedItemSize={80}  // 估算高度,自动优化
/>
```

`FlashList`（Shopify 出品）和 `RecyclerListView`（Flipkart 出品）都基于**更接近 RecyclerView 模型**——和原生 RecyclerView 一样池化复用 cell，比 FlatList 快几倍。它们更适合：

1. 数据量极大（10万+）
2. item 高度差异大（聊天列表）
3. 需要瀑布流、网格混合布局

代价是 API 略复杂、第三方依赖。

**补充要点：**

1. **能讲清虚拟化的原理**：只渲染可见区域 + 缓冲区，复用 cell。FlatList 维护一个虚拟窗口（由 `windowSize` 控制），窗口内的 item 被挂载渲染，窗口外的被卸载回收。当用户滚动时，离开窗口的 item 的组件实例被放入回收池，新进入窗口的 item 优先从回收池取出复用而非重新创建，因此实际挂载的组件数量始终有限（通常 15-30 个）。FlashList/RecyclerListView 基于这个思路更再往下，采用原生 RecyclerView 的 cell pooling 机制，cell 不是卸载而是detach，复用时只需更新数据而非重新挂载，效率更高。

2. **知道 `getItemLayout` 的限制和价值**：需要定高，但能跳过测量阶段。FlatList 默认需要通过 `onLayout` 回调异步测量每个 item 的实际高度，才能计算滚动位置和虚拟化窗口。这个过程是异步的且逐项进行，长列表初始渲染和快速滚动时会成为瓶颈。`getItemLayout` 提前告知每个 item 的精确高度和偏移量，FlatList 跳过测量步骤，同步计算出所有位置信息。限制是所有 item 需要是固定高度（或高度可由 index 和 data 预计算），一旦有动态高度（如展开/收起、多行文本自适应）就无法使用。

3. **能区分 React 重渲染和原生重排**：memo 优化的是 React reconcile，不是原生 layout。`React.memo` 阻止的是 JS 线程中的虚拟 DOM diff 和组件重渲染，减少的是 React 侧的 JS 计算开销；而原生重排（layout pass）发生在 UI 线程，当原生视图树的布局属性变化时触发（如某个 View 的 frame 改变导致兄弟节点重新排列）。两者在不同线程、不同层面，memo 无法减少原生重排——如果一个组件的 `height` 在动画中频繁变化，即使 memo 了该组件，原生层仍然会频繁触发布局计算。

4. **了解 FlashList 的实现思路**：cell pooling + recycling，是更接近"原生级"列表。FlashList（Shopify 出品）借鉴了原生 RecyclerView 的回收策略：item 不是被卸载（unmount）而是被 detach（从视图树移除但保留组件实例），新 item 进入时从池中取出已有实例更新 props 即可，省掉了 React 的 mount/unmount 开销。配合 `estimatedItemSize` 提供估算高度，FlashList 的首次渲染和滚动流畅度都优于 FlatList，在 10 万级数据的场景下差距更明显。

5. **从产品角度确认需求**：是否真的需要一次性展示 10000 条数据。多数情况下这是设计问题，不是技术问题。用户很少会滚动到 1000 条之后，浏览集中在首屏和前几页。更合理的做法是分页加载（每页 20-50 条 + 无限滚动），服务端只返回用户实际需要的数据，客户端内存占用始终可控。技术优化用于兜底，产品设计才是根本。

### 如何实现一个可拖拽排序的列表？

可拖拽排序是中等难度的实战题，考察对**手势、动画、状态管理**的综合掌握。

**为什么这个需求难？**

1. **手势识别**：需要区分"点击"和"长按拖拽"。
2. **动画跟手**：拖拽时 item 需要实时跟随手指，60fps 无法掉帧。
3. **占位计算**：手指划过时其他 item 要让位，且让位有过渡动画。
4. **数据更新**：拖拽结束后要更新数组顺序，且新顺序的渲染无法闪一下。

#### 常见方案：用现成库 react-native-draggable-flatlist

代码见 [实践：拖拽排序列表](#practices-draggable)

**这个库的工作原理（理解即可，不需要自行实现）：**

1. 内部基于 `Animated.View` 包装每个 item，监听 `PanGestureHandler`。
2. 长按触发后,把当前 item 的 `position` 改为 `absolute`，跟手指 translate。
3. 用 `runOnJS` 把当前 hover 的 index 通知 JS 层，重排其他 item 的 transform。
4. 释放手指时计算最终 index，更新数据并复位动画。

#### 自己实现的做法

如果继续往下拆到“不让用库怎么写”，可以讲思路：

代码见 [实践：手势拖拽组件](#practices-gesture-drag)

**关键技术点：**

1. **`react-native-gesture-handler`**：替代 RN 内置 PanResponder,手势在原生层处理,不会被 JS 卡顿打断。
2. **`react-native-reanimated`**：用 worklet 让动画运行在 UI 线程,跟手 60fps。`useSharedValue` 的修改不触发 React 渲染。
3. **`runOnJS`**：从 worklet 切回 JS 线程更新数据(因为 setState 无法在 worklet 里调用)。
4. **`activateAfterLongPress`**：长按激活,避免和列表滚动手势冲突——这是容易出现的问题。

**手势冲突的处理：**

```
列表手势(竖向滚动) + 拖拽手势(竖向移动单项) → 同方向冲突
```

解决方案：

1. **长按阈值**：要求按住 300ms 才激活拖拽,短按时手势直接交给 ScrollView 处理。
2. **激活区域**：只在 item 的拖拽 handle（一个图标）上响应长按，其他区域允许滚动。
3. **拖拽时禁用滚动**：拖拽激活后调用 `setNativeProps({ scrollEnabled: false })`，避免双重响应。

**补充要点：**

1. **能区分 PanResponder 和 react-native-gesture-handler**：前者跑在 JS 线程容易卡，后者跑在原生层平滑。PanResponder 是 RN 内置的手势系统，所有触摸事件都需要通过 Bridge 传到 JS 线程进行识别和判断，一旦 JS 线程忙于其他任务（列表渲染、数据处理），手势响应会延迟 50-100ms，导致拖拽卡顿。`react-native-gesture-handler` 将手势识别下沉到原生层执行（iOS 的 UIGestureRecognizer、Android 的 GestureDetector），识别结果才通知 JS，因此即使 JS 线程繁忙，手势仍然流畅跟手。

2. **理解 worklet**：Reanimated 的核心，把 JS 函数编译成在 UI 线程独立运行的代码块。worklet 是一种特殊的 JS 函数，通过 `'worklet'` 指令标记，Reanimated 在编译时将其提取并注册到原生 UI 线程。运行时 worklet 直接在 UI 线程执行，读写 `useSharedValue` 创建的共享变量，完全绕过 Bridge 和 JS 线程。也就是动画和手势的每帧计算都不受 JS 线程阻塞影响，可以稳定保持 60fps。

3. **能讲手势冲突的解决思路**：长按、激活区、动态禁用其他手势，是真实业务里的高频问题。拖拽和列表滚动都是竖向手势，同方向会冲突。解决方式有三种：长按阈值（`activateAfterLongPress(300)` 让 300ms 内的触摸交给滚动处理）、激活区域限制（只在拖拽 handle 图标上监听长按，其他区域正常滚动）、动态禁用（拖拽激活后 `setNativeProps({ scrollEnabled: false })` 禁用列表滚动，释放后恢复）。

4. **知道性能边界**：item 数量大时(1000+),拖拽时所有 item 都重算 transform 会卡顿——需要只动 hover 区域附近的 item。拖拽排序需要给 hover 位置附近的 item 添加偏移动画，如果列表有 1000 项且每项都响应 offset 变化，就有 1000 个 SharedValue 同时更新，UI 线程也会承受较大压力。优化时通常只对 hover 位置前后 N 个 item（如 ±5）做偏移动画，其他 item 保持不动，将动画计算量从 O(n) 降到 O(1)。

5. **补充无障碍方案**：拖拽排序对屏幕阅读器用户支持不足,需要提供"上移/下移"按钮作为替代方案。拖拽依赖精细的手指操作和视觉反馈，屏幕阅读器用户无法完成。更稳的做法是为每个 item 添加 `accessibilityActions`（`up`/`down`），通过 `onAccessibilityAction` 回调执行与拖拽相同的数据重排逻辑，保证所有用户都能使用排序功能。

**追问：**

Q: 为什么不用 PanResponder？
A: PanResponder 是 RN 内置的旧手势系统，所有手势事件都过 Bridge 到 JS 层处理。一旦 JS 线程忙（比如列表渲染），手势响应会延迟，体验差。`react-native-gesture-handler` 把手势识别下沉到原生层，识别完才通知 JS，性能远胜。

Q: Reanimated 和 Animated API 有什么区别？
A: Animated 默认运行在 JS 线程,只有 `useNativeDriver: true` 且属性允许时才能下沉到原生层。Reanimated 的 worklet 默认就在 UI 线程跑,且支持复杂逻辑（条件、循环、读写多个 SharedValue）。可以理解为：Animated 偏向"声明动画曲线",Reanimated 偏向"在 UI 线程写代码"。

Q: 拖拽过程中其他 item 怎么让位？
A: 主流做法是给每个 item 一个动画 offset：手指 hover 到的 item 索引以下的所有 item 的 `translateY` 设为 +ITEM_HEIGHT 或 -ITEM_HEIGHT，配合 `withSpring` 做平滑过渡。这里逻辑在 worklet 里直接计算，效率很高。

### 如何实现一个聊天界面（含上拉加载历史 + 下拉到底自动滚动）？

聊天界面是综合考察题，涉及**反向列表、键盘联动、性能优化、消息插入位置**等多个点。

**主要难点：**

1. **新消息追加在底部**：但用户视角看是"列表往上滚",和普通列表相反。
2. **历史消息加载在顶部**：上拉触底时反向加载,无法让滚动位置跳变。
3. **键盘弹起时输入框上移**，且最新消息可见。
4. **图片消息高度未知**，需要有占位机制避免布局抖动。

代码见 [实践：聊天界面](#practices-chat)

**为什么用 `inverted={true}`？**

聊天列表的视觉是"最新消息在底部，历史消息在上方"，但 FlatList 默认是"index 0 在顶部"。如果不反转，要么数组顺序反着存（处理麻烦），要么手动 `scrollToEnd`（频繁触发）。`inverted` 一行解决：

1. 数组 index 0 自动渲染到屏幕底部
2. 滚动方向反转，往上滑 = 往新消息滚动
3. `onEndReached` 触发条件变成"滚到列表顶部"，正好对应"加载历史"

**`maintainVisibleContentPosition` 的作用：**

加载历史消息时，新消息会被插入数组末尾（即列表上方）。如果不处理，FlatList 会以"列表顶部"为锚点，导致用户当前看的消息整体下滑——体验体验较差。这个属性告诉 FlatList："以 index 0 这条消息为锚点，新插入的内容把上方撑开，当前可见区域不动"。

**补充要点：**

1. **`inverted` 和 `maintainVisibleContentPosition` 是聊天列表的关键配置**。`inverted={true}` 让 FlatList 反转渲染方向：数组 index 0 渲染在屏幕底部，新消息通过 `unshift` 插入数组开头即自动出现在底部，`onEndReached` 的触发位置变为列表顶部（对应"加载更多历史"）。`maintainVisibleContentPosition` 解决加载历史时的跳动问题：新历史消息插入数组末尾（列表顶部视觉位置），FlatList 默认以顶部为锚点导致当前可见消息整体下滑；设置此属性后 FlatList 以当前可见的第一条消息为锚点，新内容向上撑开，用户正在阅读的位置不动。

2. **图片消息预先占位**：消息体里带 `width/height`，占位 `View` 撑高，图片加载完不引起布局抖动。如果图片消息没有预知尺寸，FlatList 在图片加载完成前无法正确计算 item 高度，加载后高度突变会导致：该 item 下方的所有消息位置跳变、虚拟化窗口重新计算、用户正在阅读的消息突然移位。解决方案是消息体中携带图片的原始宽高，渲染时按容器宽度等比缩放计算占位高度，图片加载完平滑替换，布局零抖动。

3. **未读消息分隔线**：在数组里插入虚拟的"未读起点"项，进入聊天时滚到这个位置。实现方式是在消息数组中插入一条 type 为 `'unreadDivider'` 的虚拟项，记录未读消息的起始 ID。进入聊天页面时，通过 FlatList 的 `scrollToIndex` 或 `scrollToOffset` 滚动到该虚拟项的位置，用户一眼就能看到未读消息的起点。

4. **WebSocket 状态管理**：网络断开重连后要补拉缺失消息（带 lastMessageId），防止丢消息。WebSocket 断连期间服务端可能产生了新消息，重连后如果只恢复监听，断连期间的消息就会丢失。一般做法是重连时带上最后一条消息的 ID（`lastMessageId`），服务端返回该 ID 之后的所有消息，客户端去重后插入列表。此外还需处理消息的局部排序（服务端推送的消息可能乱序到达，需要按 timestamp 或 seqId 排序后渲染）。

5. **大型聊天群优化**：单聊天 10000+ 消息时即使虚拟化也吃力，可以考虑用 FlashList 或者按时间段分页持久化。1 万条消息即使用 FlatList 虚拟化，JS 线程遍历 data 数组做 diff 的开销也不可忽略。FlashList 的 cell pooling 机制更高效；另一种思路是将历史消息持久化到本地数据库（MMKV/Realm），内存中只保留最近 200 条 + 当前可视区域的数据，用户滚动到更早的位置时按需从数据库分页加载，内存占用恒定。


## 十三、实践代码

> 以下为各题中提到的代码实现，按功能分组归纳，便于查阅与复用。

### 页面级组件实现

#### LoginScreen 登录页 {#practices-login}

展示 RN 核心组件（View、Text、Image、TextInput、TouchableOpacity）的组合用法，包含受控输入与 StyleSheet 样式定义。

```javascript
import React, { useState } from 'react'
import {
  View,
  Text,
  Image,
  TextInput,
  TouchableOpacity,
  StyleSheet
} from 'react-native'

function LoginScreen() {
  const [username, setUsername] = useState('')
  const [password, setPassword] = useState('')

  return (
    <View style={styles.container}>
      <Image
        source={require('./logo.png')}
        style={styles.logo}
      />

      <TextInput
        style={styles.input}
        placeholder="用户名"
        value={username}
        onChangeText={setUsername}
      />

      <TextInput
        style={styles.input}
        placeholder="密码"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />

      <TouchableOpacity
        style={styles.button}
        onPress={handleLogin}
      >
        <Text style={styles.buttonText}>登录</Text>
      </TouchableOpacity>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    padding: 20
  },
  logo: {
    width: 100,
    height: 100,
    alignSelf: 'center',
    marginBottom: 40
  },
  input: {
    height: 50,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    paddingHorizontal: 15,
    marginBottom: 15
  },
  button: {
    height: 50,
    backgroundColor: '#007AFF',
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center'
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold'
  }
})
```


#### ChatScreen 键盘适配（手动监听方案） {#practices-keyboard}

通过 Keyboard.addListener 监听 keyboardWillShow/Hide 事件，用 Animated.Value 驱动 paddingBottom 联动键盘高度，实现输入框紧贴键盘的精细控制。

```javascript
import { Keyboard, Animated } from 'react-native'

function ChatScreen() {
  const keyboardHeight = useRef(new Animated.Value(0)).current

  useEffect(() => {
    const showSub = Keyboard.addListener('keyboardWillShow', (e) => {
      Animated.timing(keyboardHeight, {
        toValue: e.endCoordinates.height,
        duration: e.duration,  // 跟随系统键盘动画时长
        useNativeDriver: false  // 高度变化无法用原生驱动
      }).start()
    })

    const hideSub = Keyboard.addListener('keyboardWillHide', (e) => {
      Animated.timing(keyboardHeight, {
        toValue: 0,
        duration: e.duration,
        useNativeDriver: false
      }).start()
    })

    return () => {
      showSub.remove()
      hideSub.remove()
    }
  }, [])

  return (
    <Animated.View style={{ flex: 1, paddingBottom: keyboardHeight }}>
      <ChatList />
      <TextInput />
    </Animated.View>
  )
}
```


#### ProductList 下拉刷新与上拉加载 {#practices-pull-refresh}

FlatList + RefreshControl 实现下拉刷新，onEndReached 实现上拉加载更多，包含防重复触发、页码 ref 管理、分页状态判断等关键细节。

```javascript
import { FlatList, RefreshControl } from 'react-native'

function ProductList() {
  const [data, setData] = useState([])
  const [refreshing, setRefreshing] = useState(false)
  const [loadingMore, setLoadingMore] = useState(false)
  const [hasMore, setHasMore] = useState(true)
  const pageRef = useRef(1)

  // 下拉刷新：重置数据
  const onRefresh = useCallback(async () => {
    setRefreshing(true)
    try {
      const fresh = await fetchProducts({ page: 1 })
      setData(fresh.list)
      setHasMore(fresh.hasMore)
      pageRef.current = 1
    } finally {
      setRefreshing(false)
    }
  }, [])

  // 上拉加载：追加下一页
  const onLoadMore = useCallback(async () => {
    if (loadingMore || !hasMore) return  // 防重复触发
    setLoadingMore(true)
    try {
      const next = await fetchProducts({ page: pageRef.current + 1 })
      setData(prev => [...prev, ...next.list])
      setHasMore(next.hasMore)
      pageRef.current += 1
    } finally {
      setLoadingMore(false)
    }
  }, [loadingMore, hasMore])

  return (
    <FlatList
      data={data}
      renderItem={({ item }) => <ProductCard product={item} />}
      keyExtractor={item => item.id}
      // 下拉刷新
      refreshControl={
        <RefreshControl
          refreshing={refreshing}
          onRefresh={onRefresh}
          tintColor="#007AFF"  // iOS loading 颜色
          colors={['#007AFF']}  // Android loading 颜色
        />
      }
      // 上拉加载
      onEndReached={onLoadMore}
      onEndReachedThreshold={0.5}  // 距底部 50% 时触发
      ListFooterComponent={
        loadingMore ? <ActivityIndicator style={{ padding: 20 }} /> :
        !hasMore ? <Text style={{ textAlign: 'center', padding: 20 }}>没有更多了</Text> :
        null
      }
    />
  )
}
```


#### ChatScreen 聊天界面 {#practices-chat}

反向列表（inverted）+ maintainVisibleContentPosition 防跳动 + KeyboardAvoidingView + 历史消息加载 + 新消息自动滚动。

```javascript
import { FlatList, KeyboardAvoidingView, Platform } from 'react-native'

function ChatScreen({ chatId }) {
  const [messages, setMessages] = useState([])
  const [loadingHistory, setLoadingHistory] = useState(false)
  const flatListRef = useRef(null)

  // 关键：列表反向(inverted),最新消息在底部
  const renderItem = ({ item }) => <MessageBubble message={item} />

  // 上拉加载历史(在反向列表中,onEndReached 触发于"看到顶部"时)
  const loadHistory = useCallback(async () => {
    if (loadingHistory) return
    setLoadingHistory(true)
    try {
      const oldest = messages[messages.length - 1]
      const history = await api.fetchHistory(chatId, oldest?.id)
      setMessages(prev => [...prev, ...history])  // 追加到末尾(=列表顶部)
    } finally {
      setLoadingHistory(false)
    }
  }, [chatId, messages, loadingHistory])

  // 收到新消息：插入到数组开头(=列表底部)
  useEffect(() => {
    const sub = socket.on('newMessage', (msg) => {
      if (msg.chatId !== chatId) return
      setMessages(prev => [msg, ...prev])
      // 如果用户在底部,自动滚到新消息;否则只更新数据
      if (isAtBottom.current) {
        flatListRef.current?.scrollToOffset({ offset: 0, animated: true })
      }
    })
    return () => sub.unsubscribe()
  }, [chatId])

  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : undefined}
      style={{ flex: 1 }}
    >
      <FlatList
        ref={flatListRef}
        data={messages}
        renderItem={renderItem}
        keyExtractor={m => m.id}
        inverted={true}                       // 关键!列表反向
        onEndReached={loadHistory}            // 滚到顶部触发(因为反向了)
        onEndReachedThreshold={0.3}
        ListFooterComponent={
          loadingHistory ? <ActivityIndicator /> : null
        }
        // 性能优化
        maintainVisibleContentPosition={{
          minIndexForVisible: 0  // 加载历史时保持当前可见消息位置不跳变
        }}
      />
      <ChatInput onSend={handleSend} />
    </KeyboardAvoidingView>
  )
}
```


### 高性能列表实现

#### BigList 万级数据列表优化 {#practices-biglist}

React.memo + useCallback + getItemLayout + 虚拟化参数调优的方案，包含自定义 memo 比较函数和样式定义。

```javascript
import React, { useCallback, memo } from 'react'
import { FlatList, StyleSheet, View, Text } from 'react-native'
import FastImage from 'react-native-fast-image'

const ITEM_HEIGHT = 80  // 固定高度,关键!

// 1. item 用 React.memo 隔离
const ItemRow = memo(({ item, onPress }) => {
  return (
    <View style={styles.row}>
      <FastImage
        source={{
          uri: item.thumbnail,
          priority: FastImage.priority.normal,
          cache: FastImage.cacheControl.immutable
        }}
        style={styles.thumb}
      />
      <Text numberOfLines={1} style={styles.title}>{item.title}</Text>
    </View>
  )
}, (prev, next) => {
  // 自定义比较:只比关心的字段
  return prev.item.id === next.item.id &&
         prev.item.title === next.item.title
})

function BigList({ data }) {
  // 2. 用 useCallback 稳定 renderItem 引用
  const renderItem = useCallback(({ item }) => (
    <ItemRow item={item} onPress={handlePress} />
  ), [])

  // 3. getItemLayout 跳过测量(需要 ITEM_HEIGHT 已知)
  const getItemLayout = useCallback((_, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index
  }), [])

  const keyExtractor = useCallback((item) => item.id, [])

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      // 4. 调优虚拟化参数
      initialNumToRender={15}        // 首屏精简,加快 TTI
      maxToRenderPerBatch={10}       // 每帧最多渲染 10 项,避免长任务
      updateCellsBatchingPeriod={50} // 50ms 批处理,合并多次更新
      windowSize={5}                 // 维护 5 屏内容,平衡流畅度和内存
      removeClippedSubviews={true}   // 屏幕外视图卸载(Android 显著)
      // 5. 滚动优化
      onEndReachedThreshold={0.5}
      onEndReached={loadMore}
    />
  )
}

const styles = StyleSheet.create({
  row: {
    height: ITEM_HEIGHT,
    flexDirection: 'row',
    paddingHorizontal: 16,
    alignItems: 'center'
  },
  thumb: { width: 60, height: 60, borderRadius: 8 },
  title: { flex: 1, marginLeft: 12, fontSize: 14 }
})
```


#### SortableList 拖拽排序（react-native-draggable-flatlist） {#practices-draggable}

使用 react-native-draggable-flatlist 实现长按拖拽排序，onDragEnd 回调更新数据顺序。

```javascript
import DraggableFlatList from 'react-native-draggable-flatlist'

function SortableList() {
  const [data, setData] = useState([
    { key: '1', label: '任务 A' },
    { key: '2', label: '任务 B' },
    { key: '3', label: '任务 C' }
  ])

  const renderItem = ({ item, drag, isActive }) => (
    <TouchableOpacity
      onLongPress={drag}        // 长按触发拖拽
      disabled={isActive}
      style={[
        styles.row,
        { backgroundColor: isActive ? '#eee' : '#fff' }
      ]}
    >
      <Text>{item.label}</Text>
    </TouchableOpacity>
  )

  return (
    <DraggableFlatList
      data={data}
      onDragEnd={({ data }) => setData(data)}  // 拖拽结束更新数据
      keyExtractor={item => item.key}
      renderItem={renderItem}
    />
  )
}
```


#### DraggableItem 手势拖拽组件（自实现） {#practices-gesture-drag}

基于 react-native-gesture-handler + react-native-reanimated 自实现拖拽，含 Pan 手势、worklet 动画、runOnJS 数据回传。

```javascript
import { GestureDetector, Gesture } from 'react-native-gesture-handler'
import Animated, { useSharedValue, useAnimatedStyle, withSpring, runOnJS } from 'react-native-reanimated'

function DraggableItem({ item, index, onReorder }) {
  const translateY = useSharedValue(0)
  const isActive = useSharedValue(false)

  const gesture = Gesture.Pan()
    .activateAfterLongPress(300)  // 长按 300ms 触发
    .onStart(() => {
      isActive.value = true
    })
    .onUpdate((e) => {
      translateY.value = e.translationY
    })
    .onEnd(() => {
      // 计算最终位置,通知 JS 层重排
      const targetIndex = Math.round(translateY.value / ITEM_HEIGHT)
      runOnJS(onReorder)(index, index + targetIndex)
      translateY.value = withSpring(0)
      isActive.value = false
    })

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: translateY.value }],
    zIndex: isActive.value ? 999 : 0,
    elevation: isActive.value ? 8 : 0
  }))

  return (
    <GestureDetector gesture={gesture}>
      <Animated.View style={[styles.item, animatedStyle]}>
        <Text>{item.label}</Text>
      </Animated.View>
    </GestureDetector>
  )
}
```


### 导航与路由配置

#### Stack Navigator 栈导航配置 {#practices-stack-nav}

React Navigation Stack 导航配置，包含 NavigationContainer、screenOptions 统一主题、页面间参数传递与返回。

```javascript
import React from 'react'
import { NavigationContainer } from '@react-navigation/native'
import { createStackNavigator } from '@react-navigation/stack'

const Stack = createStackNavigator()

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          headerStyle: { backgroundColor: '#007AFF' },
          headerTintColor: '#fff',
          headerTitleStyle: { fontWeight: 'bold' }
        }}
      >
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{ title: '首页' }}
        />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={({ route }) => ({ title: route.params.title })}
        />
      </Stack.Navigator>
    </NavigationContainer>
  )
}

// 页面组件
function HomeScreen({ navigation }) {
  return (
    <View>
      <Button
        title="跳转到详情"
        onPress={() => navigation.navigate('Details', {
          id: 1,
          title: '详情页'
        })}
      />
    </View>
  )
}

function DetailsScreen({ route, navigation }) {
  const { id, title } = route.params

  return (
    <View>
      <Text>{title}</Text>
      <Button title="返回" onPress={() => navigation.goBack()} />
    </View>
  )
}
```


#### Bottom Tab Navigator 标签导航配置 {#practices-tab-nav}

底部标签导航配置，含自定义 tabBarIcon（区分聚焦态）、全局主题色设置。

```javascript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs'
import Icon from 'react-native-vector-icons/Ionicons'

const Tab = createBottomTabNavigator()

function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator
        screenOptions={({ route }) => ({
          tabBarIcon: ({ focused, color, size }) => {
            let iconName
            if (route.name === 'Home') {
              iconName = focused ? 'home' : 'home-outline'
            } else if (route.name === 'Profile') {
              iconName = focused ? 'person' : 'person-outline'
            }
            return <Icon name={iconName} size={size} color={color} />
          },
          tabBarActiveTintColor: '#007AFF',
          tabBarInactiveTintColor: 'gray'
        })}
      >
        <Tab.Screen name="Home" component={HomeScreen} />
        <Tab.Screen name="Profile" component={ProfileScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  )
}
```


#### Deep Linking 应用组件 {#practices-deeplink}

处理冷启动（getInitialURL）和热启动（addEventListener 'url'）两种场景，解析 URL 路径和参数后路由分发。

```javascript
import { Linking } from 'react-native'
import { useNavigation } from '@react-navigation/native'

function App() {
  const navigation = useNavigation()

  useEffect(() => {
    // 1. 处理冷启动场景：App 未启动时点击链接打开
    Linking.getInitialURL().then(url => {
      if (url) handleDeepLink(url)
    })

    // 2. 处理热启动场景：App 已在后台，点击链接唤起
    const subscription = Linking.addEventListener('url', ({ url }) => {
      handleDeepLink(url)
    })

    return () => subscription.remove()
  }, [])

  const handleDeepLink = (url) => {
    // 解析 URL：myapp://product/123?from=share
    const route = url.replace(/.*?:\/\//g, '')      // product/123?from=share
    const [path, queryString] = route.split('?')
    const [screen, ...params] = path.split('/')      // ['product', '123']

    // 解析 query
    const query = Object.fromEntries(
      new URLSearchParams(queryString).entries()
    )

    // 路由分发
    switch (screen) {
      case 'product':
        navigation.navigate('ProductDetail', { id: params[0], ...query })
        break
      case 'order':
        navigation.navigate('OrderDetail', { id: params[0] })
        break
      default:
        navigation.navigate('Home')
    }
  }
}
```


#### React Navigation Linking 配置 {#practices-linking-config}

声明式配置 prefixes、screens 路径映射与 parse 参数转换，让 React Navigation 自动处理 Deep Link 路由。

```javascript
import { NavigationContainer } from '@react-navigation/native'

const linking = {
  prefixes: ['myapp://', 'https://example.com'],
  config: {
    screens: {
      Home: '',
      ProductDetail: 'product/:id',
      OrderDetail: 'order/:id',
      Profile: {
        path: 'user/:userId',
        parse: {
          userId: (id) => parseInt(id)  // 类型转换
        }
      }
    }
  }
}

<NavigationContainer linking={linking}>
  <RootNavigator />
</NavigationContainer>
```


### 原生模块开发

#### iOS 原生模块（Objective-C） {#practices-ios-module}

使用 RCT_EXPORT_MODULE / RCT_EXPORT_METHOD 宏暴露方法给 JS，通过 Promise 回调返回结果。

```objc
// CalendarModule.h
#import <React/RCTBridgeModule.h>

@interface CalendarModule : NSObject <RCTBridgeModule>
@end

// CalendarModule.m
#import "CalendarModule.h"

@implementation CalendarModule

// 注册模块,JS 端通过 NativeModules.CalendarModule 访问
RCT_EXPORT_MODULE();

// 暴露方法给 JS
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                  location:(NSString *)location
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  @try {
    NSString *eventId = [self saveEventToCalendar:title location:location];
    resolve(eventId);
  } @catch (NSException *exception) {
    reject(@"create_event_failed", exception.reason, nil);
  }
}

@end
```


#### Android 原生模块（Java） {#practices-android-module}

继承 ReactContextBaseJavaModule，用 @ReactMethod 注解暴露方法，通过 Promise 回调返回结果。

```java
// CalendarModule.java
package com.myapp;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Promise;

public class CalendarModule extends ReactContextBaseJavaModule {

    public CalendarModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }

    // JS 端通过 NativeModules.CalendarModule 访问
    @Override
    public String getName() {
        return "CalendarModule";
    }

    // 暴露方法给 JS
    @ReactMethod
    public void createCalendarEvent(String title, String location, Promise promise) {
        try {
            String eventId = saveEventToCalendar(title, location);
            promise.resolve(eventId);
        } catch (Exception e) {
            promise.reject("create_event_failed", e.getMessage());
        }
    }
}
```


### 功能模块与工具

#### 响应式工具模块 {#practices-responsive}

基于 Dimensions 和 PixelRatio 封装的 wp/hp/normalize 工具函数，实现基于设计稿的尺寸适配方案。

```javascript
// utils/responsive.js
import { Dimensions, PixelRatio } from 'react-native'

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window')

// 基于设计稿宽度 375 的缩放
const scale = SCREEN_WIDTH / 375

export function wp(percentage) {
  return (percentage * SCREEN_WIDTH) / 100
}

export function hp(percentage) {
  return (percentage * SCREEN_HEIGHT) / 100
}

export function normalize(size) {
  const newSize = size * scale
  return Math.round(PixelRatio.roundToNearestPixel(newSize))
}

// 使用
import { wp, hp, normalize } from './utils/responsive'

const styles = StyleSheet.create({
  container: {
    width: wp(90),      // 屏幕宽度的 90%
    height: hp(50),     // 屏幕高度的 50%
    padding: normalize(20)
  }
})
```


#### CodePush 手动检查更新 {#practices-codepush}

手动调用 codePush.checkForUpdate 检查更新，配合 Alert 提示用户，sync 下载并监听进度与状态回调。

```javascript
async function checkForUpdate() {
  try {
    const update = await codePush.checkForUpdate()
    if (!update) {
      Alert.alert('已是最新版本')
      return
    }

    // 提示用户更新
    Alert.alert(
      '发现新版本',
      update.description || '点击立即更新',
      [
        { text: '取消' },
        {
          text: '更新',
          onPress: () => {
            codePush.sync(
              { installMode: codePush.InstallMode.IMMEDIATE },
              (status) => {
                // 监听状态：DOWNLOADING_PACKAGE / INSTALLING_UPDATE 等
              },
              (progress) => {
                // 显示下载进度
                console.log(`${progress.receivedBytes}/${progress.totalBytes}`)
              }
            )
          }
        }
      ]
    )
  } catch (e) {
    console.error(e)
  }
}
```


#### Firebase 推送通知实现示例 {#practices-push}

覆盖请求权限、获取 Token、监听 Token 刷新、前台/后台/冷启动/静默四种推送场景的处理流程。

```javascript
import messaging from '@react-native-firebase/messaging'

// 1. 请求推送权限（iOS 必需）
async function requestPermission() {
  const authStatus = await messaging().requestPermission()
  const enabled =
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL

  if (enabled) {
    // 2. 获取设备 Token,上传到自己的服务器
    const token = await messaging().getToken()
    await uploadTokenToServer(token)

    // 3. 监听 Token 刷新（用户卸载重装、系统重置等会变化）
    messaging().onTokenRefresh(async (newToken) => {
      await uploadTokenToServer(newToken)
    })
  }
}

// 4. 监听三种推送场景
function setupNotifications() {
  // 场景 A：App 在前台时收到推送（系统不会自动展示通知）
  messaging().onMessage(async remoteMessage => {
    // 自己用 react-native-notifications 等库展示横幅
    showInAppBanner(remoteMessage)
  })

  // 场景 B：App 在后台时点击推送通知
  messaging().onNotificationOpenedApp(remoteMessage => {
    // 跳转到对应页面
    handleNotificationClick(remoteMessage)
  })

  // 场景 C：App 完全退出时点击推送通知（冷启动）
  messaging().getInitialNotification().then(remoteMessage => {
    if (remoteMessage) handleNotificationClick(remoteMessage)
  })

  // 场景 D：App 在后台收到数据消息（不显示 UI 仅做静默处理）
  messaging().setBackgroundMessageHandler(async remoteMessage => {
    // 比如：增量同步消息、刷新本地缓存
    await syncDataInBackground(remoteMessage.data)
  })
}
```


#### 跨平台权限请求 {#practices-permission}

使用 react-native-permissions 封装跨平台权限请求，区分 GRANTED/DENIED/BLOCKED/UNAVAILABLE 四种状态并分别处理。

```javascript
import { PermissionsAndroid, Platform, Alert, Linking } from 'react-native'
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions'

// 权限请求流程（推荐用 react-native-permissions 跨平台库）
async function requestCameraPermission() {
  const permission = Platform.select({
    ios: PERMISSIONS.IOS.CAMERA,
    android: PERMISSIONS.ANDROID.CAMERA,
  })

  // 1. 先检查当前状态
  const status = await check(permission)

  switch (status) {
    case RESULTS.GRANTED:
      return true  // 已授权,直接用

    case RESULTS.DENIED:
      // 还能申请,弹系统授权框
      const result = await request(permission)
      return result === RESULTS.GRANTED

    case RESULTS.BLOCKED:
      // 用户曾经选了"不再询问",需要引导去设置
      Alert.alert(
        '需要相机权限',
        '请在系统设置中开启相机权限',
        [
          { text: '取消' },
          { text: '去设置', onPress: () => Linking.openSettings() }
        ]
      )
      return false

    case RESULTS.UNAVAILABLE:
      // 设备不支持(如平板没相机)
      Alert.alert('设备不支持')
      return false

    default:
      return false
  }
}
```


#### i18next 国际化配置 {#practices-i18n}

i18next + react-i18next 配置，含资源定义、变量插值、复数处理、日期格式化、语言切换。

```javascript
import i18n from 'i18next'
import { initReactI18next, useTranslation } from 'react-i18next'

const resources = {
  en: {
    translation: {
      welcome: 'Welcome, {{name}}',
      itemCount_one: '{{count}} item',
      itemCount_other: '{{count}} items',
      lastSeen: 'Last seen {{date, datetime}}'
    }
  },
  zh: {
    translation: {
      welcome: '欢迎，{{name}}',
      itemCount: '{{count}} 件商品',
      lastSeen: '最后访问于 {{date, datetime}}'
    }
  }
}

i18n
  .use(initReactI18next)
  .init({
    resources,
    lng: 'zh',
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false  // RN 不需要 XSS 转义
    }
  })

function HomeScreen() {
  const { t, i18n } = useTranslation()

  return (
    <View>
      {/* 带变量插值 */}
      <Text>{t('welcome', { name: 'Alice' })}</Text>

      {/* 复数处理（英文 1 item / 2 items，中文不区分） */}
      <Text>{t('itemCount', { count: 5 })}</Text>

      {/* 日期格式化 */}
      <Text>{t('lastSeen', { date: new Date() })}</Text>

      {/* 切换语言 */}
      <Button title="English" onPress={() => i18n.changeLanguage('en')} />
      <Button title="中文" onPress={() => i18n.changeLanguage('zh')} />
    </View>
  )
}
```

## 进阶高频题与新架构实战

### React Native 新架构（New Architecture）到底解决了什么？

新架构是 24-25 年 RN 最大的事，0.76 起默认开启。它包含三大块：**Fabric（新渲染器）+ TurboModules（新原生模块系统）+ JSI（新通信层）**，外加 Codegen 这个编译工具。

先说明老架构的痛点，才能理解新架构为什么要重写。

**老架构的瓶颈：JS Bridge**

老 RN 的 JS 和原生之间靠一个「Bridge」通信，它是个**异步消息队列 + JSON 序列化**的东西：

```
JS 线程 ──[序列化为 JSON]──> Bridge ──[反序列化]──> 原生线程
```

每次 JS 想调原生（比如让原生开个相机）、原生想调 JS（比如手势事件回调），都要走这条链路。问题：

- 序列化开销大——传一个大对象（手势事件、列表数据）每帧都要 JSON 化
- 异步——没法做「JS 同步问原生要个值」这种事
- 单 Bridge——所有跨线程通信挤一个通道

实际表现：列表滚动时一帧要传几百个消息，Bridge 堵了，FPS 掉到 30 以下。

**JSI（JavaScript Interface）**

JSI 是 C++ 写的「JS 引擎到原生的调用层」。它不是消息队列——而是让 JS 能拿到「原生 C++ 对象的 JS 代理」，直接同步调用。

```cpp
// 简化的概念示意
class NativeModule : public jsi::HostObject {
  jsi::Value get(jsi::Runtime& rt, const jsi::PropNameID& name) {
    if (name.utf8(rt) == "getDeviceId") {
      return jsi::Function::createFromHostFunction(rt, ...,
        [](jsi::Runtime& rt, ...) {
          return jsi::String::createFromUtf8(rt, getRealDeviceId());
        }
      );
    }
  }
};
```

JS 拿到这个对象后，`module.getDeviceId()` 是**同步**的 C++ 调用，没有序列化、没有 Bridge。也就是 Reanimated 2 这种「JS 写动画但跑在 UI 线程」成为可能——动画 worklet 通过 JSI 直接读写原生 view 的属性。

**Fabric（新渲染器）**

老架构的渲染：JS 算 vdom → Bridge 传 → 原生 shadow tree → 原生 view tree，全程异步。

Fabric 把这条链路改成同步——React commit 阶段在 C++ 层直接构建 shadow tree，commit 完成时 UI 线程立刻能拿到。配合 React 18 的 concurrent rendering，可以做到「优先级抢占」和「并发渲染」。

**TurboModules + Codegen**

老的原生模块用 `requireNativeComponent` 注册，运行时动态查找，类型不安全。

新方案：

- 用 TypeScript 定义模块接口
- `Codegen` 工具自动生成 C++ / Java / Objective-C 胶水代码
- TurboModule 直接通过 JSI 暴露给 JS

```ts
// MyModuleSpec.ts
import type { TurboModule } from 'react-native'
import { TurboModuleRegistry } from 'react-native'

export interface Spec extends TurboModule {
  getValue(): string
  add(a: number, b: number): Promise<number>
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyModule')
```

Codegen 会根据这个 spec 生成对应 Native 代码骨架。

**实际收益**：

- 启动时间减少 20-40%
- 列表滚动 FPS 提升明显
- Reanimated 3 + Gesture Handler 在新架构下能跑超复杂动画

**开启新架构（0.76+ 默认开）**：

```bash
# iOS
cd ios && RCT_NEW_ARCH_ENABLED=1 pod install

# Android: gradle.properties 加
# newArchEnabled=true
```

要注意的几点：

- 老架构 = JS Bridge（异步、JSON 序列化、单通道）
- 新架构 = JSI（C++ 同步调用）+ Fabric（同步渲染）+ TurboModules（强类型原生模块）
- 0.76 默认开启，老项目升级要踩兼容性

### Hermes 引擎跟 JSC 的对比，什么时候应该禁用？

Hermes 是 Meta 为 RN 自研的 JS 引擎，0.70 起 Android/iOS 都默认启用。

**Hermes 跟 JSC 的差异**：

| 维度 | JSC（JavaScriptCore，老默认） | Hermes |
|------|------------------------------|--------|
| 启动速度 | 慢（要解析 + JIT） | 快（bytecode 预编译） |
| 内存 | 大 | 小（启动时少 30-50%） |
| 性能 | 长时间运行后 JIT 优化能跟上 | 没有 JIT，纯解释执行字节码 |
| 包体积 | 小 | 略大（要带 Hermes 引擎） |
| 调试 | Safari/Chrome | Chrome（Hermes 适配） |
| Profiler | 一般 | 内置 sampling profiler |

**Hermes 的做法**：预编译 JS 为 bytecode（`.hbc` 文件），跳过运行时解析。结果是冷启动快、内存低，但长时间运行的 hot loop 没有 JIT 加持，理论峰值性能不如 JSC。

实测移动 App 场景：

- 冷启动：Hermes 快 30-50%
- 内存：Hermes 省 30%
- 长时间运行：差不多

**什么时候禁用 Hermes？**

- 你的应用是「跑很久 + 高性能计算」的（游戏、实时音视频处理）—— JSC 的 JIT 可能更适合
- 用了某些不兼容 Hermes 的库（早年的某些库依赖 JSC 特有 API，现在基本都修了）
- 调试需要 Safari 链路（Hermes 主要用 Chrome）

**怎么禁用**：

```js
// android/app/build.gradle
project.ext.react = [
  enableHermes: false
]

// iOS: Podfile
use_react_native!(
  :hermes_enabled => false
)
```

但 24-25 年的项目几乎都用 Hermes，除非有特殊原因。

### Reanimated 3 跟老 Animated API 的本质差异

老的 `Animated` API 在新架构出现前就是「在 JS 线程算动画值 + Bridge 传给原生 view」，每帧都要跨线程通信。复杂动画一卡顿就掉帧。

`Animated` 有个 `useNativeDriver: true` 模式能把简单动画跑在 UI 线程，但限制不少：只能动画 transform / opacity，无法改 width/height/color。

**Reanimated 2/3 的范式**：

```jsx
import Animated, {
  useSharedValue, useAnimatedStyle, withSpring
} from 'react-native-reanimated'

function Box() {
  const offset = useSharedValue(0)

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: offset.value }]
  }))

  return (
    <>
      <Animated.View style={[styles.box, animatedStyle]} />
      <Button onPress={() => offset.value = withSpring(Math.random() * 200)} />
    </>
  )
}
```

`useAnimatedStyle` 里的回调是个 **worklet**——通过 JSI 直接编译为 C++ 字节码，跑在 UI 线程。整个动画过程：

- JS 线程：触发动画（`offset.value = ...`）
- UI 线程：worklet 计算新值、直接更新原生 view 属性
- 全程不走 JS 线程，60fps 不会因为 JS 卡而掉帧

**能做到老 Animated 做不到的事**：

- 动画任何属性（width、height、color、shadow）
- 手势驱动的复杂联动（拖拽时实时缩放、旋转）
- 物理引擎（spring、decay、复杂插值）
- 跟 Gesture Handler 配合实现 60fps 手势

**Gesture Handler 的位置**：

新架构下 `react-native-gesture-handler` v2 也是基于 worklet 的，跟 Reanimated 配合实现「手势开始 → 实时跟随 → 释放后动画」全程 UI 线程。这也是 Bottom Sheet、Drawer、Swipe Action 这些交互在 RN 里能接近原生体验的原因。

选型上的经验：

- 新项目直接 Reanimated 3 + Gesture Handler 2
- 简单 fadeIn / slideIn 用老 Animated 没问题
- 复杂手势动画需要上 Reanimated

### Expo SDK 50+ 现代 RN 开发

24-25 年新 RN 项目几乎都从 Expo 开始。原因：

**传统方案：bare RN**

```bash
npx react-native init MyApp
# 然后你要自己维护 ios/、android/ 原生项目
# 每次升级 RN 都是改 Podfile、改 build.gradle 的大工程
```

**新路：Expo**

```bash
npx create-expo-app MyApp
# 你的项目里没有 ios/ android/ 目录（默认）
# Expo 帮你管理原生工程,叫"prebuild"
```

**Expo 50+ 的关键改进**：

**1. Prebuild（CNG，Continuous Native Generation）**：

Expo 的 `app.json` / `app.config.js` 是配置入口，里面声明「我要 push 通知、要相机权限」。运行 `npx expo prebuild` 时 Expo 根据配置生成 ios/ android/ 目录。

```js
// app.config.js
export default {
  expo: {
    name: "MyApp",
    plugins: [
      "expo-camera",
      ["expo-notifications", {
        icon: "./assets/notif-icon.png"
      }]
    ]
  }
}
```

每次升级 SDK，删掉 ios/ android/ 重新 prebuild，永远不用手动改 Podfile。

**2. EAS Build**：

云端构建服务。本地不用装 Xcode、不用装 Android Studio，提交命令到 Expo 服务器构建：

```bash
eas build --platform ios --profile production
```

输出 .ipa / .apk 后即可下载。对没有 Mac 的 Windows 开发者很有帮助。

**3. EAS Update（替代 CodePush）**：

CodePush 24 年起逐步停服（微软不再维护），EAS Update 是 RN 生态的新标准动态化方案：

```bash
eas update --branch production --message "fix login bug"
```

App 启动时检查更新，没新版本直接跑、有新版本下载 JS bundle 热更新。

**4. expo-router**：

类似 Next.js App Router 的「文件即路由」：

```
app/
├── _layout.tsx
├── index.tsx           # 首页
├── (tabs)/
│   ├── home.tsx
│   └── profile.tsx
└── post/[id].tsx       # 动态路由
```

替代手写 React Navigation 路由配置，工程化体验大幅提升。

**Expo 适用边界**：

- 通用业务 App：Expo 通常已经足够
- 重度原生定制（自定义渲染、复杂相机滤镜）：bare RN + 手写原生模块更灵活，但工作量大
- 中间地带：Expo prebuild + 自己加原生模块也行

**新项目选型**：多数场景可以优先考虑 Expo，除非有特殊的原生需求。

### RN 大项目的导航 + 状态管理选型

**导航：React Navigation 6/7**

事实标准。RN 生态没有更接近竞品（Wix 的 react-native-navigation 已经式微）。

```jsx
import { NavigationContainer } from '@react-navigation/native'
import { createNativeStackNavigator } from '@react-navigation/native-stack'

const Stack = createNativeStackNavigator()

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Detail" component={DetailScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  )
}
```

关键选型：用 `native-stack`（基于原生 native API）还是 `stack`（纯 JS 实现）？**native-stack 性能更好、动画更原生，新项目首选**。

**状态管理**：

跟 Web React 一致，但因为 RN 没有 url 概念，URL state 这里要换思路。

| 场景 | 推荐 |
|------|------|
| 服务端数据 | TanStack Query / SWR |
| 客户端 UI 状态 | Zustand |
| 复杂表单 | react-hook-form |
| 实时协作 | Zustand + Yjs |

**深链处理**：

```jsx
const linking = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      Home: '',
      Detail: 'detail/:id',
    },
  },
}

<NavigationContainer linking={linking}>
```

外部点击 `myapp://detail/123` 或 `https://myapp.com/detail/123` 都能正确进入 Detail 页。

**推送处理**：

```jsx
import * as Notifications from 'expo-notifications'

// 接收推送时跳转
Notifications.addNotificationResponseReceivedListener((response) => {
  const { screen, params } = response.notification.request.content.data
  navigation.navigate(screen, params)
})
```

需要 expo-notifications + 服务端配合 FCM/APNs。

### RN 性能优化实战

新架构开启后性能基线已经很好，但大型 App 还是要主动优化。

**长列表用 FlashList**：

`FlatList` 是 RN 内置的虚拟列表，但 Shopify 出的 `@shopify/flash-list` 性能更好——重用机制更彻底，复杂列表能从 30fps 提到 60fps。

```jsx
import { FlashList } from '@shopify/flash-list'

<FlashList
  data={items}
  renderItem={({ item }) => <ListItem item={item} />}
  estimatedItemSize={80}   // 关键:预估高度让 FlashList 优化
/>
```

**图片用 expo-image**：

```jsx
import { Image } from 'expo-image'

<Image
  source={uri}
  placeholder={blurhash}     // 占位图(支持 BlurHash)
  contentFit="cover"
  transition={200}
  cachePolicy="memory-disk"
/>
```

替代 `react-native-fast-image`（基于 SDWebImage / Glide），expo-image 是较新的选择，性能跟 FastImage 持平但 API 更现代。

**避免 re-render**：

- 用 `React.memo` 包列表项
- `useCallback` 稳定传给子组件的函数
- 状态拆细，避免一个 setState 重渲整个屏幕

**Profiling 工具**：

- **Flipper**（停止维护，逐步被替代）
- **React DevTools** + RN：profile render
- **Hermes Profiler**：CPU profile，定位 JS 热点
- **Xcode Instruments** / **Android Studio Profiler**：原生性能

**包体积优化**：

- 删掉不用的依赖（`depcheck`）
- 用 `react-native-bundle-visualizer` 看 JS bundle 占比
- 图片资源走 CDN 别打包进 App
- 启用 Hermes（默认）+ ProGuard / R8

### TurboModule 编写实战

新架构下写原生模块的流程跟老方式（`NativeModules`）差异明显。简化路径：

**1. 定义 TS 接口**：

```ts
// MyTurboModuleSpec.ts
import type { TurboModule } from 'react-native'
import { TurboModuleRegistry } from 'react-native'

export interface Spec extends TurboModule {
  getDeviceId(): string
  vibrate(duration: number): void
  showAlert(title: string, message: string): Promise<void>
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyTurboModule')
```

**2. 配置 Codegen**：

```jsonc
// package.json
{
  "codegenConfig": {
    "name": "MyTurboModuleSpec",
    "type": "modules",
    "jsSrcsDir": "./src"
  }
}
```

**3. iOS 实现**：

```objc
// MyTurboModule.h
#import "MyTurboModuleSpec.h"

@interface MyTurboModule : NSObject <NativeMyTurboModuleSpec>
@end

// MyTurboModule.mm
@implementation MyTurboModule

RCT_EXPORT_MODULE()

- (NSString *)getDeviceId {
  return [[UIDevice currentDevice].identifierForVendor UUIDString];
}

- (void)vibrate:(double)duration {
  AudioServicesPlaySystemSound(kSystemSoundID_Vibrate);
}
@end
```

**4. Android 实现**：

```kotlin
// MyTurboModule.kt
class MyTurboModule(reactContext: ReactApplicationContext)
    : NativeMyTurboModuleSpec(reactContext) {

  override fun getName() = "MyTurboModule"

  override fun getDeviceId(): String {
    return Settings.Secure.getString(
      reactApplicationContext.contentResolver,
      Settings.Secure.ANDROID_ID
    )
  }

  override fun vibrate(duration: Double) {
    val vibrator = reactApplicationContext.getSystemService(VIBRATOR_SERVICE) as Vibrator
    vibrator.vibrate(duration.toLong())
  }
}
```

**5. 注册到 App**：

iOS 在 AppDelegate 注册，Android 在 ReactPackage 里注册。具体看 `codegen` 生成的骨架。

跟老的 NativeModule 比，优势：

- 强类型——TS 接口跟原生实现强对齐，跨语言 bug 编译期发现
- 同步调用——getter 可以同步返回，不强制 Promise
- 性能更好——JSI 调用、没有 Bridge 序列化

### 新架构下渲染流程具体长什么样？

老架构的渲染流程是：JS 算虚拟 DOM → JSON 序列化 → Bridge 传给 Native → Native 同步构建 view → 渲染。新架构重写了整条链路，理解它能解释 New Architecture 为什么启动更快、动画更稳定。

**新架构的三个角色**：

1. **React renderer**（JS 线程）：React 组件 tree → React shadow tree
2. **Fabric renderer**（C++ 层）：React shadow tree → 跨平台 shadow tree → 平台 view
3. **平台渲染层**（iOS UIKit / Android View）：实际绘制

**一次更新的链路**：

```
[JS 线程]
  React 组件 render
  ↓
  生成 element tree(JSX 结果)
  ↓
  Fiber reconciler diff,算出变化
  ↓
  通过 JSI 调用 C++ 创建 / 更新 Shadow Node

[C++ 层]
  Shadow tree 构建(平台无关的 view 描述)
  ↓
  Yoga 算 layout(flex 布局算法)
  ↓
  Commit 阶段:把新 tree atomically 替换旧 tree

[UI 线程]
  Mount 阶段:Diff shadow tree,生成平台操作指令
  ↓
  iOS: UIKit 更新 UIView; Android: View 系统更新 View
  ↓
  绘制
```

关键差异跟老架构：

- **同步 commit**：老架构是「JS 算完 → 异步 post 到 UI 线程 → UI 线程慢慢应用」，新架构是「JS commit 完成时 shadow tree 已经就绪，UI 线程立刻可以拿到」
- **C++ 层做布局**：老架构 layout 算在 Java/Obj-C 里，跨平台一致性靠人工对齐；新架构 Yoga 是 C++ 共享代码，iOS 和 Android 行为完全一致
- **支持并发渲染**：React 18 的 useTransition、Suspense 在新架构下能真正发挥——可以中断、可以丢弃低优 commit

**为什么新架构在低端机上提升更明显**：

老架构的 Bridge JSON 序列化对慢 CPU 影响很大——iPhone 15 Pro 序列化几百字节几乎无明显感知，但 Android 1GB RAM 的低端机能感受到每帧 5-10ms 的开销。新架构通过 JSI 直接传 C++ 对象引用，消除序列化开销，低端机上的掉帧会明显改善。

**实测对比**（开 New Arch vs 关）：

- 启动时间：减少 20-40%（视项目大小）
- 滚动 FPS：低端机从 30-40 → 55-60
- 内存：占用减少 10-20%

但开启新架构有兼容性成本——一些老库（不支持 TurboModule）跑不了，要等社区适配。0.76 起新项目默认开启，老项目升级谨慎评估。

### RN 应用的崩溃监控怎么做？

RN 崩溃分两类——JS 崩溃和 Native 崩溃，处理路径差异明显。

**JS 崩溃**：

`globalThis.ErrorUtils.setGlobalHandler` 是 RN 提供的全局错误处理：

```js
const defaultHandler = ErrorUtils.getGlobalHandler()

ErrorUtils.setGlobalHandler((error, isFatal) => {
  // 上报
  Sentry.captureException(error, {
    tags: { isFatal },
  })

  // 调用原 handler(让 RN 显示红屏 / 关闭)
  defaultHandler(error, isFatal)
})
```

未捕获的 Promise rejection 也要监听：

```js
require('promise/setimmediate/rejection-tracking').enable({
  allRejections: true,
  onUnhandled: (id, error) => {
    Sentry.captureException(error, { tags: { type: 'unhandled-rejection' } })
  },
})
```

**Native 崩溃**：

JS 监控抓不到原生层 crash（OOM、原生 SDK 崩、JSI 调用越界等）。要在 Native 层接入：

- **iOS**：用 PLCrashReporter / Sentry iOS SDK，处理 SIGSEGV / SIGABRT 等信号
- **Android**：用 Sentry Android SDK / Bugly，处理 JVM 异常 + native crash（NDK）

```bash
# 装 Sentry RN SDK,自动配置 iOS/Android 原生 SDK
npm install @sentry/react-native
npx @sentry/wizard@latest -i reactNative -p ios android
```

```js
// App.tsx
import * as Sentry from '@sentry/react-native'

Sentry.init({
  dsn: 'https://xxx@sentry.io/xxx',
  environment: __DEV__ ? 'development' : 'production',
  release: 'myapp@1.2.3',
  dist: '42',   // build number

  // 性能监控
  tracesSampleRate: 0.1,

  // attach 用户信息
  beforeSend(event) {
    event.user = { id: getCurrentUserId() }
    return event
  },
})

export default Sentry.wrap(App)
```

**source map / symbolication**：

JS 崩溃堆栈是 `main.bundle:1:123456` 这种，没用。要在 release build 时把 source map 上传到监控平台，让平台还原成源码堆栈：

```bash
# Sentry 自动上传
sentry-cli react-native xcode build.sh
```

iOS native 崩溃堆栈是 0x100abcdef 这种，要上传 dSYM 文件让平台用符号表还原。Android 用 Proguard mapping。

**常见崩溃模式**：

- **JS 层 NPE**：`undefined.toLowerCase()` 这种，TypeScript 能拦一大半
- **JSI 调用越界**：访问已销毁的 TurboModule，常见于热重载场景
- **OOM**：图片大量加载 + 没释放，长列表的 cell 没复用
- **Native SDK 崩**：第三方 SDK（地图、支付）的 bug，要找厂商修

**实战工具栈**：

中小项目：Sentry RN SDK（一站式覆盖 JS + Native）

大型项目：Sentry（JS）+ Bugly（Android Native）+ Firebase Crashlytics（iOS Native），不同维度交叉对比

### RN vs Capacitor vs Ionic 三个混合方案有啥本质区别？

经常被问「跨端方案怎么选」——RN 是其中一个选项。看清楚 Capacitor / Ionic 的定位，对比 RN 更清晰。

**RN（Meta）**：

- 渲染层：原生 view（iOS UIKit / Android View）
- 业务层：React 写 JS / TS
- 性能：接近原生
- 开发体验：要懂原生工程（Xcode / Gradle）
- 适合：要原生体验的 App、对性能敏感的业务

**Capacitor（Ionic 团队出的）**：

- 渲染层：WebView（iOS WKWebView / Android WebView）
- 业务层：任意前端（React / Vue / Angular）+ Capacitor 插件调原生
- 性能：取决于 WebView，比 RN 慢一截
- 开发体验：完全跟写 Web 一样，前端零门槛上手
- 适合：跨端 H5 + 想包成 App、对原生性能没强诉求

**Ionic（同公司另一产品）**：

- 在 Capacitor 之上的 UI 框架（提供 iOS / Material Design 风格的 web 组件）
- 解决「Web 写 App，怎么模仿原生 UI」
- 需要跟 Capacitor 或 Cordova 配合

**根本差异**：

```
[RN]                            [Capacitor + Ionic]
React → JS engine                React/Vue → WebView
↓ JSI                            ↓ JS Bridge
原生 View                        DOM
                                 ↓
                                 WebView 渲染
```

RN 用原生 view，UI 是真的原生组件——按钮就是 UIButton / android.widget.Button。Capacitor 是「Web 渲染 + 想办法模仿原生」。

**业务选型**：

- App 是核心产品、长期维护、追求体验 → RN 或 Flutter
- App 是 H5 的「壳」、不想做两套 → Capacitor
- 已有大量 Web 资产、想快速给用户一个 App → Capacitor
- 团队前端能力强、不想学原生 → Capacitor
- 团队能投入原生开发资源 → RN

**性能对比**（启动 + 列表滚动）：

- 原生：100 分基准
- Flutter：90-95 分
- RN（新架构）：80-90 分
- RN（老架构）：65-75 分
- Capacitor：50-65 分（取决于 WebView 性能）

但这数据存在口径差异——不少业务场景下 Capacitor 用户感知不到性能差，特别是内容型 App（新闻、电商展示）。游戏类、重交互类需要用 RN/Flutter。

从社区项目看：

24-25 年新项目里：

- 「我们已经有现成 React/Vue Web 应用」→ Capacitor 加个壳，1 周上线
- 「我们要做一个新的 App」→ RN 或 Flutter
- 「我们要做游戏 / 复杂交互应用」→ 原生 或 Unity / Cocos

理解每个方案的本质（渲染层用什么），选型就不用纠结了。

### RN 的动态化方案：EAS Update / CodePush 实战

App 走 App Store / Play 审核每次至少 1-7 天，发版后想紧急修 bug 等不起。RN 的 JS bundle 跟原生分离这个特性让「动态更新」成为可能——只更新 JS 部分，不走商店审核。

**CodePush 历史地位**：

CodePush 是微软出的方案，最早做 RN 动态更新。但**24 年 7 月起微软已经宣布 CodePush 进入维护模式，27 年彻底下线**。新项目千万别选它。

**EAS Update（推荐）**：

Expo 出的官方方案，跟 CodePush 概念一致但生态更新：

```bash
# 装 expo-updates
pnpm add expo-updates

# 配置 eas update
eas update:configure

# 发更新
eas update --branch production --message "fix login bug"
```

App 启动时按配置策略检查更新：

```ts
// app.config.js
{
  expo: {
    updates: {
      url: 'https://u.expo.dev/your-project-id',
      checkAutomatically: 'ON_LOAD',   // 启动时检查
      fallbackToCacheTimeout: 0,        // 不等待,先用旧版
    }
  }
}
```

**通道（channel）跟分支策略**：

```bash
# 生产用户
eas update --branch production

# 内测用户
eas update --branch staging

# 灰度 10% 用户
# (需要配合 EAS 后台的 rollout 控制)
```

业务里典型策略：

- main 分支 → staging channel（内测）
- release 标签 → production channel（生产）
- 紧急修复直接发 hotfix → production

**动态更新的边界**：

**能更新**:

- 所有 JS / TS 业务代码
- 图片、JSON 等 asset
- 大部分原生模块的 JS 调用代码

**无法更新**:

- 原生代码（iOS Swift/Obj-C、Android Kotlin/Java）
- App Icon、Launch Screen
- 新增的原生权限
- TurboModule 接口本身（接口签名变了要走商店发版）

**审核合规问题**:

App Store 对动态化更新有严格规定——只能修 bug、改样式，无法改核心功能、无法加新功能模块。违规的 App 被发现会下架。EAS Update 默认走的更新内容（JS bundle）符合规定，但「发个 0day 新功能」这种用法会触线。

**注意点**:

- **回滚要快**：发了有 bug 的更新立刻 `eas update --branch production --message revert`，新启动的 App 拿到回滚版本
- **强制更新场景**：业务有破坏性 bug（数据错乱、支付出错）时，配 `expo-updates` 的同步 API 强制等更新加载完才进入：

```ts
import * as Updates from 'expo-updates'

async function checkUpdate() {
  const update = await Updates.checkForUpdateAsync()
  if (update.isAvailable) {
    await Updates.fetchUpdateAsync()
    Updates.reloadAsync()   // 立即重启应用新版本
  }
}
```

- **存量用户跟踪**：发了更新后哪些用户已经更新了？EAS Dashboard 能看，国内自建方案要自己埋点

### RN 跟 SwiftUI / Jetpack Compose 对比怎么看？

RN 跟两端原生声明式 UI 框架（SwiftUI、Compose）的对比近两年讨论较多——SwiftUI / Compose 体验持续提升，「为什么还要 RN」的质疑也越来越多。

**对比框架**：

| 维度 | RN | SwiftUI | Compose |
|------|-----|---------|---------|
| 范式 | React 声明式 | Swift 声明式 | Kotlin 声明式 |
| 跨端 | iOS + Android + Web | iOS / iPadOS / macOS 等 Apple 全家 | Android + 桌面（KMP 复用） |
| 渲染 | 各端原生 view | UIKit / AppKit 之上自研 | Android View 之上自研 |
| 团队要求 | 会 React 就上手 | 要懂 Swift + iOS | 要懂 Kotlin + Android |
| 工程化 | npm 生态 | Apple 工具链 | Gradle / Android 工具链 |

**RN 仍有优势的场景**：

- **团队主要是 Web / 前端背景**：学 SwiftUI + Compose 等于学两门新语言 + 两套 SDK，时间成本巨大
- **跨端体验完全一致**：RN 一份代码 iOS + Android 几乎一样，原生方案要写两份且 UI 风格自然不一致
- **快速迭代 / 动态化**：EAS Update 让 RN 能绕过商店审核改 bug，原生没法
- **重 React 生态**：要用 Redux、React Query、Storybook，RN 复用

**SwiftUI / Compose 优势**：

- **性能上限更高**：完全原生，不存在 JS / 原生之间的桥接
- **系统集成平滑**：HealthKit、CarPlay、Wear OS 这类深度系统集成 RN 跟不上
- **苹果 / Google 重点投入**：每年 WWDC / Google I/O 都有大更新，新平台特性（visionOS、Wear OS）几乎只能用原生
- **跟设计语言贴近**：Material You / iOS 大圆角、动态字体这些跟系统紧密的设计 RN 模拟起来费劲

看现在大厂的实际做法：

- 25 年大趋势：**核心 App 走原生 + 部分页面用 RN/Flutter 嵌入**。比如美团、字节系 App 主体原生，活动页 / 营销页用 RN
- 创业公司 / 中小团队：仍以 RN / Flutter 为主，原生开发成本太高
- 海外大公司：Meta（RN 自家做的）、Discord、Microsoft Office 仍在大量用 RN
- 国内大公司：B 端工具类 / 内部 App 用 RN 较多，C 端核心 App 倾向原生 + 模块化嵌入

**Compose Multiplatform 是新变量**：JetBrains 把 Compose 扩展到 iOS + 桌面（24 年开始稳定），意味着 Kotlin 一份代码也能跨端，对 RN / Flutter 有冲击。但生态还不成熟，主流仍是 KMP（业务逻辑共享）+ Compose Android + SwiftUI iOS 的组合。

### RN 国际化（i18n）+ RTL 实战

RN 项目做国际化时比 Web 更复杂——除了文案，还要处理 RTL（阿拉伯语、希伯来语等右到左语言）的布局问题。

**i18n 库选型**：

- **i18next + react-i18next**：跟 Web 同一套，跨端复用方便
- **react-intl**（FormatJS）：Meta 出的，ICU MessageFormat 支持最好
- **Lingui**：编译时优化，bundle 更小

项目里常用 `i18next + react-i18next`，理由是 Web 和 RN 项目能共享一份翻译文件。

```js
// i18n/index.ts
import i18n from 'i18next'
import { initReactI18next } from 'react-i18next'
import { getLocales } from 'expo-localization'

const locales = getLocales()
const defaultLang = locales[0]?.languageCode || 'en'

i18n.use(initReactI18next).init({
  resources: {
    en: { translation: require('./locales/en.json') },
    zh: { translation: require('./locales/zh.json') },
    ar: { translation: require('./locales/ar.json') },
  },
  lng: defaultLang,
  fallbackLng: 'en',
  interpolation: { escapeValue: false },
})
```

```jsx
import { useTranslation } from 'react-i18next'

function HomeScreen() {
  const { t, i18n } = useTranslation()

  return (
    <View>
      <Text>{t('welcome', { name: 'Alice' })}</Text>
      <Text>{t('itemCount', { count: 5 })}</Text>
      <Button onPress={() => i18n.changeLanguage('ar')} title="عربي" />
    </View>
  )
}
```

**RTL 处理（关键）**：

阿拉伯语、希伯来语等语言是「右到左」（Right To Left）。RTL 布局下：

- 文本对齐方向反转
- 图标位置左右翻转（返回按钮原来在左、RTL 下应该在右）
- padding/margin 的 left/right 语义反转

```js
import { I18nManager } from 'react-native'

// 检测当前是不是 RTL
const isRTL = I18nManager.isRTL

// 强制切换 RTL
I18nManager.forceRTL(true)
// 切换后需要重启 App 才生效
```

**Style 写法**：

```js
// ❌ 老写法:写死 left/right,RTL 下错
{ marginLeft: 16 }

// ✅ 新写法:用 start/end,自动跟随 RTL
{ marginStart: 16 }   // LTR 下等于 marginLeft,RTL 下等于 marginRight
```

RN 0.66+ 支持 `start/end`、`paddingStart/End`、`borderStartWidth` 等：

```js
const styles = StyleSheet.create({
  card: {
    paddingStart: 16,    // RTL 自动适配
    paddingEnd: 16,
    borderStartWidth: 1,
    borderStartColor: '#ddd',
  }
})
```

**图标翻转**：

```jsx
<Image
  source={require('./back-arrow.png')}
  style={{
    transform: [{ scaleX: I18nManager.isRTL ? -1 : 1 }]   // RTL 下水平翻转
  }}
/>
```

**复数 / 性别 / 数字格式**：

```json
// en.json
{
  "itemCount": "{{count}} item",
  "itemCount_other": "{{count}} items"
}

// zh.json
{
  "itemCount": "{{count}} 个商品"  // 中文不区分单复数
}

// ar.json (阿拉伯语有六种复数形式)
{
  "itemCount_zero": "لا توجد عناصر",
  "itemCount_one": "عنصر واحد",
  "itemCount_two": "عنصران",
  "itemCount_few": "{{count}} عناصر",
  "itemCount_many": "{{count}} عنصرًا",
  "itemCount_other": "{{count}} عنصر"
}
```

i18next 自动按当前语言的规则选对的版本。

**日期 / 货币 / 数字格式**：

用 `Intl.NumberFormat`、`Intl.DateTimeFormat`（Hermes 74+ 完整支持）：

```js
new Intl.NumberFormat('ar-EG', { style: 'currency', currency: 'EGP' })
  .format(1234.56)   // "١٬٢٣٤٫٥٦ ج.م.‏"

new Intl.DateTimeFormat('zh-CN', { dateStyle: 'full' })
  .format(new Date())   // "2025年5月13日星期二"
```

### RN 推送通知怎么接：FCM / APNs / 国内推送服务

App 推送是多数应用需要处理的功能。RN 接推送表面上简单，实际涉及客户端 SDK + 服务端推送 + 平台限制三方协调。

**推送的整体模型**：

```
你的服务器
  ↓ (POST /push,带 device token)
推送服务(FCM / APNs / 国内厂商)
  ↓
用户的 App(收到通知)
```

`device token`/`registration token` 是每个设备上 App 的唯一标识——App 启动时拿这个 token 上报给你的服务器存起来，发推送时用它。

**iOS:APNs (Apple Push Notification service)**:

苹果官方推送通道。App 拿 token 要先申请「Push Notifications」capability + 用户授权。

```jsx
import * as Notifications from 'expo-notifications'

async function registerForPush() {
  const { status } = await Notifications.requestPermissionsAsync()
  if (status !== 'granted') return null

  const token = (await Notifications.getDevicePushTokenAsync()).data
  // 把 token 发给你的服务器
  await api.registerDevice(token)
}
```

服务端走 APNs 协议发推送（HTTP/2 + JWT 鉴权），或者用 Firebase Cloud Messaging 作为中间层（FCM 替你转发给 APNs）。

**Android:FCM (Firebase Cloud Messaging)**:

Google 的推送通道。国内特殊——FCM 在国内无法直接用，因为依赖 Google 服务（Google Play Services）。海外项目用 FCM 作为统一方案，国内项目通常要接入「华为推送 / 小米推送 / vivo 推送 / OPPO 推送 / 个推」等通道。

国内的常见方案：

- **个推 / 极光推送 / 友盟**：第三方推送聚合服务,一个 SDK 集成所有国内厂商
- **华为 / 小米 / vivo / OPPO 各自推送**:在自家手机上后台保活更稳定,所以国内做法是「在华为手机用华为推送、小米手机用小米推送…」最大化送达率
- **uniPush(uni-app)/原生集成**:对应 RN 都有相应库

**RN 集成路径**：

1. **简单场景(海外或纯 FCM)**:`expo-notifications` 一站完成,iOS + Android 都覆盖
2. **国内多厂商**:`react-native-push-notification` + 国内厂商 SDK 桥接,或者直接用「极光推送 / 友盟」的 RN SDK
3. **完全自建**:基于 `@notifee/react-native` 处理通知 UI,推送通道自己接

**推送内容怎么处理**:

收到推送有两种情况:

- **App 在前台**:推送默认不显示通知,要 App 自己决定怎么处理(显示 banner / 静默)
- **App 在后台 / 进程被系统终止**:推送由系统显示,用户点了打开 App + 跳转对应页面

跳转一般通过 deeplink:

```js
Notifications.addNotificationResponseReceivedListener((response) => {
  const { screen, params } = response.notification.request.content.data
  navigation.navigate(screen, params)
})
```

服务端发推送时带上业务数据:

```json
{
  "title": "您有新订单",
  "body": "订单 #12345 已支付",
  "data": {
    "screen": "OrderDetail",
    "params": { "id": "12345" }
  }
}
```

**问题点**:

- **iOS 用户需要主动授权**:第一次启动弹「允许推送通知」,用户拒绝后再要的话只能引导到设置页
- **国内 Android 厂商后台策略**:推送送达率跟厂商手机型号强相关,vivo / OPPO 后台进程管理较严格,推送可能无法送达
- **静默推送**:`content-available` payload 让 App 后台收到推送时执行代码(同步数据),但 iOS 限制了静默推送频率,滥用会被掐
- **本地通知**:不需要服务端推送的场景(定时提醒、闹钟)用本地通知,`Notifications.scheduleNotificationAsync` 一行完成

### RN 接蓝牙 / 蓝牙 IoT 实战

物联网 / 健身设备 / 智能家居 / 医疗设备这类项目经常要接蓝牙。RN 生态里 `react-native-ble-plx` 是主流。

```bash
pnpm add react-native-ble-plx
# iOS 跟 Android 都要在原生配置文件加权限
```

```js
import { BleManager } from 'react-native-ble-plx'

const manager = new BleManager()

// 扫描设备
manager.startDeviceScan(null, null, (error, device) => {
  if (error) return console.error(error)
  if (device.name === 'MyDevice') {
    manager.stopDeviceScan()
    connectToDevice(device)
  }
})

// 连接 + 读写特征值
async function connectToDevice(device) {
  const connected = await device.connect()
  await connected.discoverAllServicesAndCharacteristics()

  // 读
  const char = await connected.readCharacteristicForService(
    'service-uuid', 'characteristic-uuid'
  )
  console.log(char.value)   // base64 编码的数据

  // 写
  await connected.writeCharacteristicWithResponseForService(
    'service-uuid', 'characteristic-uuid', base64Data
  )

  // 订阅通知
  connected.monitorCharacteristicForService(
    'service-uuid', 'characteristic-uuid',
    (error, char) => {
      if (char?.value) console.log('收到数据:', char.value)
    }
  )
}
```

**实战要点**：

- **iOS 权限**：需要在 Info.plist 加 `NSBluetoothAlwaysUsageDescription`、`NSBluetoothPeripheralUsageDescription` 说明
- **Android 权限**：12+ 起需要 `BLUETOOTH_SCAN`、`BLUETOOTH_CONNECT` 运行时权限
- **状态机管理**：「未连接 → 扫描中 → 已连接 → 通信中 → 已断开」每个状态都要处理 UI
- **断线重连**：蓝牙设备容易断（出范围、设备休眠），要写重连逻辑
- **协议解析**：拿到的是 base64 字节，要按设备厂商提供的协议解析

新架构下：`react-native-ble-plx` 4+ 兼容 Fabric / TurboModule。

**Bluetooth Mesh / Beacon**：

更复杂的蓝牙场景（一个手机连多个设备形成 mesh、室内定位 beacon）社区库支持不全，可能要写自己的原生模块。

### RN 视频 / 音频处理

电商 / 内容 / 社交类 App 经常要播视频。RN 视频生态:

**react-native-video**:

最经典的视频播放器，社区版本一直在维护：

```jsx
import Video from 'react-native-video'

<Video
  source={{ uri: 'https://example.com/video.mp4' }}
  style={{ width: '100%', height: 200 }}
  controls
  resizeMode="contain"
  onLoad={(data) => console.log('时长:', data.duration)}
  onProgress={(data) => console.log('进度:', data.currentTime)}
  onEnd={() => console.log('播完')}
/>
```

支持：

- HLS / DASH 流媒体
- 字幕（`<track>` 类似的 API）
- 全屏、画中画（iOS）
- 后台播放配置

**expo-av / expo-video**：

Expo SDK 50+ 起把视频拆成 `expo-video`（新）和老的 `expo-av`。`expo-video` 用了原生 video 控件，性能更好、API 更现代：

```jsx
import { VideoView, useVideoPlayer } from 'expo-video'

function VideoScreen() {
  const player = useVideoPlayer('https://example.com/video.mp4', (player) => {
    player.loop = true
    player.play()
  })

  return (
    <VideoView
      player={player}
      style={{ width: '100%', height: 200 }}
      contentFit="contain"
      nativeControls
    />
  )
}
```

新项目使用 Expo 时可以考虑 expo-video，接入成本比 react-native-video 更低。

**音频**：

- **expo-av**：基础音频播放、录音
- **react-native-track-player**：音乐 App 类（带播放列表、后台播放、锁屏控制）
- **react-native-audio-recorder-player**：录音 + 播放

**视频编辑**：

复杂场景（剪辑、裁剪、加滤镜）RN 生态弱，要么自己接原生（iOS AVFoundation、Android MediaCodec），要么用商业 SDK（Banuba、Filtric）。

**短视频流（抖音风格）**：

- `react-native-video` + `FlatList` 滚动切换
- 关键：预加载下一个视频、上一个视频暂停 + 卸载、内存控制
- 上层 UI（点赞、评论、关注按钮）跟 video 用 absolute 叠

国内场景下 React Native 承载短视频核心页面的上限不如原生——内存控制、动画流畅度都有差距。因此国内大厂的短视频核心页面通常仍采用原生实现。
