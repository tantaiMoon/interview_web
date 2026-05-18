[toc]



## 一、TypeScript 基础认知

### 说说你对 TypeScript 的理解?与 JavaScript 的区别是什么?

JavaScript 是动态弱类型语言,类型错误要等到**运行时**才暴露。TypeScript 在 JavaScript 之上加一层**编译期静态类型系统**,把"运行时才失败"的问题提前到"写代码时就报错"——用类型信息换可维护性。

几个关键特征：

1. **超集**:任何合法的 JS 都是合法的 TS,渐进迁移没成本。
2. **类型擦除**:TS 编译时移除所有类型注解,产物是纯 JS,运行时**没有任何类型信息**(所以不能 `typeof` 判断 TS 接口)。
3. **类型推导**:不写注解时,TS 也会根据上下文反推类型,大量场景下无需手写。
4. **结构化类型(Structural Typing)**:类型兼容看"形状",不看名字——只要字段对得上就兼容。

跟 JS 对比起来：

| 维度 | JavaScript | TypeScript |
|------|-----------|-----------|
| 类型检查 | 运行时 | 编译期 |
| 错误发现 | 上线后用户暴露 | 写代码时 IDE 标红 |
| 大型项目协作 | 重构靠搜索 | 重构靠类型驱动 |
| 学习成本 | 低 | 中 |
| 运行时性能 | 直接运行 | 需编译,产物等同 JS |
| 生态 | 无类型 | DefinitelyTyped 提供 `@types/*` |

工程里一般这么处理：

1. 新项目直接上 TS,迁移老项目用 `allowJs` + `checkJs` 渐进改造。
2. `tsconfig.json` 开 `strict: true`,享受类型系统的全部红利;关掉等于白用。
3. 不要用 `// @ts-ignore` 当通用兜底手段,它只解决报错,不解决问题——优先用类型收窄或断言。

### 说说 TypeScript 的数据类型有哪些?

TS 类型系统在 JS 的 7 种基础类型之上,加了一组"描述类型本身行为"的特殊类型(`any` / `unknown` / `never` / `void`),用来表达**未知**、**不可能**、**无返回**等语义。

清单：

1. **基础类型**:对应 JS 的原始值:`string`、`number`、`boolean`、`null`、`undefined`、`symbol`、`bigint`。
2. **特殊类型**:
   - `any`:**关掉类型检查**,赋值给任何类型都行(危险)。
   - `unknown`:类型安全的 `any`,**用之前必须收窄**。
   - `void`:函数无返回值。
   - `never`:**永远不会有值**(抛错、死循环、被穷尽的分支)。
3. **复合类型**:数组 `T[]`、元组 `[T, U]`、对象、函数。
4. **引用相关**:`interface`、`type`、`class`、`enum`。

`any` / `unknown` / `never` 这三个容易混淆,看一下它们的赋值方向差异：

| 类型 | 能否赋值给它 | 它能否赋值给别人 | 典型用途 |
|------|------------|----------------|---------|
| `any` | ✅ 任何类型 | ✅ 任何类型 | 临时绕过类型(慎用) |
| `unknown` | ✅ 任何类型 | ❌ 必须先收窄 | 三方数据入口、`JSON.parse` |
| `never` | ❌ 没有值能赋给它 | ✅ 任何类型 | 抛错函数、穷尽检查 |

`never` 常见的用法是穷尽检查：

```ts
type Status = 'pending' | 'success' | 'fail'

function handle(s: Status) {
  switch (s) {
    case 'pending': return 0
    case 'success': return 1
    case 'fail':    return 2
    default:
      // 漏写一个 case,这里 s 就不再是 never,编译报错
      const _exhaustive: never = s
      return _exhaustive
  }
}
```

工程上注意:三方接口、`JSON.parse`、`localStorage` 取值这些"边界数据"用 `unknown`,内部代码用具体类型;用 `never` 做联合类型穷尽检查,新增枚举值时编译器自动告诉你哪里漏处理了;不要把 `void` 当 `undefined` 用——`void` 表示"调用者不应该关心返回值"。

## 二、类型系统与核心能力

### 说说你对 TypeScript 中枚举类型的理解?应用场景是什么?

枚举是给一组**有限的常量**起一个有语义的名字,避免代码里到处出现 `0`、`1`、`'PENDING'` 这种"魔法值"。但 TS 的 `enum` 编译后会生成**额外运行时代码**(双向映射对象),不像类型那样被擦除,这是它和"字符串字面量联合类型"最大的区别。

具体有这么几类：

1. **数字枚举**:不指定值时从 0 开始递增,会生成正反向映射(`Color[0] === 'Red'`)。
2. **字符串枚举**:必须显式赋值,只有正向映射,可读性更好。
3. **常量枚举(`const enum`)**:编译时**完全内联**,不生成运行时对象,体积最小。
4. **异构枚举**:数字字符串混用,**不推荐**。

跟「字符串字面量联合类型」对比一下,差异挺明显：

| 维度 | `enum` | 联合字面量 |
|------|--------|-----------|
| 运行时产物 | 有(对象) | 无(类型擦除) |
| 反向查值 | 数字枚举支持 | 不支持 |
| 摇树优化 | 较差 | 好 |
| 可读性 | `Color.Red` | `'red'` |
| 跨文件复用 | 直接 import | 写 `as const` |

```ts
// 数字枚举
enum Status { Pending, Success, Fail }
console.log(Status.Pending)       // 0
console.log(Status[0])            // 'Pending'(反向映射)

// 字符串枚举(推荐)
enum Role {
  Admin = 'admin',
  User = 'user',
}

// 常量枚举(编译期内联,产物 0 字节)
const enum Direction { Up, Down }
const d = Direction.Up            // 编译为 const d = 0

// 联合字面量(不少团队的首选)
type Status2 = 'pending' | 'success' | 'fail'
```

实操上：新项目优先用**字符串字面量联合类型**,体积小、Tree-shaking 友好、调试看到的就是字面量本身;如果需要用 `enum` 选**字符串枚举**,日志里出现 `'admin'` 比 `0` 容易排查;`const enum` 跨包发布会被 Babel 等工具误解析,共享库慎用——可改用 `as const` 对象。

### 说说你对 TypeScript 中接口的理解?应用场景是什么?

接口(`interface`)是描述"对象形状"的契约。TS 用**结构化类型**判断兼容性——一个对象只要"长得像"接口,就算兼容,不需要显式 `implements`。这种宽松的契约让接口成为前端**模块边界的天然描述工具**。

接口能干这些事：

1. **描述对象结构**:字段 + 类型。
2. **可选与只读**:`prop?: T`、`readonly prop: T`。
3. **索引签名**:`[key: string]: T`,描述动态键的对象。
4. **函数类型**:`interface Fn { (x: number): string }`。
5. **可合并(Declaration Merging)**:同名 interface 自动合并字段——`type` 不能。
6. **继承**:`interface B extends A`。

跟 `type` 别名对比：

| 维度 | `interface` | `type` |
|------|------------|--------|
| 描述对象 | ✅ | ✅ |
| 联合 / 交叉类型 | ❌ | ✅ |
| 元组 / 字面量 | ❌ | ✅ |
| 同名合并 | ✅(自动) | ❌(报错) |
| 继承语法 | `extends` | `&` 交叉 |
| 性能(大型项目) | 略快 | 略慢 |

```ts
interface User {
  readonly id: number
  name: string
  age?: number
  [key: string]: unknown   // 索引签名:允许任意额外字段
}

interface User { email: string }  // 自动合并,User 现在多了 email 字段

interface Admin extends User { permissions: string[] }
```

工程上的分工简单:描述对象/类用 `interface`,描述联合、元组、工具类型用 `type`;三方库扩展(给 `Window` 加属性、给 `Vue` 加 `$xxx`)只能用 `interface` 的合并能力;接口里的方法名拼错和实际实现对不上排查成本很高——开 `noImplicitOverride` + 单元测试兜底。

### 说说你对 TypeScript 中类的理解?应用场景是什么?

TS 的 class 在 ES class 之上加了**访问修饰符、抽象类、参数属性、实现接口**等能力,让面向对象建模在 TS 中真正可落地。但前端业务里"组件 + hook"的函数式风格更主流,class 的核心战场是**SDK、领域模型、Node 后端**。

