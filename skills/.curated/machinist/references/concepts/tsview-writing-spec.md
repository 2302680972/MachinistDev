# TS 视图编写规范

TS 视图是"给编辑器看的脚本视图"：看起来像 TypeScript，但不是 JS/TS 程序。在这里写的内容会被校验后写回到原版 BeScript 脚本。

这份文档只说两件事：
- 在 TS 视图里**应该怎么改**，才能被接受并正确写回。
- 哪些信息是"定位/关联用"的，**不要乱动**，未被许可的修改会直接被拒。

## 本篇范围

- 只讲 TS 视图的结构、语法约束、写回边界与“禁止改动”的视图上定位信息。
- 机械复用、投射物对象池、碰撞层、UI/Canvas 运行时规则等特性和技巧不在此处,应该到相应位置查找。

## 1. TS 文件分类

### 1.1 零件 TS 聚合视图 (一个文件对应一个零件的整体视图)

- 一个零件对应一个 `export class Device_...`。
- **使用完整 TS 语法**：变量声明、赋值、调用、控制流，会被完整解析转换为 BeScript。
- 全局变量是 class 的 `private` 字段，并用上一行的 `// @var("原名")` 标记回原名。
- 私有自定义脚本使用 `protected` 修饰符（允许机械产生器虚拟核心继承访问）。
- 事件脚本必须放在事件区段 `// [Event ...] BEGIN/END` 之间。
- 自定义脚本必须放在脚本区段 `// [Script] BEGIN/END` 之间，且方法名使用 `Script__...` 前缀。

### 1.2 CodeRepo.ts（代码仓库视图：只读索引 + 原始命令行）

- `CodeRepo.ts` 用来**定位/引用**代码仓库脚本的符号。
- **仅限 CodeRepo**：方法体用 `//|` 原始命令行格式，不做 TS 语义转换。此规则不适用于零件视图。
- 原版注释行展示为 `//|注释内容`，不要写 `//|//注释`，也不应出现 `@3@3...` 这类转义串。
- 当前 `CodeRepo.ts` **禁止写回**；不要手改它。
- 需要“提取为代码仓库 / 关联到代码仓库 / 删除代码仓库脚本”时，用重构动作：
  - `script.toRepository`
  - `script.linkRepository`
  - `script.deleteRepository`

### 1.3 d.ts声明

- sdk.ts SDK声明
- custom-logic.d.ts 自定义内容索引: 这里是一个快速索引,用于解决循环依赖的问题和提供对外接口定义来源. 此处禁止主动修改,应随零件TS视图修改而被动更新.

## 2. 核心约束

### 2.0 写入范围

- 地图/机械/零件（实体本身）的增删改不在支持范围内；允许读取，但写入仅限于“零件 class 内部”的脚本/变量更新。
- 若需求涉及新增/删除/重命名零件、调整零件挂载、修改地图/机械结构等，必须明确告知用户由其手动完成后再继续。
- 禁止脱离实际而强行假设具有允许范围外的修改能力

### 2.1 禁止当成 JS/TS 写

- 禁止 JS/TS 标准库与外部库（例如 `Math.*`、`Array.*`）。
- 禁止基础运算符（`+ - * / == >` 等），必须改成平台调用（`G.*` 或变量方法）。
- 禁止对象/数组字面量（`{}`、`[]`）。
- 禁止 `new`、函数表达式、`let`/`const`。
- 禁止主动使用 `as` 强转。如果视图中出现 `as`，说明底层存在类型不一致，应分析原因并尽量消除。
- 控制流只允许 `if` / `while` / `break` / `continue` / `return`；禁止 `else` / `switch` / `try/catch` / `for` 等。

### 2.2 局部变量必须走 locals 区

方法体里如果需要局部变量：
- 必须写在 `// @locals-begin` 与 `// @locals-end` 之间；
- 用 `var 名称: Be类型` 声明；
- 后续只做赋值与调用，不要再新增声明语句。

