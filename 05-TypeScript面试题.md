[toc]



## 一、TypeScript 基础认知

### 说说你对 TypeScript 的理解?与 JavaScript 的区别是什么?

**根本原因**

JavaScript 是动态弱类型语言,类型错误要等到**运行时**才暴露。TypeScript 在 JavaScript 之上加一层**编译期静态类型系统**,把"运行时才崩"的问题提前到"写代码时就报错",本质是用类型信息换可维护性。

**具体机制**

1. **超集**:任何合法的 JS 都是合法的 TS,渐进迁移没成本。
2. **类型擦除**:TS 编译时移除所有类型注解,产物是纯 JS,运行时**没有任何类型信息**(这也是为什么不能 `typeof` 判断 TS 接口)。
3. **类型推导**:不写注解时,TS 也会根据上下文反推类型,大量场景下无需手写。
4. **结构化类型(Structural Typing)**:类型兼容看"形状",不看名字——只要字段对得上就兼容。

**对比表**

| 维度 | JavaScript | TypeScript |
|------|-----------|-----------|
| 类型检查 | 运行时 | 编译期 |
| 错误发现 | 上线后用户暴露 | 写代码时 IDE 标红 |
| 大型项目协作 | 重构靠搜索 | 重构靠类型驱动 |
| 学习成本 | 低 | 中 |
| 运行时性能 | 直接运行 | 需编译,产物等同 JS |
| 生态 | 无类型 | DefinitelyTyped 提供 `@types/*` |

**实操建议**

1. 新项目直接上 TS,迁移老项目用 `allowJs` + `checkJs` 渐进改造。
2. `tsconfig.json` 开 `strict: true`,享受类型系统的全部红利;关掉等于白用。
3. 不要用 `// @ts-ignore` 当万能膏药,它只解决报错,不解决问题——优先用类型收窄或断言。

### 说说 TypeScript 的数据类型有哪些?

**根本原因**

TS 类型系统在 JS 的 7 种基础类型之上,加了一组"描述类型本身行为"的特殊类型(`any` / `unknown` / `never` / `void`),用来表达**未知**、**不可能**、**无返回**等语义。

**具体机制**

1. **基础类型**:对应 JS 的原始值:`string`、`number`、`boolean`、`null`、`undefined`、`symbol`、`bigint`。
2. **特殊类型**:
   - `any`:**关掉类型检查**,赋值给任何类型都行(危险)。
   - `unknown`:类型安全的 `any`,**用之前必须收窄**。
   - `void`:函数无返回值。
   - `never`:**永远不会有值**(抛错、死循环、被穷尽的分支)。
3. **复合类型**:数组 `T[]`、元组 `[T, U]`、对象、函数。
4. **引用相关**:`interface`、`type`、`class`、`enum`。

**对比表(`any` / `unknown` / `never`)**

| 类型 | 能否赋值给它 | 它能否赋值给别人 | 典型用途 |
|------|------------|----------------|---------|
| `any` | ✅ 任何类型 | ✅ 任何类型 | 临时绕过类型(慎用) |
| `unknown` | ✅ 任何类型 | ❌ 必须先收窄 | 三方数据入口、`JSON.parse` |
| `never` | ❌ 没有值能赋给它 | ✅ 任何类型 | 抛错函数、穷尽检查 |

**代码:用 never 做穷尽检查**

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

**实操建议**

1. 三方接口、`JSON.parse`、`localStorage` 取值这些"边界数据"用 `unknown`,内部代码用具体类型。
2. 用 `never` 做联合类型穷尽检查,新增枚举值时编译器自动告诉你哪里漏处理了。
3. 不要把 `void` 当 `undefined` 用——`void` 表示"调用者不应该关心返回值"。

## 二、类型系统与核心能力

### 说说你对 TypeScript 中枚举类型的理解?应用场景是什么?

**根本原因**

枚举是给一组**有限的常量**起一个有语义的名字,避免代码里到处出现 `0`、`1`、`'PENDING'` 这种"魔法值"。但 TS 的 `enum` 编译后会生成**额外运行时代码**(双向映射对象),不像类型那样被擦除,这是它和"字符串字面量联合类型"最大的区别。

**具体机制**

1. **数字枚举**:不指定值时从 0 开始递增,会生成正反向映射(`Color[0] === 'Red'`)。
2. **字符串枚举**:必须显式赋值,只有正向映射,可读性更好。
3. **常量枚举(`const enum`)**:编译时**完全内联**,不生成运行时对象,体积最小。
4. **异构枚举**:数字字符串混用,**不推荐**。

**对比:enum vs 联合字面量类型**