具体能力：

1. **访问修饰符**:`public`(默认)、`private`、`protected`——只在编译期检查,运行时不生效;ES 私有字段 `#prop` 才是运行时硬隔离。
2. **`readonly`**:实例属性只读。
3. **抽象类 `abstract`**:不能直接 `new`,只能被继承。
4. **参数属性**:构造函数参数加修饰符,自动赋值给实例,省样板代码。
5. **`implements` vs `extends`**:`implements` 只校验结构,不继承实现;`extends` 继承父类。

```ts
abstract class Animal {
  abstract speak(): void          // 子类必须实现

  // 参数属性:等价于 this.name = name
  constructor(public readonly name: string) {}
}

class Dog extends Animal {
  #secret = 'bone'                // ES 私有,运行时也访问不到
  speak() { console.log(`${this.name}: woof`) }
}

interface Logger { log(msg: string): void }
class ConsoleLogger implements Logger {
  log(msg: string) { console.log(msg) }
}
```

几条经验：优先 ES 私有字段 `#prop`,它提供真正的运行时隔离;`private` 只是编译期约束,被强转后仍可访问;业务代码优先使用函数和 hook,class 更适合**有状态机、需要继承、生命周期复杂**的场景(比如 SDK);不要在 class 里塞太多静态方法——那本质是命名空间下的工具函数,直接用 `export function` 更轻。

### 说说你对 TypeScript 中函数的理解?与 JavaScript 函数的区别是什么?

TS 函数运行时还是 JS 函数,差别只在**签名层**:把"参数怎么传、返回什么、有哪几种重载形态"用类型固定下来,调用方通过类型即可了解使用方式,不需要先阅读实现。

能力清单：

1. **参数/返回值标注**:`function f(x: number): string`。
2. **可选 / 默认 / 剩余参数**:`x?: T`、`x: T = 1`、`...args: T[]`。
3. **函数重载**:多个签名 + 一个实现,提供更精确的类型推导。
4. **函数类型字面量**:`type Fn = (x: number) => string`。
5. **`this` 类型**:第一个参数声明 `this: SomeType`,运行时不传,只供类型校验。

重载是最有意思的场景:

```ts
// 重载签名(暴露给外部)
function parse(input: string): object
function parse(input: string, raw: true): string
// 实现签名(对外不可见)
function parse(input: string, raw?: boolean): object | string {
  return raw ? input : JSON.parse(input)
}

const obj = parse('{}')           // 推导为 object
const str = parse('{}', true)     // 推导为 string
```

公共 API 优先重载,不要靠"返回联合类型 + 调用方断言",体验差;回调函数签名尽量声明返回类型,避免实现里偷偷 `return` 不该返回的值导致联合类型扩散;函数泛型默认值善用,降低调用方理解成本:`function fetch<T = unknown>(): Promise<T>`。

### 说说你对 TypeScript 中泛型的理解?应用场景是什么?

泛型解决的是"**复用**"和"**类型安全**"的矛盾。没有泛型时,通用工具只能写 `any` 牺牲类型;有了泛型,类型变成"参数",调用时再具体化,做到"一次定义,随类型变化"。

几种常见形态：

1. **泛型函数**:`function id<T>(x: T): T`,T 在调用时确定。
2. **泛型接口/类**:`interface Box<T> { value: T }`。
3. **泛型约束(`extends`)**:限制 T 必须满足某种结构。
4. **默认泛型**:`T = string`,不传时取默认值。
5. **多泛型协作**:`<K extends keyof T>`,实现属性级类型安全。

```ts
// 取对象属性,key 必须是 obj 真实存在的键
function pick<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
const u = { id: 1, name: 'Tom' }
pick(u, 'id')      // ✅ 返回 number
pick(u, 'age')     // ❌ 编译报错:'age' 不是 keyof typeof u

// 通用请求封装
async function request<T>(url: string): Promise<T> {
  const res = await fetch(url)
  return res.json() as T
}
const list = await request<User[]>('/api/users')   // list 自动推为 User[]
```

不要为了用泛型而用泛型——只在"类型随调用变化"时引入,否则只是噪音;给泛型加约束(`<T extends Record<string, unknown>>`),否则 T 是 `unknown`,调用方写出来还是没用;泛型命名:单字母用于"通用占位"(`T`、`K`、`V`、`R`),多字母用于"语义明确"(`TProps`、`TData`)。

### 说说你对 TypeScript 中高级类型的理解?有哪些?

高级类型让 TS 从"类型标注工具"升级为"**类型计算工具**"——可以基于已有类型推导、过滤、变换出新类型,实现"类型驱动开发"。其核心能力是 `keyof` + `extends` + `infer` 三件套。

能力清单：

1. **联合类型 `A | B`**:或关系。
2. **交叉类型 `A & B`**:与关系(对象合并)。
3. **字面量类型 `'a' | 'b'`**:精确到值。
4. **条件类型 `T extends U ? X : Y`**:类型层面的 if-else。
5. **映射类型 `{ [K in keyof T]: ... }`**:遍历键改类型。
6. **`keyof T`**:取所有键的联合。
7. **`typeof v`**:取变量的类型。
8. **`infer R`**:在条件类型中"声明并推导"一个新变量。
9. **内置工具类型**:`Partial<T>`、`Required<T>`、`Pick<T,K>`、`Omit<T,K>`、`Record<K,V>`、`ReturnType<F>`、`Parameters<F>`、`Awaited<P>`。

常用工具类型一表通：

| 工具类型 | 作用 | 等价实现 |
|---------|------|---------|
| `Partial<T>` | 所有字段可选 | `{ [K in keyof T]?: T[K] }` |
| `Required<T>` | 所有字段必填 | `{ [K in keyof T]-?: T[K] }` |
| `Readonly<T>` | 所有字段只读 | `{ readonly [K in keyof T]: T[K] }` |
| `Pick<T, K>` | 挑选指定键 | `{ [P in K]: T[P] }` |
| `Omit<T, K>` | 排除指定键 | `Pick<T, Exclude<keyof T, K>>` |
| `Record<K, V>` | 构造键值对类型 | `{ [P in K]: V }` |
| `ReturnType<F>` | 取函数返回类型 | `F extends (...a: any) => infer R ? R : never` |

```ts
type User = { id: number; name: string; age: number }

// Partial:全字段可选,常用于 patch 更新
type UserPatch = Partial<User>

// Pick:挑选字段,常用于响应模型
type UserPreview = Pick<User, 'id' | 'name'>

// 用 infer 自定义工具:取 Promise 的内部类型
type Unwrap<T> = T extends Promise<infer U> ? U : T
type R = Unwrap<Promise<string>>   // string

// 条件 + 映射:把所有方法名挑出来
type FunctionKeys<T> = {
  [K in keyof T]: T[K] extends Function ? K : never
}[keyof T]
```

优先用内置工具类型,可读性通常比自己写 `infer` 更清晰,不够再自定义;类型嵌套过深时要评估是否存在更简单的表达方式,难以维护的复杂类型会增加长期成本;复杂类型建议加注释和类型测试(用 `expectTypeOf` 或 `tsd`),否则后续改动容易引发连锁问题。

## 三、进阶能力与工程落地

### 说说你对 TypeScript 装饰器的理解?应用场景是什么?

装饰器是给类、方法、属性、参数加**额外行为或元数据**的语法糖。它就是个**高阶函数**——接收原类/方法,返回增强后的版本。装饰器解决的是"在不修改业务代码的前提下加横切逻辑"(日志、权限、依赖注入)。

几种形态：

1. **类装饰器**:接收构造函数,返回新构造函数。
2. **方法装饰器**:接收 `(target, key, descriptor)`,可改写方法。
3. **属性装饰器**:常配合 `reflect-metadata` 收集类型元信息。
4. **参数装饰器**:常用于参数注入(NestJS 的 `@Body()`)。
5. **执行顺序**:同一目标多个装饰器,**自下而上**执行。