示例（零件 TS 视图片段：`Script__限制昵称长度`）：
```ts
      /**
       * name: 限制昵称长度
       * guid: 092f7874c1f54dc19287189d1a2e76ee
       * repo: CodeRepo.通用逻辑.限制昵称长度
       */
      @bsColor(255, 132, 255)
      @CodeRepo.通用逻辑.限制昵称长度()
      public Script__限制昵称长度(w: BeString): BeString {
        // @locals-begin
        var N: BeFloat
        var r: BeString
        // @locals-end

        N = w.length()
        if (G.float.gt(N, BeFloat.fromBeConst("8"))) {
          w = w.substring(BeFloat.fromBeConst("0"), BeFloat.fromBeConst("6"))
          w = G.string.add(w, BeString.fromBeConst("..."))
        }
        r = G.create.string(w)
        // --- 隐式返回勿修改
        return r
      }
```

示例（零件 TS 视图片段：`while` + `if` 嵌套）：
```ts
        time = G.create.float(BeFloat.fromBeConst("0"))
        X_continue = G.create.bool(BeBool.fromBeConst("1"))
        first = G.create.bool(BeBool.fromBeConst("1"))
        while (G.create.bool(X_continue)) {
          if (G.bool.not(first)) {
            timeD = G.time.deltaTime()
            time.addEqual(timeD)
          }
          rate = G.float.division(time, duration)
          index = G.long.lerp(start, end, rate)
          name = G.create.string(index)
          length = name.length()
          if (G.float.Approximately(length, BeFloat.fromBeConst("1"))) {
            name = G.string.add(BeString.fromBeConst("000"), name)
          }
          name = G.string.add(prefix, name, BeString.fromBeConst(".png"))
          UI.loadPic(name)
          first = G.create.bool(BeBool.fromBeConst("0"))
          X_continue = G.float.lt(time, duration)
          G.delay(G.create.float(BeFloat.fromBeConst("0")))
        }
```

### 2.3 常量写法

- 数值/布尔/字符串常量通常写成 `Be*.fromBeConst("...")`，不要直接写 `8`、`true`、`"abc"`。
- 例外：内联 Canvas（见 §6.1）会出现反引号包起来的 HTML 文本。

### 2.4 返回值与分支

- 返回值固定指向一个标识符：不要在不同分支里 `return` 不同的变量/表达式。
- 也不要做"未赋值就使用"的分支写法（原版局部变量有状态，行为会变得不可预测）。

### 2.5 局部变量有状态的处理

原版局部变量跨调用保持状态，TS 视图中会出现 TS2454 警告。按以下规则处理：

**安全初始化**：纯计算用途的临时变量，不涉及 UI/Canvas，不需要跨调用保持值，可直接初始化：
```ts
var 临时计算: BeFloat = BeFloat.fromBeConst("0")
```

**禁止初始化**：
- Canvas/UI 类型变量：随便初始化会导致多重 UI 对象创建
- 需要跨调用保持值的变量：应提升为全局变量而不是初始化

### 2.6 尾部隐式返回

每个方法体末尾会自动附加一段不可改动区段：

```ts
        // --- 隐式返回勿修改
        return $ret
      }
```

- 这是原版隐式返回的体现，无论方法体内是否已有 `return` 语句都会存在。
- 注释 `// --- 隐式返回勿修改` 和紧随其后的 `return` 必须保留，不要删除或修改。
- 写回时会自动忽略这段区段，不会重复写入。

## 3. 禁止修改的部分

### 3.1 事件区段边界不要动

- `// [Event <事件名>] BEGIN` 与 `// [Event <事件名>] END` 这一对边界必须保留。
- 允许在区段内部增删方法；禁止新增/删除/重命名事件区段本身。