| 维度 | `enum` | 联合字面量 |
|------|--------|-----------|
| 运行时产物 | 有(对象) | 无(类型擦除) |
| 反向查值 | 数字枚举支持 | 不支持 |
| 摇树优化 | 较差 | 好 |
| 可读性 | `Color.Red` | `'red'` |
| 跨文件复用 | 直接 import | 写 `as const` |

**代码**

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

// 联合字面量(很多团队的首选)
type Status2 = 'pending' | 'success' | 'fail'
```

**实操建议**

1. 新项目优先用**字符串字面量联合类型**,体积小、Tree-shaking 友好、调试看到的就是字面量本身。
2. 真要用 `enum` 选**字符串枚举**,日志里出现 `'admin'` 比 `0` 容易排查。
3. `const enum` 跨包发布会被 Babel 等工具误解析,共享库慎用——可改用 `as const` 对象。

### 说说你对 TypeScript 中接口的理解?应用场景是什么?

**根本原因**

接口(`interface`)是描述"对象形状"的契约。TS 用**结构化类型**判断兼容性——一个对象只要"长得像"接口,就算兼容,不需要显式 `implements`。这种宽松的契约让接口成为前端**模块边界的天然描述工具**。

**具体机制**

1. **描述对象结构**:字段 + 类型。
2. **可选与只读**:`prop?: T`、`readonly prop: T`。
3. **索引签名**:`[key: string]: T`,描述动态键的对象。
4. **函数类型**:`interface Fn { (x: number): string }`。
5. **可合并(Declaration Merging)**:同名 interface 自动合并字段——`type` 不能。
6. **继承**:`interface B extends A`。

**对比:interface vs type**

| 维度 | `interface` | `type` |
|------|------------|--------|
| 描述对象 | ✅ | ✅ |
| 联合 / 交叉类型 | ❌ | ✅ |
| 元组 / 字面量 | ❌ | ✅ |
| 同名合并 | ✅(自动) | ❌(报错) |
| 继承语法 | `extends` | `&` 交叉 |
| 性能(大型项目) | 略快 | 略慢 |

**代码**

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

**实操建议**

1. 描述对象/类用 `interface`,描述联合、元组、工具类型用 `type`——一条简单分工。
2. 三方库扩展(给 `Window` 加属性、给 `Vue` 加 `$xxx`)只能用 `interface` 的合并能力。
3. 接口里的方法名拼错和实际实现对不上,排查极痛苦——开 `noImplicitOverride` + 单元测试兜底。

### 说说你对 TypeScript 中类的理解?应用场景是什么?

**根本原因**

TS 的 class 在 ES class 之上加了**访问修饰符、抽象类、参数属性、实现接口**等能力,让面向对象建模在 TS 中真正可落地。但前端业务里"组件 + hook"的函数式风格更主流,class 的核心战场是**SDK、领域模型、Node 后端**。

**具体机制**

1. **访问修饰符**:`public`(默认)、`private`、`protected`——只在编译期检查,运行时不生效;ES 私有字段 `#prop` 才是运行时硬隔离。
2. **`readonly`**:实例属性只读。
3. **抽象类 `abstract`**:不能直接 `new`,只能被继承。
4. **参数属性**:构造函数参数加修饰符,自动赋值给实例,省样板代码。
5. **`implements` vs `extends`**:`implements` 只校验结构,不继承实现;`extends` 继承父类。

**代码**

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

**实操建议**

1. 优先 ES 私有字段 `#prop`,真正运行时隔离;`private` 只是编译期约束,被强转就破。
2. 业务代码"能用函数+hook 就别用 class",class 适合**有状态机、需要继承、生命周期复杂**的场景(比如 SDK)。
3. 不要在 class 里塞太多静态方法——那其实就是命名空间下的工具函数,直接用 `export function` 更轻。

### 说说你对 TypeScript 中函数的理解?与 JavaScript 函数的区别是什么?

**根本原因**

TS 函数运行时还是 JS 函数,差别只在**签名层**:把"参数怎么传、返回什么、有哪几种重载形态"用类型固定下来,调用方一看类型就知道怎么用,不用翻实现。

**具体机制**

1. **参数/返回值标注**:`function f(x: number): string`。
2. **可选 / 默认 / 剩余参数**:`x?: T`、`x: T = 1`、`...args: T[]`。
3. **函数重载**:多个签名 + 一个实现,提供更精确的类型推导。
4. **函数类型字面量**:`type Fn = (x: number) => string`。
5. **`this` 类型**:第一个参数声明 `this: SomeType`,运行时不传,只供类型校验。

**代码:重载的典型场景**

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

**实操建议**

1. 公共 API 优先重载,不要靠"返回联合类型 + 调用方断言",体验差。
2. 回调函数签名尽量声明返回类型,避免实现里偷偷 `return` 不该返回的值导致联合类型扩散。
3. 函数泛型默认值善用,降低调用方心智:`function fetch<T = unknown>(): Promise<T>`。