```ts
// 旧装饰器(experimentalDecorators):NestJS、TypeORM 用这套
function Log(target: any, key: string, desc: PropertyDescriptor) {
  const orig = desc.value
  desc.value = function (...args: any[]) {
    console.log(`call ${key}`, args)
    return orig.apply(this, args)
  }
}

class Service {
  @Log
  fetch(id: number) { return id }
}

// TS 5.0+ 标准装饰器(Stage 3 提案)语法:
function log<This, Args extends any[], R>(
  target: (this: This, ...args: Args) => R,
  ctx: ClassMethodDecoratorContext
) {
  return function (this: This, ...args: Args): R {
    console.log(`call ${String(ctx.name)}`, args)
    return target.call(this, ...args)
  }
}
```

装饰器在哪些地方用得多：

| 场景 | 典型框架/库 |
|------|-----------|
| 依赖注入 | NestJS、Angular |
| ORM 映射 | TypeORM、MikroORM |
| 路由声明 | NestJS、midway |
| 接口校验 | class-validator |
| 埋点 / 日志 | 自研 AOP |

区分两套装饰器:**老 `experimentalDecorators` + `emitDecoratorMetadata`** 和 **TS 5.0 标准装饰器**,语法和行为不一样,确认目标框架支持哪套;装饰器常需要 `reflect-metadata`,需要在入口顶部 `import 'reflect-metadata'`;装饰器写多了会让控制流难以追踪,写之前先问"普通函数能不能做"——能用普通函数解决时优先用普通函数。

### 说说对 TypeScript 中命名空间与模块的理解?区别是什么?

命名空间(namespace)是 TS 早期、ES Module 规范还没普及时的"全局变量隔离方案",通过包一层闭包对象避免命名冲突。模块(module)是 ES Module 规范下的标准代码组织方式。**两者只在历史上并存,新项目应该只用模块**。

具体差异：

1. **命名空间**:`namespace Foo {}`,编译为一个全局对象 `Foo`。
2. **模块**:`export` / `import`,文件即模块,作用域天然隔离。
3. 命名空间可以跨文件用 `/// <reference path>` 拼接(脆弱)。
4. 模块走打包工具的依赖图,Tree-shaking 友好。

| 维度 | 命名空间 | 模块 |
|------|---------|------|
| 隔离方式 | 全局对象 | 文件作用域 |
| 引入方式 | 全局可见 / `<reference>` | `import` |
| Tree-shaking | 支持较差 | 支持较好 |
| 现代构建工具 | 几乎不支持 | 原生支持 |
| 推荐使用 | ❌ 仅遗留代码 | ✅ |

```ts
// 命名空间(不推荐,仅老项目)
namespace Util {
  export function add(a: number, b: number) { return a + b }
}
Util.add(1, 2)

// 模块(推荐)
// util.ts
export function add(a: number, b: number) { return a + b }
// main.ts
import { add } from './util'
```

新项目一律用模块,`.d.ts` 文件里也优先用 `declare module`,不要用 `declare namespace`;老项目从命名空间迁移到模块时,先把所有 `namespace` 改成单独文件 + `export`,再逐步替换 reference 路径;例外:为浏览器全局变量补声明(如 `declare namespace JSX {}`)还得用命名空间,这是规范规定。

### 说说如何在 React 项目中应用 TypeScript?

React + TS 的核心是把"组件作为契约"——`props`、`state`、`event`、`ref`、`context` 全部纳入类型系统,让组件之间的依赖在编译期可校验,重构时类型驱动。

具体几个方向：

1. **组件 props 类型**:函数组件用 `React.FC<Props>` 或直接 `function C(props: Props)`(更推荐,后者不强制 children)。
2. **事件类型**:`React.ChangeEvent<HTMLInputElement>`、`React.MouseEvent<HTMLButtonElement>`。
3. **Hooks 类型**:`useState<T>(init)`、`useRef<T>(null)`、自定义 hook 加返回类型。
4. **Context**:`createContext<T | null>(null)`,消费侧做空值收窄。
5. **泛型组件**:`function List<T>(props: { items: T[] })`。
6. **组件库 props**:用 `ComponentProps<typeof Button>` 复用三方组件类型。

```tsx
type Props = {
  title: string
  onSelect?: (id: number) => void
  children?: React.ReactNode
}

function Card({ title, onSelect, children }: Props) {
  const [count, setCount] = useState<number>(0)
  const inputRef = useRef<HTMLInputElement>(null)

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setCount(Number(e.target.value))
  }
  return <div>{title}{children}</div>
}

// 泛型组件:列表通用化
function List<T>({ items, render }: { items: T[]; render: (x: T) => React.ReactNode }) {
  return <ul>{items.map((it, i) => <li key={i}>{render(it)}</li>)}</ul>
}
```

写 React + TS 时要注意:不推荐 `React.FC`——它强制带上 `children` 类型,且对默认参数推导有问题,直接 `function Comp(props: Props)` 更清晰;`useRef` 给 DOM 用 `useRef<HTMLDivElement>(null)`,给可变值用 `useRef<number>(0)`,两者初始值含义不同;三方组件 props 用 `ComponentProps<typeof Button>` 复用,不要手抄一遍。

### 说说如何在 Vue 项目中应用 TypeScript?

Vue3 的组合式 API 设计时就考虑了 TS,`ref`、`reactive`、`computed` 都能精确推导。`<script setup lang="ts">` 用编译宏(`defineProps` / `defineEmits`)实现"写类型即声明",让 TS 在 Vue 项目里几乎低成本。

主要能力：

1. **`defineProps<T>()`**:用泛型直接声明 props 类型,编译后自动转成运行时声明。
2. **`defineEmits<T>()`**:声明事件载荷类型。
3. **`ref<T>()` / `reactive<T>()`**:精确推导。
4. **`computed`**:返回类型由计算函数自动推导。
5. **`provide` / `inject`**:用 `InjectionKey<T>` 携带类型信息。
6. **路由元信息**:扩展 `RouteMeta` 接口,所有路由 `meta` 都有类型。

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface Props {
  title: string
  size?: 'sm' | 'md' | 'lg'
}
const props = withDefaults(defineProps<Props>(), { size: 'md' })

const emit = defineEmits<{
  (e: 'select', id: number): void
  (e: 'close'): void
}>()

const count = ref<number>(0)
const double = computed(() => count.value * 2)
</script>
```

项目配置可以这样写：SFC 一律 `<script setup lang="ts">`,用 `defineProps<T>()` 而非运行时声明,体验维护成本更低;跨组件共享类型放 `types/` 目录或单独包,不要在每个组件里重复声明;Pinia 的 store 用 setup 风格(组合式)写,类型推导基本不用额外写,Options 风格在大型 store 上类型丢失严重;`vue-router` 的 `meta` 加全局声明:`declare module 'vue-router' { interface RouteMeta { ... } }`,路由 `meta` 都能有提示。

## 补充题

### `any`、`unknown`、`never` 有什么区别？

- **`any`**：放弃类型检查，赋什么都行、调什么都不报错。基本等于回到 JS 状态，工程里能不用就不用。
- **`unknown`**：类型安全版的 `any`——可以接收任何值，但**用之前必须先收窄**（`typeof`、`instanceof`、自定义类型守卫）。`JSON.parse` / `fetch().json()` / 三方接口出来的数据用 `unknown` 接较稳。
- **`never`**：永远不会出现的值。抛异常的函数、死循环、被穷尽的联合分支都是 `never`。最典型用法是穷尽检查（switch 漏 case 时编译期就报错）。

记忆点：`any` 是"我不管"，`unknown` 是"我先收窄再用"，`never` 是"这里根本不可能"。

### 什么是声明合并（Declaration Merging）？

同名的 `interface`、`namespace`、`function` 等声明会被 TS 自动合并成一个。

```ts
interface Window { __DEV__: boolean }       // 给全局 Window 加字段
interface Window { __VERSION__: string }
// 合并后 Window 同时拥有 __DEV__ 和 __VERSION__
```

常见用的场景：

- **扩展第三方类型**：给 Vue 的 `ComponentCustomProperties` 加 `$http`、给 Express 的 `Request` 加 `user`
- **声明全局变量**：`declare global { interface Window { ... } }`
- **模块扩展（Module Augmentation）**：`declare module 'vue-router' { interface RouteMeta { ... } }`

注意 `type` 别名**不能合并**，只能用 `interface`。

### `interface` 和 `type` 怎么选？

90% 的场景能互换。具体差异：

| 维度 | `interface` | `type` |
|------|------------|--------|
| 描述对象 / 类形状 | ✅ | ✅ |
| 联合 / 交叉 | ❌ | ✅（`A \| B`、`A & B`） |
| 元组 / 字面量 | ❌ | ✅ |
| 声明合并 | ✅ | ❌ |
| 扩展（继承） | `extends` | `&` 交叉 |
| 性能 | 略好（TS 内部 cache 友好） | 复杂 type 会慢 |

工程上建议：**对象形状用 `interface`，需要联合 / 工具类型计算用 `type`**。给三方库做类型扩展必须用 `interface`。

### `as const` / `satisfies` / `const T` 怎么用？

**`as const`**：让推断结果尽量"窄"，对象变全只读，字符串/数字变字面量类型。

```ts
const status = ['pending', 'success', 'fail'] as const
// 类型：readonly ['pending', 'success', 'fail']
type Status = typeof status[number]   // 'pending' | 'success' | 'fail'
```

**`satisfies` (TS 4.9+)**：让一个值满足某个类型约束，但**保留推导出的更具体类型**。

```ts
type RouteMap = Record<string, { path: string; auth?: boolean }>