示例（零件 TS 视图片段：事件区段 + 事件方法命名）：
```ts
      // [Event mechStart] BEGIN
      /**
       * name: GenericInitialize
       */
      @bsName("GenericInitialize")
      @bsColor(8, 0, 255)
      private Event_mechStart__GenericInitialize(): void {
        // @locals-begin
        // @locals-end

        this.Self = Act.self<Device_礼物替身_282>(this).getMech()
        this.map = G.map.getMapMech()
        this.map_handler = this.map.findDevice(BeString.fromBeConst("Scenario"))
      }
      // [Event mechStart] END
```

### 3.2 “来源标记”不要乱改

每个脚本方法的 DocBlock 里会带用于定位来源的信息（常见是 `name:`，以及 `guid:` 或 `sourcePath:` 之一）。
- 这些字段不是"重命名/换路径"的入口；改了很容易导致写回找不到目标，或者直接被拒绝。
- 实际上不需要改这里,而是只需要让其自动刷新

### 3.3 全局变量标记不要乱改

全局变量必须是 class 的 `private` 字段，并且上一行用 `// @var("原名")` 绑定原名：
```ts
      // @var("map_handler")
      private map_handler: BeDevice = BeDevice.fromBeConst("");
```
如需修改,应该二者同步修改,否则很可能被拒绝

## 4. 命名与转义

这里说清楚 3 件事：

1. 脚本“原名”（原版名字）
2. TS 方法名
3. 标记/装饰器之间如何互相对应

### 4.1 脚本原名

- 脚本的“唯一真名”来自 BeScript 源：
  - `.code` 文件名（去掉 `.code` 后缀），以及/或者
  - 脚本头里的 `name:` 字段
- 在 TS 视图中，这个原名在 3 个位置必须保持一致：
  - DocBlock 中的 `name: <原名>`
  - 装饰器：`@bsName("<原名>")`（底层等价于 `@BeScript.name("<原名>")`）
  - 写回时用来定位的脚本条目名（`list.json` 中的 name）

`<原名>` 是**未转义**的名字，可以包含 `'`、中文等：
- 不要把 `'` 手动写成 `_x27_`；
- 不要在原名里加 `Script__` / `Event_` 等前缀；
- 不要在 `@bsName`/`@BeScript.name` 里写 TS 方法名。

### 4.2 TS 方法名（前缀 + 转义）

- TS 方法名只负责“在 TypeScript 里合法且可读”，不会改变脚本原名。
- 生成规则：
  - 自定义脚本：`Script__<转义后的脚本名>`
  - 事件脚本：`Event_<事件名>__<转义后的脚本名>`
- 转义规则：按 `ToBaseIdentifier` 把不合法字符编码，例如把 `'` 变成 `_x27_`：

```text
Utils'MoveByCameraLocal -> X_Utils_x27_MoveByCameraLocal
Generic'PlayAnimation   -> X_Generic_x27_PlayAnimation
```

写回时：
- 写回只依赖来源标记（`name/sourcePath/guid`）还原脚本，不依赖方法名本身；
- 但 `mechtoolkit.patch_mech_view` 会用"脚本原名 + 前缀 + 转义规则"推导出期望的方法名，并与实际方法名比对，若不一致会直接拒绝写回。

### 4.3 常用前缀约定（复述）

- 自定义脚本：`Script__<...>`
- 事件脚本：`Event_<事件名>__<...>`
- 底层内置 Act 无类似前缀, 因此在 TS 视图中很容易区分出哪些是自定义脚本、哪些是内置方法

## 5. "//|" 与禁用行（只讲展示）

**重要区分**：零件视图中 `//|` 仅用于用户注释，代码语句使用完整 TS 语法。

- 零件视图中的用户注释用 `//|注释内容`
  - 不要重复写 `//`
  - 不要出现 `@3@3...`
  - 例：原版 `//@3@3@3@3@3` 在 TS 视图中显示为 `//|=====`
- CodeRepo 的原始命令行用 `//| <原版命令行>`
  - 仅限 CodeRepo，零件视图不适用
  - 这不是 TS 语句，只用于展示原版命令行文本