### 说说你对 TypeScript 中泛型的理解?应用场景是什么?

**根本原因**

泛型解决的是"**复用**"和"**类型安全**"的矛盾。没有泛型时,通用工具只能写 `any` 牺牲类型;有了泛型,类型变成"参数",调用时再具体化,做到"一次定义,随类型变化"。

**具体机制**

1. **泛型函数**:`function id<T>(x: T): T`,T 在调用时确定。
2. **泛型接口/类**:`interface Box<T> { value: T }`。
3. **泛型约束(`extends`)**:限制 T 必须满足某种结构。
4. **默认泛型**:`T = string`,不传时取默认值。
5. **多泛型协作**:`<K extends keyof T>`,实现属性级类型安全。

**代码**

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

**实操建议**

1. 不要为了用泛型而用泛型——只在"类型随调用变化"时引入,否则只是噪音。
2. 给泛型加约束(`<T extends Record<string, unknown>>`),否则 T 是 `unknown`,调用方写出来还是没用。
3. 泛型命名:单字母用于"通用占位"(`T`、`K`、`V`、`R`),多字母用于"语义明确"(`TProps`、`TData`)。

### 说说你对 TypeScript 中高级类型的理解?有哪些?

**根本原因**

高级类型让 TS 从"类型标注工具"升级为"**类型计算工具**"——可以基于已有类型推导、过滤、变换出新类型,实现"类型驱动开发"。其核心能力是 `keyof` + `extends` + `infer` 三件套。

**具体机制**

1. **联合类型 `A | B`**:或关系。
2. **交叉类型 `A & B`**:与关系(对象合并)。
3. **字面量类型 `'a' | 'b'`**:精确到值。
4. **条件类型 `T extends U ? X : Y`**:类型层面的 if-else。
5. **映射类型 `{ [K in keyof T]: ... }`**:遍历键改类型。
6. **`keyof T`**:取所有键的联合。
7. **`typeof v`**:取变量的类型。
8. **`infer R`**:在条件类型中"声明并推导"一个新变量。
9. **内置工具类型**:`Partial<T>`、`Required<T>`、`Pick<T,K>`、`Omit<T,K>`、`Record<K,V>`、`ReturnType<F>`、`Parameters<F>`、`Awaited<P>`。

**常用工具类型对照**

| 工具类型 | 作用 | 等价实现 |
|---------|------|---------|
| `Partial<T>` | 所有字段可选 | `{ [K in keyof T]?: T[K] }` |
| `Required<T>` | 所有字段必填 | `{ [K in keyof T]-?: T[K] }` |
| `Readonly<T>` | 所有字段只读 | `{ readonly [K in keyof T]: T[K] }` |
| `Pick<T, K>` | 挑选指定键 | `{ [P in K]: T[P] }` |
| `Omit<T, K>` | 排除指定键 | `Pick<T, Exclude<keyof T, K>>` |
| `Record<K, V>` | 构造键值对类型 | `{ [P in K]: V }` |
| `ReturnType<F>` | 取函数返回类型 | `F extends (...a: any) => infer R ? R : never` |

**代码**

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

**实操建议**

1. 优先用内置工具类型,可读性比自己 `infer` 一通强;不够再自定义。
2. 类型体操写到第三层嵌套就停下问问自己:**这个类型有没有更简单的表达**——别人维护不了的"炫技类型"是负资产。
3. 复杂类型加注释 + 单测(用 `expectTypeOf` 或 `tsd`),否则改一处崩一片没人敢动。

## 三、进阶能力与工程实践

### 说说你对 TypeScript 装饰器的理解?应用场景是什么?

**根本原因**

装饰器是给类、方法、属性、参数加**额外行为或元数据**的语法糖。它的本质是**高阶函数**——接收原类/方法,返回增强后的版本。装饰器解决的是"在不修改业务代码的前提下加横切逻辑"(日志、权限、依赖注入)。

**具体机制**

1. **类装饰器**:接收构造函数,返回新构造函数。
2. **方法装饰器**:接收 `(target, key, descriptor)`,可改写方法。
3. **属性装饰器**:常配合 `reflect-metadata` 收集类型元信息。
4. **参数装饰器**:常用于参数注入(NestJS 的 `@Body()`)。
5. **执行顺序**:同一目标多个装饰器,**自下而上**执行。

**代码**

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

**应用场景**

| 场景 | 典型框架/库 |
|------|-----------|
| 依赖注入 | NestJS、Angular |
| ORM 映射 | TypeORM、MikroORM |
| 路由声明 | NestJS、midway |
| 接口校验 | class-validator |
| 埋点 / 日志 | 自研 AOP |

**实操建议**