// 用 : RouteMap 会让 user 变宽到 RouteMap 的成员
const routes = {
  home: { path: '/' },
  profile: { path: '/me', auth: true },
} satisfies RouteMap

routes.profile.auth   // ✅ boolean | undefined，类型精确
routes.unknown        // ❌ 编译报错
```

**`const T`（TS 5.0+ 泛型参数前的 const 修饰）**：让函数调用时把传入字面量直接推为字面量类型，不必让调用方写 `as const`。

```ts
function defineRoutes<const T>(routes: T): T { return routes }
const r = defineRoutes(['/home', '/about'])   // 自动是 readonly ['/home', '/about']
```

### TS 的类型守卫（Type Guard）有哪些？

收窄类型用，几种方式：

```ts
// 1. typeof —— 原始类型
if (typeof v === 'string') v.toUpperCase()

// 2. instanceof —— 类
if (e instanceof Error) e.message

// 3. in —— 属性是否存在
if ('id' in obj) obj.id

// 4. 自定义类型谓词（推荐）
function isUser(x: unknown): x is User {
  return !!x && typeof x === 'object' && 'id' in x && 'name' in x
}
if (isUser(data)) data.name      // 这里 data 收窄为 User

// 5. asserts（断言函数）
function assertNonNull<T>(x: T): asserts x is NonNullable<T> {
  if (x == null) throw new Error('null')
}
assertNonNull(user)              // 这行之后 user 收窄为非 null
user.name
```

`is` 用于"返回 true/false 表示是不是"，`asserts` 用于"不抛错就当作"。后者特别适合写 `assertDefined`、`assertNever` 这类校验函数。

### 模板字面量类型（Template Literal Types）

字符串层面的类型计算，常用来做 API 路径、CSS 属性等强约束。

```ts
type HttpMethod = 'GET' | 'POST'
type ApiPath = `/api/${string}`
type Endpoint = `${HttpMethod} ${ApiPath}`
// 'GET /api/...' | 'POST /api/...'

// 配合 keyof + 映射类型，做 getter 名生成
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
}
type UserGetters = Getters<{ id: number; name: string }>
// { getId: () => number; getName: () => string }
```

`Uppercase` / `Lowercase` / `Capitalize` / `Uncapitalize` 四个内置工具配合起来很灵活。

### `tsconfig` 的 `strict` 都包含什么？

`strict: true` 是一个聚合开关，等价于打开：

- `strictNullChecks`：`null` / `undefined` 不再能赋给其他类型，必须显式处理
- `strictFunctionTypes`：函数参数双向兼容改成逆变检查（更严格）
- `strictBindCallApply`：`bind/call/apply` 的参数类型严格校验
- `strictPropertyInitialization`：类字段必须在构造函数里初始化
- `noImplicitAny`：参数 / 变量未标类型时不再隐式当 `any`
- `noImplicitThis`：函数里的 `this` 必须能推断
- `alwaysStrict`：编译出的 JS 文件加 `'use strict'`
- `useUnknownInCatchVariables`：`catch (e)` 的 e 默认是 `unknown` 而不是 `any`

新项目建议直接开 `strict: true` + `noUncheckedIndexedAccess`（数组下标访问返回 `T | undefined`，避免下标越界）。

## 进阶题与类型体操

### TS 5.0 标准装饰器跟老的 experimentalDecorators 到底差在哪？

5.0 把装饰器跟进了 Stage 3 提案的实现，跟老的 `experimentalDecorators` 不是一回事。看一段就知道差异：

```ts
// 老版装饰器（experimentalDecorators + emitDecoratorMetadata）
function Log(target: any, key: string, desc: PropertyDescriptor) {
  const orig = desc.value
  desc.value = function (...args: any[]) {
    console.log(`call ${key}`)
    return orig.apply(this, args)
  }
}

// 标准装饰器（5.0+）
function log<This, Args extends any[], R>(
  target: (this: This, ...args: Args) => R,
  ctx: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => R>
) {
  return function (this: This, ...args: Args): R {
    console.log(`call ${String(ctx.name)}`)
    return target.call(this, ...args)
  }
}

class Service {
  @log
  fetch(id: number) { return id }
}
```

差异的关键不在写法长短，而在你怎么理解装饰器：

- 老版本是「描述符（PropertyDescriptor）的改写器」——拿到 desc，改它的 value/get/set，靠副作用生效
- 标准版本是「函数变换器」——接收原方法，返回新方法，类似高阶函数

标准版的优势：

- **能在类型层面追踪 this**——`ClassMethodDecoratorContext<This, F>` 把上下文信息编入类型，IDE 提示完整
- **没有「副作用 vs 返回值」的歧义**——标准装饰器明确「返回新函数 / 返回 undefined（保留原函数）」
- **不依赖 `emitDecoratorMetadata`**——老版本要靠 `reflect-metadata` 拿参数类型，标准装饰器走 `Symbol.metadata` 提案，更清晰
- **能装饰 class 字段的 initializer**（老版本拿不到 initializer）

但生态还卡在过渡期：

- **NestJS、TypeORM、TypeStack 全家桶仍用老装饰器**——它们重度依赖 `emitDecoratorMetadata` 自动拿参数类型做依赖注入，标准装饰器还没好的等价方案（Symbol.metadata 还是提案）
- **Angular** 用自己的编译器变换装饰器，跟两套提案都不完全一致

要点：「TS 5.0 跟进了 Stage 3 标准装饰器，类型更完善；老 experimentalDecorators 仍在主流框架中使用，短期不会消失；新项目如果不依赖 NestJS/TypeORM，建议用标准装饰器」。

### TypeScript 编译变慢怎么排查？

中后台 TS 项目跑到第二年，`tsc --noEmit` 从 5 秒变 50 秒是常态。排查思路就这几步。

**第一步：定位是哪个文件 / 哪个类型慢**：

```bash
# 让 tsc 输出类型检查耗时分布
tsc --noEmit --extendedDiagnostics

# 输出大致如下
# Files:                          12450
# Lines of Library:               40000
# Lines of TypeScript:            234000
# Identifiers:                    1200000
# Symbols:                        1500000
# Types:                          800000     ← 这个数大就要警惕
# Instantiations:                 5000000    ← 类型实例化次数,>1M 通常就慢
# Memory used:                    1500MB
# Check time:                     32.5s      ← 类型检查耗时
```

`Instantiations` 是关键指标——它代表「TS 把泛型类型展开了多少次」。如果存在大量深度递归的类型体操（`DeepPartial`、`Path<T>` 这种），这个数会快速升高。

**第二步：用 tsc trace 看具体哪个表达式慢**：

```bash
tsc --noEmit --generateTrace ./trace-out
# 然后用 Chrome 打开 chrome://tracing 加载 ./trace-out/trace.json
```

火焰图里能直接看出「这个文件里这一行类型断言耗了 800ms」。`@typescript/analyze-trace` 是 TS 团队官方的 trace 分析工具，能输出 hot path。

**第三步：常见的慢类型反模式**：

```ts
// ❌ 慢：深度条件类型 + 大联合类型
type DeepReadonlyAll<T> = {
  readonly [K in keyof T]: T[K] extends object
    ? T[K] extends Function ? T[K] : DeepReadonlyAll<T[K]>
    : T[K]
}
// 套用到一个 100 字段的复杂 schema 上,实例化次数会快速增长