- 禁用行用 `/* <一行可解析的 TS 语句> */`
  - 一行只写一条 TS 语句
  - `/*` 后固定一个空格，`*/` 前固定一个空格
  - 缩进写在 `/*` 外面，禁用语句本体不额外加前导空格
  - 禁止写 `/*fun ...` 或 `/*  fun ...` 这类原版禁用格式
  - 写回时会自动转换为原版格式，行首 `/*`，随后按语句层级缩进，再跟随原版命令行

示例，禁用行写法。来源为内置测试工程 TS 视图节选：
```ts
        while (G.float.lt(列表编号_横, 循环次数_横)) {
          布尔_碰撞开关 = G.readExcel2<"BeBool">(对象层序号_整数, 列名, BeString.fromBeConst("碰撞层级设置"))
          G.layer.enableCollide(列表编号_纵, 列表编号_横, 布尔_碰撞开关)
          /* const a: BeString = G.string.add(列表编号_纵, BeString.fromBeConst("和"), 列表编号_横, 布尔_碰撞开关) */
          /* G.debug.print(a, BeColor.fromBeConst("255,255,255,255"), BeBool.fromBeConst("0")) */
          列表编号_横 = G.float.add(列表编号_横, BeFloat.fromBeConst("1"))
        }
```

示例，原始命令行写法。来源为内置测试工程 CodeRepo TS 视图节选：
```ts
    export function X_Launcher_x27_LaunchUniformlyAuto(): RepoDecorator<"117ef6cc070042c092a3db9b0552ef7c", (type: BeLong, aimedX: BeFloat, aimedY: BeFloat, amount: BeLong, maxIncAngle: BeFloat, offset: BeFloat, limited: BeBool, skillKey: BeString, extraCost: BeLong, HPMul: BeFloat, crit: BeFloat, initHp: BeFloat, useManualSpeed: BeBool, manualSpeed: BeFloat) => BeLong> {
      //| Long amount=fun long.min(var amount:var maxL)
      //| if{fun long.lte(var amount:Long 0)
      //|   fun break()
      //| if{fun long.equal(var amount:Long 1)
      //|   Struct player=act 1.this.Skills'GetFirstElement(var skillKey)
      //|   if{act 1.this.Launcher'Launch(var type:var bulletDir:var offset:var player:var HPMul:var crit:var initHp:var useManualSpeed:var manualSpeed)
      //|     =act 1.this.Skills'Decrease(var skillKey:var totalCost)
      //|   fun break()
      //| while{fun long.lt(var count:var amount)
      //|   Bool success=act 1.this.Launcher'Launch(var type:var bulletDir:var offset:var player:var HPMul:var crit:var initHp:var useManualSpeed:var manualSpeed)
      //|   if{fun create.bool(var success)
      //|     =act 1.this.Skills'Decrease(var skillKey:var totalCost)
      //|     varf Long.count.addEqual(Long 1)
      //|   if{fun bool.not(var success)
      //|     fun break()
      return ((value: (type: BeLong, aimedX: BeFloat, aimedY: BeFloat, amount: BeLong, maxIncAngle: BeFloat, offset: BeFloat, limited: BeBool, skillKey: BeString, extraCost: BeLong, HPMul: BeFloat, crit: BeFloat, initHp: BeFloat, useManualSpeed: BeBool, manualSpeed: BeFloat) => BeLong, context: RepoDecoratorContext) => {}) as any;
    }
```

## 其他相关内容应查找对应文档

- 单位/投射物复用、对象池与碰撞层规则：见 `../tips/projectile-collision-detector.md`
- 机械生成器与注入机制：见 `../tips/mech-reuse-guide.md`
- UI/Canvas 运行时规则与组件管理：见 `canvas-component-guide.md`
- UI 动画机制：见 `../tips/canvas-animation-mechanism.md`

## 相关文档

- `view-directory-structure.md`
- `canvas-component-guide.md`
- `../tips/projectile-collision-detector.md`
- `../tips/mech-reuse-guide.md`