1. 区分两套装饰器:**老 `experimentalDecorators` + `emitDecoratorMetadata`** 和 **TS 5.0 标准装饰器**,语法和行为不一样,确认目标框架支持哪套。
2. 装饰器常需要 `reflect-metadata`,别忘在入口顶部 `import 'reflect-metadata'`。
3. 装饰器写多了会让控制流难以追踪,写之前先问"普通函数能不能做"——能就别用装饰器。

### 说说对 TypeScript 中命名空间与模块的理解?区别是什么?

**根本原因**

命名空间(namespace)是 TS 早期、ES Module 规范还没普及时的"全局变量隔离方案",通过包一层闭包对象避免命名冲突。模块(module)是 ES Module 规范下的标准代码组织方式。**两者只在历史上并存,新项目应该只用模块**。

**具体机制**

1. **命名空间**:`namespace Foo {}`,编译为一个全局对象 `Foo`。
2. **模块**:`export` / `import`,文件即模块,作用域天然隔离。
3. 命名空间可以跨文件用 `/// <reference path>` 拼接(脆弱)。
4. 模块走打包工具的依赖图,Tree-shaking 友好。

**对比表**

| 维度 | 命名空间 | 模块 |
|------|---------|------|
| 隔离方式 | 全局对象 | 文件作用域 |
| 引入方式 | 全局可见 / `<reference>` | `import` |
| Tree-shaking | 不友好 | 友好 |
| 现代构建工具 | 几乎不支持 | 原生支持 |
| 推荐使用 | ❌ 仅遗留代码 | ✅ |

**代码**

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

**实操建议**

1. 新项目一律用模块,`.d.ts` 文件里也优先用 `declare module`,不要用 `declare namespace`。
2. 老项目从命名空间迁移到模块时,先把所有 `namespace` 改成单独文件 + `export`,再逐步替换 reference 路径。
3. 例外:为浏览器全局变量补声明(如 `declare namespace JSX {}`)还得用命名空间,这是规范规定。

### 说说如何在 React 项目中应用 TypeScript?

**根本原因**

React + TS 的核心是把"组件作为契约"——`props`、`state`、`event`、`ref`、`context` 全部纳入类型系统,让组件之间的依赖在编译期可校验,重构时类型驱动。

**具体机制**

1. **组件 props 类型**:函数组件用 `React.FC<Props>` 或直接 `function C(props: Props)`(更推荐,后者不强制 children)。
2. **事件类型**:`React.ChangeEvent<HTMLInputElement>`、`React.MouseEvent<HTMLButtonElement>`。
3. **Hooks 类型**:`useState<T>(init)`、`useRef<T>(null)`、自定义 hook 加返回类型。
4. **Context**:`createContext<T | null>(null)`,消费侧做空值收窄。
5. **泛型组件**:`function List<T>(props: { items: T[] })`。
6. **组件库 props**:用 `ComponentProps<typeof Button>` 复用三方组件类型。

**代码**

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

**实操建议**

1. 不推荐 `React.FC`——它强制带上 `children` 类型,且对默认参数推导有坑,直接 `function Comp(props: Props)` 更干净。
2. `useRef` 给 DOM 用 `useRef<HTMLDivElement>(null)`,给可变值用 `useRef<number>(0)`,两者初始值含义不同。
3. 三方组件 props 用 `ComponentProps<typeof Button>` 复用,不要手抄一遍。

### 说说如何在 Vue 项目中应用 TypeScript?

**根本原因**

Vue3 的组合式 API 设计时就考虑了 TS,`ref`、`reactive`、`computed` 都能精确推导。`<script setup lang="ts">` 用编译宏(`defineProps` / `defineEmits`)实现"写类型即声明",让 TS 在 Vue 项目里几乎零摩擦。

**具体机制**

1. **`defineProps<T>()`**:用泛型直接声明 props 类型,编译后自动转成运行时声明。
2. **`defineEmits<T>()`**:声明事件载荷类型。
3. **`ref<T>()` / `reactive<T>()`**:精确推导。
4. **`computed`**:返回类型由计算函数自动推导。
5. **`provide` / `inject`**:用 `InjectionKey<T>` 携带类型信息。
6. **路由元信息**:扩展 `RouteMeta` 接口,所有路由 `meta` 都有类型。

**代码**

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

**实操建议**

1. SFC 一律 `<script setup lang="ts">`,用 `defineProps<T>()` 而非运行时声明,体验最好。
2. 跨组件共享类型放 `types/` 目录或单独包,不要在每个组件里重复声明。
3. Pinia 的 store 用 setup 风格(组合式)写,类型推导完全自动;Options 风格在大型 store 上类型丢失严重。
4. `vue-router` 的 `meta` 加全局声明:`declare module 'vue-router' { interface RouteMeta { ... } }`,所有路由 `meta` 都有提示。

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