// ❌ 慢:函数返回类型推导链太长
function pipe<A, B, C, D, E, F>(a: A, ...fns) { ... }  // 多层泛型推导

// ❌ 慢:大对象联合类型 + index access
type Result = SomeBigUnion[keyof SomeBigUnion]
```

**优化手段**：

- **拆 tsconfig**——`composite: true` + 项目引用，每个子项目独立增量编译，CI 只重编改动的那部分
- **关掉 `skipLibCheck: false`**——不要去检查 node_modules 里的类型，几乎是低风险加速项
- **`isolatedModules: true` + 用 esbuild/swc 转译**——tsc 只做类型检查，转译交给更快的工具
- **复杂类型加 `@ts-ignore` 或简化**——某些场景类型推导带来的安全收益不如它的编译开销大

**CI 优化**：用 `tsc --build --incremental` + 缓存 `tsbuildinfo`，二次 CI 只检查改动文件。

### TypeScript 类型体操实战：Path<T> 跟 Get<T, P>

复杂类型经常被认为维护成本高，但 Path / Get 这俩是真有实战价值。场景：你想类型安全地访问深层属性。

```ts
type User = {
  profile: {
    name: string
    address: {
      city: string
      coordinates: { lat: number; lng: number }
    }
  }
  posts: { title: string }[]
}

// 想要：get(user, 'profile.address.city') 返回 string，错的路径编译报错
function get<T, P extends Path<T>>(obj: T, path: P): Get<T, P> {
  return path.split('.').reduce((acc, key) => acc[key], obj as any)
}

get(user, 'profile.address.city')      // ✅ 推导为 string
get(user, 'profile.address.zip')       // ❌ 编译报错: 'zip' 不存在
get(user, 'profile.name.length')       // ✅ 推导为 number（string.length）
```

`Path<T>` 实现：

```ts
type Path<T, P extends string = ''> = T extends object
  ? {
      [K in keyof T & string]:
        | `${P}${P extends '' ? '' : '.'}${K}`
        | Path<T[K], `${P}${P extends '' ? '' : '.'}${K}`>
    }[keyof T & string]
  : never

// Path<User> 推导为
// "profile" | "profile.name" | "profile.address" | "profile.address.city"
// | "profile.address.coordinates" | "profile.address.coordinates.lat" | ...
```

`Get<T, P>` 实现：

```ts
type Get<T, P extends string> =
  P extends `${infer Key}.${infer Rest}`
    ? Key extends keyof T
      ? Get<T[Key], Rest>
      : never
    : P extends keyof T
      ? T[P]
      : never
```

这类工具常用在：

- **i18n 类型安全**：`t('user.profile.name')` 自动校验 key 存在、返回类型推导
- **表单字段 path**：react-hook-form 的 `register('user.address.city')` 已经内置这种类型
- **配置访问**：`config.get('database.replica.host')` 一行调用，全程类型安全

但要警惕：路径展开类型对**大对象**会快速膨胀——一个 100 字段的对象，Path 联合类型可能有上千个成员，IDE 提示也会变慢。所以这类工具要么限制递归深度（`Path<T, '', Depth = 3>`），要么只用在中小型对象上。

### `satisfies` 跟 `as` 的边界：为什么 5.0 推荐用 satisfies？

`satisfies` 是 4.9 加的，做的事是「检查这个值满足某类型，但保留它本来的精确类型」。看例子：

```ts
type RouteConfig = Record<string, { path: string; auth?: boolean }>

// 用 as
const routes1 = {
  home: { path: '/' },
  admin: { path: '/admin', auth: true },
} as RouteConfig

routes1.admin.auth   // 类型是 boolean | undefined（被宽化了）
routes1.unknown      // ✅ 编译不报错（as 强制断言）

// 用 satisfies
const routes2 = {
  home: { path: '/' },
  admin: { path: '/admin', auth: true },
} satisfies RouteConfig

routes2.admin.auth   // 类型是 true（保留了字面量类型）
routes2.unknown      // ❌ 编译报错（推断的类型里没这个 key）
```

差异在三点：

1. **`as` 是「绕过检查」**——你声明这是 RouteConfig，TS 就当它是。少了字段、多了字段，都不报错
2. **`satisfies` 是「检查 + 保留具体类型」**——保证符合 RouteConfig 约束，但展开后的字面量类型保留
3. **`as` 会把推导出的精确类型「宽化」到目标类型**；`satisfies` 不会

什么时候用 `as`？只有两种：

- **JSON.parse 之后**：`JSON.parse(str) as User`，运行时校验你自己做
- **跟外部 API 类型对接断言**：`fetch().then(r => r.json() as ApiResponse)`

什么时候用 `satisfies`？适合「保证某个值符合约束，但具体类型仍交给 TS 推导」的场景：

```ts
// 颜色 map
const colors = {
  primary: '#007bff',
  danger: '#dc3545',
} satisfies Record<string, `#${string}`>

// API 响应处理映射
const handlers = {
  200: (data) => data,
  404: () => null,
  500: () => { throw new Error() },
} satisfies Record<number, (data?: any) => unknown>
```

可以这样区分：**`as` 更接近「相信我」，`satisfies` 更接近「检查我」**。能用 satisfies 时，通常优先于 as。

### `unknown` / `any` / `never` 在工程里的实战边界

这三个常被混用，但实战里有明确分工。

**`unknown` 用在「不可信的边界」**：

```ts
// JSON.parse 出来的东西
const raw: unknown = JSON.parse(text)
if (isUser(raw)) {           // 用 type guard 收窄
  use(raw.name)
}

// 三方接口响应
async function fetchUser(): Promise<unknown> {
  const res = await fetch('/api/user')
  return res.json()
}

// catch 块（开了 useUnknownInCatchVariables）
try { ... } catch (e: unknown) {
  if (e instanceof Error) console.log(e.message)
}
```

`unknown` 强制你「用之前必须先检查类型」，是类型安全的 `any`。

**`any` 用在「真的不想纠结」的临时场景**：

- 写 demo / spike
- 三方库类型不完整、补类型成本太高
- 跟动态类型语言对接（GraphQL resolver 的 root 参数）

但**生产代码里出现的每一个 `any` 都应该有明确理由**。最好用 ESLint 的 `no-explicit-any` 规则强制，例外的地方加注释说明。

**`never` 用在「这里不可能到达」**：

```ts
// 穷尽检查
type Status = 'pending' | 'success' | 'fail'
function handle(s: Status) {
  switch (s) {
    case 'pending': return ...
    case 'success': return ...
    case 'fail': return ...
    default:
      const _: never = s   // 新增状态忘了处理 → 编译报错
      throw new Error()
  }
}

// 不可达的错误函数
function unreachable(): never {
  throw new Error('Unreachable')
}

// 过滤掉某些 union 成员
type WithoutNull<T> = T extends null ? never : T
```

**特别注意：`any` 和 `unknown` 在「赋值给别人」时方向相反**：

- 任何类型都能赋值给 `unknown`（吸收）；`unknown` 只能赋值给 `unknown` 或 `any`（不传染）
- 任何类型都能赋值给 `any`；`any` 也能赋值给任何类型（双向传染）

所以「类型未知」用 `unknown`，「放弃检查」用 `any`，「不可能到达」用 `never`。

### TS 跟运行时校验怎么协作？Zod / Valibot 该怎么用？

TS 的类型在编译完就擦除了，运行时拿不到。也就是说「接口返回的数据是不是符合 User 类型」TS 没法保证——服务端改了字段名你的代码可能直到访问 `user.name` 才挂。

Zod 就是为补这个洞设计的。它让你写「运行时 schema」，schema 能在运行时校验数据、也能反向推导出 TS 类型。

```ts
import { z } from 'zod'

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']),
})

// 类型自动从 schema 推导出来
type User = z.infer<typeof UserSchema>
// 等价于 type User = { id: number; name: string; email: string; age?: number; role: 'admin' | 'user' | 'guest' }

// 运行时校验
const result = UserSchema.safeParse(unknownData)
if (result.success) {
  result.data    // 类型是 User,可以安心用
} else {
  result.error   // ZodError,告诉你哪个字段不对
}
```

这套机制最大的价值是「类型和校验是一份源代码」。以前你要写一份 TS interface 给编译器看，再写一份校验函数给运行时看，两份永远在飘——schema 一改 interface 忘改、interface 加字段忘加校验。Zod 直接消灭这个问题。

**实战中怎么用**：

接口边界做校验，内部代码相信类型。`fetch` 拿到的数据先过 schema：

```ts
async function getUser(id: number): Promise<User> {
  const raw = await fetch(`/api/user/${id}`).then(r => r.json())
  return UserSchema.parse(raw)   // 不符合直接抛错,符合的话返回类型是 User
}
```

表单校验跟接口校验共用同一份 schema：

```ts
// shared/schemas/user.ts
export const CreateUserSchema = z.object({
  name: z.string().min(2, '名字至少 2 个字'),
  email: z.string().email('邮箱格式不对'),
  password: z.string().min(8, '密码至少 8 位'),
})

// 前端 react-hook-form
const { register, handleSubmit } = useForm({
  resolver: zodResolver(CreateUserSchema)
})

// 后端 Express
app.post('/users', (req, res) => {
  const data = CreateUserSchema.parse(req.body)
  // ...
})
```

前后端校验规则可以保持一致，规则改了双端一起生效。这是 Zod 最实用的地方。

**Valibot 是什么**：

Valibot 是 23 年起来的「Zod 替代品」，主打 tree-shaking 友好——Zod 是单一对象包含所有方法，bundle 进项目就是 50KB+；Valibot 是函数式 API，按需引入：

```ts
import { object, string, number, parse } from 'valibot'

const UserSchema = object({
  id: number(),
  name: string(),
})

parse(UserSchema, data)
```

只用到的部分会被打包，最小情况 1KB 都不到。前端体积敏感的项目（C 端 H5、小程序）优先 Valibot；服务端 / 中后台 Zod 仍是主流（生态丰富、tRPC / drizzle 都内置支持）。

### 索引签名 `[key: string]: T` 跟 `Record<string, T>` 有什么区别？

表面像同一个意思，实际有微妙差别——边界场景容易暴露问题。

```ts
// 索引签名
interface A {
  [key: string]: number
  name: number   // 必须跟索引签名兼容
}

// Record
type B = Record<string, number>
```

两者最大的差异在「能不能再扩展具名字段」：

```ts
// ❌ 索引签名+具名字段:具名字段必须满足索引类型
interface User {
  [key: string]: string
  age: number   // 编译报错:number 不能赋给 string
}

// ✅ 索引签名兼容
interface User {
  [key: string]: string | number
  name: string
  age: number
}

// Record 也类似,但用 intersection 扩展更自然
type User = Record<string, unknown> & {
  name: string
  age: number
}
```

更隐蔽的差异在「访问未声明 key 时的类型」：

```ts
interface MapA {
  [key: string]: string
}
const a: MapA = { foo: 'bar' }
a.baz   // string(索引签名兜底)

type MapB = Record<'foo' | 'bar', string>
const b: MapB = { foo: 'x', bar: 'y' }
b.baz   // ❌ 编译报错(没这个 key)
```

`Record<具体联合, T>` 是封闭的，编译器知道有哪些 key；`Record<string, T>` 和索引签名是开放的，任意 key 都返回 T（即便你没真的设置）。

开 `noUncheckedIndexedAccess` 后行为更安全：

```ts
const a: Record<string, string> = {}
const v = a.foo   // 类型是 string | undefined,被迫处理 undefined
```

**实际选型**：

- key 集合明确（特定枚举值映射）→ `Record<'a' | 'b', T>`
- key 是动态字符串但都映射同类型 → `Record<string, T>` 或索引签名
- 想在动态 key 之外再加几个固定字段 → 索引签名 + 具名字段，类型要兼容
- 大型 API 推荐开 `noUncheckedIndexedAccess`，编译时强制处理 undefined

### TS 的协变 / 逆变 / 双变是什么？什么时候会遇到问题？

这个话题容易讲抽象，回到代码层面本质是「函数参数类型兼容性怎么判断」。

先看一个常见场景：

```ts
class Animal { name: string }
class Dog extends Animal { breed: string }

let a: (x: Animal) => void
let d: (x: Dog) => void

a = d   // ✅ 还是 ❌?
d = a   // ✅ 还是 ❌?
```

直觉上「Dog 是 Animal 的子类型」，所以处理 Dog 的函数应该兼容处理 Animal 的函数？但**反过来**：

- `d = a`：✅ 可以。a 能处理任意 Animal，肯定能处理 Dog（Dog 也是 Animal）
- `a = d`：❌ 不行（开 `strictFunctionTypes`）。d 只能处理 Dog，传 Animal 进去（比如 Cat）就会出错

这就是**逆变（contravariance）**：函数参数的兼容方向跟类型的子类型关系是**反着**的。

**协变（covariance）**：返回值方向相反。返回 Dog 的函数可以被当作返回 Animal 的函数：

```ts
let getA: () => Animal
let getD: () => Dog
getA = getD   // ✅(返回的是 Dog,但调用方期待 Animal,Dog 也是 Animal)
getD = getA   // ❌(调用方期待 Dog,但拿到 Animal)
```

**双变（bivariance）**：参数同时支持协变和逆变。TS 在 `strictFunctionTypes: false` 时就是双变——上面的 `a = d` 也会被允许。

项目里遇到问题的高频场景：

**事件回调签名**：

```ts
interface Listener<E> {
  (event: E): void
}

// 想要一个能处理所有 Event 的通用监听器,接收子类型的回调
function on<E>(name: string, cb: Listener<E>) { ... }

on<MouseEvent>('click', (e: Event) => {})   // strict 下报错
// 因为参数类型不兼容:Event 不是 MouseEvent 的子类型
```

**数组的特殊性**：

数组在 TS 里是协变的（不严格符合类型论但实用）：

```ts
let dogs: Dog[] = [new Dog()]
let animals: Animal[] = dogs   // ✅(协变)
animals.push(new Cat())        // 编译过了,但 dogs 里现在塞了 Cat!
dogs[1].breed                  // 运行时崩(Cat 没 breed)
```

这是 TS 为了实用性主动留的洞。Rust / Scala 在这种地方会强制不变（invariant），TS 放过了。

**类方法的双变**：

类的方法签名默认是双变的（即便开了 strictFunctionTypes），这是「为了兼容大量 OO 代码」做的妥协。但接口里的方法 / 函数类型字面量是严格逆变的。所以同样的逻辑写成 class method 不报错、写成 interface property 报错，根源在这里。

```ts
class A {
  handler(x: Animal): void {}
}
class B extends A {
  handler(x: Dog): void {}   // ✅ class method 双变,允许重写得更具体
}

interface IA {
  handler: (x: Animal) => void
}
interface IB extends IA {
  handler: (x: Dog) => void   // ❌ interface property 逆变,不允许
}
```

遇到「这两个函数类型为什么不兼容」的编译错误，可以优先检查是否由参数逆变导致。改法通常是放宽参数类型（用更通用的父类型），或者拆成两个独立 hook。

### `export type` 跟 `export` 的边界在哪？

TS 5.0 起 `verbatimModuleSyntax` 选项被广泛推荐，配合 `isolatedModules`，强制要求「只用作类型的导出导入要写 `type`」。表面是个语法小细节，实际影响构建产物和编译速度。

```ts
// types.ts
export interface User { id: number; name: string }
export const API_URL = '/api/v1'

// usage.ts
import type { User } from './types'         // 仅类型导入,编译时擦除
import { API_URL } from './types'           // 运行时导入,会进 bundle
```

为什么要区分？

**bundler / 编译器需要的判断**：

- `tsc` 把 TS 编译成 JS 是逐文件做的（`isolatedModules: true` 强制要求）
- 编译器看到 `import { User } from './types'` 时，**只看当前文件没法知道 User 是类型还是值**
- 如果保留这行 import，运行时会真的尝试 require './types' 找 User，结果是 undefined（因为 interface 编译后不存在）
- 加上 `type` 关键字，编译器明确知道这是类型导入，编译时整行擦除

**`verbatimModuleSyntax` 的严格规则**：

```ts
// ❌ 报错
import { User, API_URL } from './types'
// User 是类型,API_URL 是值,混在一起,不让

// ✅ 拆开
import type { User } from './types'
import { API_URL } from './types'

// ✅ 或者用 inline type
import { type User, API_URL } from './types'
```

inline type 写法更紧凑，但 ESLint 默认会强制拆开（看 `import/consistent-type-specifier-style` 配置）。

**实战影响**：

- **bundle 大小**：没标 type 的接口导入会让 bundler 误以为「这个模块被使用了」，整个模块会被拉进 bundle。标了 type 后构建工具直接 drop。中大型项目体积差异能到 5-10%
- **编译速度**：esbuild、swc 这类速度优先的编译器**只能逐文件转译，没有跨文件类型信息**——`verbatimModuleSyntax` 让它们能确定地擦除类型导入
- **依赖循环避免**：循环依赖里有时一边只用类型，另一边只用值。用 type import 切断「值的依赖」就能解开循环

**re-export 的特殊场景**：

```ts
// barrel file (index.ts)
export type { User } from './user'
export type { Order } from './order'
export { api } from './api'
```

barrel 文件如果不标 type，整个 user.ts 和 order.ts 都会被拉进消费者的 bundle，即使只用了类型。

**常见错误信号**：

- 看到「module not found」但代码看起来没问题 → 可能是 type 没标，运行时去找不存在的导出
- bundle 体积莫名其妙大 → 检查是不是 barrel files 没标 type
- esbuild 报「Top-level await / 类型语法不允许」→ 通常 verbatimModuleSyntax 没开

新项目模板里建议直接加：

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "verbatimModuleSyntax": true,
    "isolatedModules": true
  }
}
```

工程养成习惯后这条规则就是「自动遵守」的，不用纠结。

### never 不只是「不可能到达」——还能拿来过滤类型

前面讲过 never 在穷尽检查里的用法。但 never 在类型层面还有个隐藏能力：**在联合类型里，never 自动消失**。这是写复杂工具类型的常用招数。

```ts
type T1 = string | never        // 等价于 string
type T2 = number | never        // 等价于 number
type T3 = string | number | never  // 等价于 string | number
```

利用这点可以做「条件过滤」：

```ts
// 从联合里挑出某种类型
type ExtractStrings<T> = T extends string ? T : never

type Mixed = string | number | boolean | 'foo' | 42
type Strings = ExtractStrings<Mixed>   // string | 'foo'
// number、boolean、42 都被映射为 never,自动消失
```

实战例子：从对象类型里挑出「值是函数」的 key。

```ts
type FunctionKeys<T> = {
  [K in keyof T]: T[K] extends Function ? K : never
}[keyof T]

type API = {
  url: string
  timeout: number
  onSuccess: () => void
  onError: (e: Error) => void
}

type Callbacks = FunctionKeys<API>   // 'onSuccess' | 'onError'
```

拆解这个类型：

1. `{ [K in keyof T]: ... }` 遍历每个 key
2. `T[K] extends Function ? K : never` 是函数就保留 key 名，不是就置 never
3. `[keyof T]` 取出值的联合——never 自动消失，剩下的就是函数 key 们

类似套路能写「所有可选属性」「所有必填属性」「所有非 readonly 属性」：

```ts
// 所有可选属性
type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never
}[keyof T]

// 排除 null 和 undefined
type NonNullableKeys<T> = {
  [K in keyof T]-?: null | undefined extends T[K] ? never : K
}[keyof T]
```

记一条：**`X extends Y ? A : B` 里返回 never 可以在联合层面「擦除」一个分支**。这是类型体操的常见技巧。

### TS 项目结构里 barrel files 该用吗？

`index.ts re-export all` 这种文件叫 barrel files，组件库 / utils 库里很常见。表面上清晰（消费侧 `import { A, B } from '@my/lib'` 一行解决），但工程上隐患不少。

```ts
// components/index.ts —— 典型 barrel
export * from './Button'
export * from './Input'
export * from './Modal'
// ... 几十个 re-export
```

消费侧：

```ts
import { Button } from '@my/lib/components'
// 等价于:打开 components/index.ts → 把所有 export 都列出来 → 拿出 Button
```

**问题 1：Tree Shaking 失效**

理论上 Webpack / Vite 能 tree shake 掉没用的 export。但实际上：

- barrel file 中的副作用模块（含 import 顶层语句）会被全部拉进来
- 某些 re-export 链路深时（barrel 套 barrel）bundler 分析容易失败
- 结果就是 bundle 里塞进了用户根本没引用的代码

**问题 2：编译速度**

- TS 编译器看到 barrel import 时要把整个 barrel 解析一遍才能找到对应类型
- 大型项目里 barrel 嵌套深，一次类型检查能慢 30-50%

**问题 3：循环依赖灾难**

```
components/index.ts re-export 所有
Button.tsx → import Modal from '../components'   // ❌ 通过 barrel 引入兄弟
Modal.tsx → import Button from '../components'
```

通过 barrel 互相引用容易形成循环依赖，调试时很困难。

**怎么治理**：

- **内部代码不走 barrel**：组件库内部组件之间引用对方时用相对路径（`from './Button'`），不走 `from '.'`
- **对外 barrel 保留**：消费者用 barrel 是合理的，但 barrel 只 re-export type 和 const，不要套 barrel
- **细粒度 export 字段**：包的 `package.json` 用 `exports` 暴露多个入口，让消费者按需 import

```jsonc
{
  "exports": {
    ".": "./dist/index.js",
    "./Button": "./dist/Button/index.js",
    "./Modal": "./dist/Modal/index.js"
  }
}
```

```ts
// 消费侧
import Button from '@my/lib/Button'   // 直接命中,不走 barrel
```

Material UI、Ant Design 早期都遇到过 barrel 带来的体积问题，后来都改成了「子路径导入」+ `babel-plugin-import` 这种按需引入插件辅助。新项目从一开始就走 `exports` 字段是更清晰的方案。

### Branded Types / Nominal Typing：怎么强制区分「都是 string 但含义不同」？

TS 是结构化类型——只要形状一样就兼容。也就是说 `type UserId = string` 跟 `type OrderId = string` 在 TS 眼里就是一回事，传参时混着传也不报错：

```ts
type UserId = string
type OrderId = string

function getUser(id: UserId) { /* ... */ }

const orderId: OrderId = 'order-123'
getUser(orderId)   // ✅ TS 不报错,但运行时业务出错
```

「真正区分两种 string」是「名义类型」（nominal typing）的事，TS 默认不支持，但能用 branded types 模拟：

```ts
// Brand 类型工具
type Brand<T, B> = T & { __brand: B }

type UserId = Brand<string, 'UserId'>
type OrderId = Brand<string, 'OrderId'>

function getUser(id: UserId) { /* ... */ }

const userId = 'u-1' as UserId
const orderId = 'o-1' as OrderId

getUser(userId)    // ✅
getUser(orderId)   // ❌ 编译报错:OrderId 不能赋给 UserId
```

`__brand` 字段在运行时根本不存在（只是个类型层面的「标签」），但 TS 编译器会按它来区分两个 brand。

**Brand 的实战场景**：

```ts
// 用户身份相关
type Email = Brand<string, 'Email'>
type UserId = Brand<number, 'UserId'>

// 单位
type Meters = Brand<number, 'Meters'>
type Seconds = Brand<number, 'Seconds'>

function speed(distance: Meters, time: Seconds): number {
  return distance / time   // 类型保证你不会把 seconds 当 meters 传
}

// 经过校验的「安全字符串」
type SafeHtml = Brand<string, 'SafeHtml'>
type UnsafeUserInput = Brand<string, 'UnsafeUserInput'>

function renderHtml(html: SafeHtml) { /* 这里相信内容安全 */ }
function sanitize(input: UnsafeUserInput): SafeHtml {
  return DOMPurify.sanitize(input) as SafeHtml
}

const userInput = 'something' as UnsafeUserInput
renderHtml(userInput)              // ❌ 拒绝直接渲染
renderHtml(sanitize(userInput))    // ✅ 先 sanitize 再渲染
```

最后这个例子是 brand 的典型用法——**把「校验过」这个事实编入类型系统**。一个值要拿到 `SafeHtml` 类型，唯一途径是经过 `sanitize` 函数，编译器强制保证业务代码不能绕过 sanitize 直接渲染脏数据。

**怎么创建 branded 值**：

较简单的是 `as`：

```ts
const userId = 'u-1' as UserId
```

但 `as` 没运行时校验。更好的做法是写一个「smart constructor」：

```ts
function createUserId(s: string): UserId {
  if (!s.startsWith('u-')) throw new Error('Invalid UserId')
  return s as UserId
}
```

这样既有运行时校验、又有类型保证。

**生态库**：

- `ts-brand`：标准实现
- `effect-ts` 的 `Schema`：把 brand + 运行时校验深度集成
- Zod 的 `.brand()`：schema 校验后直接出 branded 类型

```ts
import { z } from 'zod'

const UserIdSchema = z.string().brand<'UserId'>()
type UserId = z.infer<typeof UserIdSchema>

const id = UserIdSchema.parse('u-1')   // 校验 + 自动加 brand
```

频繁出现 ID 混用的项目可以考虑 brand。一个 type alias 几行字，能避免一类整个项目级别的基础错误。

### TS 5.5 / 5.6 / 5.7 这几个版本带来了什么实际能用的东西？

TS 每半年一个版本，小版本之间的新能力不少。这里挑几个业务开发中更常用的能力。

**TS 5.5：推导 type predicates**

以前 `Array.filter` 里要把类型缩窄，需要写 type guard 函数：

```ts
// 老版本
function isString(x: unknown): x is string {
  return typeof x === 'string'
}

const mixed: (string | number)[] = ['a', 1, 'b']
const strings = mixed.filter(isString)   // string[]
```

5.5 起编译器能从函数实现自动推导出 `is` 谓词：

```ts
const strings = mixed.filter((x) => typeof x === 'string')
// 5.5 自动推导出 strings: string[]
// 之前要显式写 (x): x is string =>
```

写 utilities 时省一行类型注解。

**TS 5.6：disallow null/truthy 检查**

TS 终于会警告「永远为真」「永远为假」的检查：

```ts
function bad(arr: number[]) {
  if (arr) {   // ❌ TS 5.6 警告:数组永远 truthy
    // ...
  }
}

function check(promise: Promise<void>) {
  if (promise === null) {   // ❌ 警告:Promise 不会是 null
    // ...
  }
}
```

帮你抓 「以为做了检查、实际没做」的 bug。

**TS 5.6：iterator helpers 类型**

Stage 3 的 `Iterator.prototype.map/filter/take` 等终于有了类型支持：

```ts
const evens = Iterator.range(0, Infinity)
  .filter(x => x % 2 === 0)
  .take(10)
  .toArray()
```

iterator helpers 还在浏览器铺开中，但 polyfill 用起来类型完整。

**TS 5.7：默认 ES2022 target**

`target` 默认从 `ES3` / `ES5` 升到 `ES2022`。意味着不用再 polyfill `?.`、`??=`、`#privateField` 这些。配合现代浏览器市场份额（90%+ 支持 ES2022），新项目可以直接定 target ES2022 甚至 ESNext。

**TS 5.7：path rewriting for .ts/.js**

老问题：TS 里写 `import { x } from './foo.ts'` 在编译时不会被改成 `./foo.js`，结果运行时找不到文件。5.7 起加 `rewriteRelativeImportExtensions` 选项：

```jsonc
{
  "compilerOptions": {
    "rewriteRelativeImportExtensions": true
  }
}
```

编译时自动把 `.ts` 改成 `.js`，跟 Node ESM 的「必须带后缀」要求对齐。

**5.5+ 通用建议**：

- 老项目挨个升级，别一次跳几个大版本——每个版本都可能引入新的严格规则，让老代码暴露出类型错误
- 升级时配合 `tsc --noEmit` 跑一遍全量检查，看哪些新警告
- `verbatimModuleSyntax` 建议开（前面提过）
- `noUncheckedIndexedAccess` 是大型项目的必备「严格性」开关

社区里 `lint-staged` 配合 `tsc --noEmit --pretty` 在 PR 时跑增量类型检查越来越普及，是替代 ESLint 类型规则的更稳方式。

### TS Project References 在大型项目里实战

`composite: true` + `references` 是 TS 给 monorepo / 大型项目的「增量编译」方案。前面提过原理，这里说怎么真正用起来。

**典型 monorepo 配置**：

```
my-monorepo/
├── tsconfig.json              # 根配置:references
├── tsconfig.base.json         # 共享的编译选项
├── packages/
│   ├── shared/
│   │   ├── tsconfig.json
│   │   ├── src/
│   │   └── dist/
│   ├── web/
│   │   ├── tsconfig.json
│   │   └── src/
│   └── api/
│       ├── tsconfig.json
│       └── src/
```

`tsconfig.base.json`（共享配置）：

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "skipLibCheck": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}
```

`packages/shared/tsconfig.json`：

```jsonc
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "composite": true,         // 关键
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

`packages/web/tsconfig.json`（依赖 shared）：

```jsonc
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "composite": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "references": [
    { "path": "../shared" }    // 声明依赖
  ],
  "include": ["src/**/*"]
}
```

`tsconfig.json`（根配置）：

```jsonc
{
  "files": [],
  "references": [
    { "path": "./packages/shared" },
    { "path": "./packages/web" },
    { "path": "./packages/api" }
  ]
}
```

**使用方式**：

```bash
# 增量构建整个项目
tsc --build

# 只 type-check 不出 emit
tsc --build --noEmit

# 看看哪些包要重 build
tsc --build --dry

# 清理 build 缓存
tsc --build --clean
```

`tsc --build` 执行时会：

1. 读 references 算出依赖图
2. 看每个包的 `.tsbuildinfo` 缓存
3. 只重 build 改动过的包 + 依赖它的包

实测下来：大型项目（10 个包、几千文件）首次 build 30 秒，二次只改一个包的 build 3 秒。差距巨大。

**注意点**：

- **dev 时跨包修改不会立刻反映**：因为 web 看到的是 `shared/dist/` 而不是 `shared/src/`。要么用 `tsc --build --watch`，要么用 path mapping 跳过 build
- **path mapping 跟 references 矛盾**：path mapping 让 TS 直接读 src，但 references 要求读 dist。两者只能选一
- **bundler 不一定理解 references**：esbuild、swc 这些直接读 src，不管 references。所以 references 主要服务于「类型检查 + 出库 build」，bundler 用 path mapping

常见组合：

- **dev / bundler**：path mapping 读源码，速度最快
- **CI 类型检查**：`tsc --build` 走 references，增量超快
- **出库 build**：`tsc --build` 生成 `.d.ts` + `dist`

一个项目两条路并存。tsconfig 可以用 `tsconfig.dev.json` / `tsconfig.build.json` 分开。

社区里 Turborepo / Nx 都对 TS Project References 有第一类支持，自动算 references 关系、自动跑 affected build。手写配置只在小项目里值得。

## 附:常见代码实现

### 1. 泛型函数

```ts
function identity<T>(value: T): T {
  return value
}
```

### 2. 接口与类

```ts
interface User {
  id: number
  name: string
}

class UserService {
  getUser(): User {
    return { id: 1, name: 'Tom' }
  }
}
```

### 3. 常见工具类型

```ts
type User = { id: number; name: string; age: number }
type UserPreview = Pick<User, 'id' | 'name'>
type UserOptional = Partial<User>
type UserMap     = Record<string, User>
type WithoutAge  = Omit<User, 'age'>
```

### 4. infer 自定义工具类型

```ts
// 取 Promise 内层
type Unwrap<T> = T extends Promise<infer U> ? U : T

// 取数组元素
type ItemOf<T> = T extends (infer I)[] ? I : never

// 取函数第一个参数
type FirstArg<F> = F extends (a: infer A, ...rest: any[]) => any ? A : never
```
