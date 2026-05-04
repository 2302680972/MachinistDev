<!-- mechtoolkit: 以下是mechtoolkit文档 -->
# CreatAI平台开发指引

## 必读规则

本文档保存为两份: AGENTS.md和CLAUDE.md 这是有意设计的不是所谓冗余,若排查冗余应该跳过此文件
本文档来自于Toolkit工具箱,按模板生成,不建议把任何的额外信息写入到这里.此文档会被覆盖

### 开发场景

- 本工作区是CreatAI平台的游戏工程
- 你的任务: 协助一名策划完成工程的开发与维护,表述风格应简洁易懂注重实用.无需引入一切团队/企业场景假设和流程,禁止报告式的表达风格.
- Skill参考: `machinist`提供了常用的开发指引
- 游戏工程通常是弹幕互动游戏(并非"东方弹幕游戏",注意区分!!!若无明确说明则"弹幕游戏"视为"弹幕互动游戏")或者单机小游戏.辅助设计玩法时应明确游戏设定且考虑此类场景常见需求.
- 技术找固定且受限: 
    - Machinist(CreatAI)平台将提供唯一的标准库SDK.不存在任何"外部依赖"或者"技术栈",也禁止相应的脱离实际的表述,比如`DevOps`/`环境变量`/`配置`.
    - 本仓库当中的常见文件或格式:
        - BeScript脚本(`*.code`文件): 原始BeScript文件,不保证可读性.编辑操作需通过用专用工具(`patch_mech_view`)编辑TS视图间接完成.禁止直接编辑的尝试,通常也不需要直接读.
        - Canvas二进制文件(`*.canvas`文件): 二进制文件,不要尝试直接读.编辑操作需通过专用工具(`patch_mech_view`)编辑HTML视图间接完成.禁止直接编辑的尝试,通常也不需要直接读.
        - Hjson文件(`*.json`和`*.guid`): Hjson文件.禁止直接编辑也不提供任何编辑支持.guid文件当中包含guid字段,可以全局搜索guid值定位相关文件的位置.
        - TS文件(`*.ts`): TS视图文件,按照语义将原始BeScript翻译之后得到的可编辑的视图.保存在`temp/toolkit/views`当中,按照规则与地图或机械中的零件一一对应.编辑操作需要通过专用工具(`patch_mech_view`)编辑.禁止直接编辑的尝试,直接编辑是无效的且会立即被抹平.
        - HTML文件(`*.html`): HTML视图文件,按照语义将二进制或者其他格式的布局翻译为可读的HTML视图.保存在`temp/toolkit/views`当中,按照规则与独立的`*.canvas`文件一一对应.相应地还有一部分嵌入在code文件中的canvas,这些canvas的视图位于TS视图里面.编辑必须经过专用工具(`patch_mech_view`)完成,直接编辑是无效的且会立即被抹平.
        - PNG文件(`*.png`): PNG预览图,Toolkit根据canvas所在的原始位置生成的可读图片预览.注意其位置是与原始canvas所在位置一一对应(比如`*.code`的位置或者`*.canvas`的位置,而不是TS视图或者HTML视图的对应位置)视图中通常会提供相应预览图位置的指引
    - 应关注哪些内容:
        - 若需要编辑逻辑,则读相应TS视图和编辑TS视图即可
        - 若需要编辑布局,则读TS视图或HTML视图并编辑
        - 无需在原始BeScript文件内容上纠结
    - 若需实现某功能则应查询SDK进行取证.SDK参考文档位于`temp/toolkit/bescript_reference/sdk.ts`,包含平台全部API的类型声明和调用签名.一切设计必须有事实依据,禁止无依据的猜测!
- 工具使用限制:
    - 对于读操作: 无任何的特别约束,可以自由使用原生工具
    - 对于写操作:
        - 若写入工程使用的代码等专用格式,务必遵守规范使用`patch_mech_view`和`refactor_*`等工具.
        - 若写入普通文本比如Markdown格式,则任意使用原生工具,无强制使用特定工具的约束.但不要假设`patch_mech_view`等能够编辑这些无关文本.
        - 绝对禁区: 严禁使用Python和sed直接编辑code/canvas/ts/html文件! 滥用无关工具将会导致严重的破坏!
        - 绝对绝对禁止直接越过工具手动改code!!!无论遇到多大的困难!

### 代码风格要求

- 在能完成工作的前提下,使用尽可能少的代码
- 禁止历史债务的存在,禁止滥用"兼容"的说法
- 用户追求高质量的简洁代码,请尊重用户的代码洁癖并保持代码整洁

#### Engineering Quality Baseline

- Follow SOLID, DRY, separation of concerns, and YAGNI.
- Use clear naming and pragmatic abstractions; add concise comments only for critical or non-obvious logic.
- Remove dead code and obsolete compatibility paths when changing behavior, unless compatibility is explicitly required by the user.
- Consider time/space complexity and optimize heavy IO or memory usage when relevant.
- Handle edge cases explicitly; do not hide failures.

#### 性能需求

- BeScript是一种脚本语言,这种语言天然地存在性能上的缺陷且任何的冗余都会快速放大这种缺陷.
- 禁止非必要的防御和重复检查: 过度防御将会明显对游戏性能产生负面影响,过度的防御逻辑会严重破坏用户的体验!
- 负面清单:
    - 费时操作: 每帧重复进行高耗时动作(如: 创建机械/创建新UI对象/重绘UI/调整机械或零件的物理属性/绑定或者移除事件回调/开启和关闭循环/用变量名称的动态调用/网络IO/文件IO)
    - 内存泄漏: 非必要的一次性对象(如: 用过即弃的机械/复杂UI/大量字典/大量结构等等),以及用过但不主动清理的行为
    - 低效算法: 低效率尤其是高时间复杂度的计算逻辑(如: 达到N^2的时间复杂度的算法设计)
    - 未知时序: 不确定的时序将导致行为不可预测
    - 重入递归: 全部方法都无法重入,重入部分场景将导致隐藏的错误
    - 有状态设计: 利用局部变量有状态的特性跨调用传递信息
    - 大量硬编码: 硬编码内容难以维护
    - 频繁网络IO: 尤其是频繁上传存档/频繁上传弹幕互动游戏数据等等,将会对平台服务器性能产生严重负面影响
- 推荐思路:
    - 时间优先: 用空间换时间(用更大的缓存减少重复计算)
    - 数据扁平化: 使用ECS处理大量同质化的数据,避免大量字典/结构操作
    - 中心化调度: 使用中心化控制器管理时序提高可预测性
    - 复用对象池: 机械/UI对象使用池进行管理,在不使用时隐藏(机械的"关闭全部功能"方法和UI的"隐藏"方法)
    - 表驱动设计: 表格存储关键设定,便于统一管理,用于正式版本
    - 运行时调参: 便于测试微调属性和规则,最终仍落到表格

## 格式规范

### BeScript格式和TS视图规范

- 概述:
    - BeScript: BeScript是一种脚本语言,一个方法保存为一个.code文件.
        - 内联脚本: 文件放在零件的脚本组目录内的脚本,只有固定唯一的使用者
        - 独立脚本(coderepo): 文件独立,有guid,可按guid导入多个不同的脚本组当中被复用
    - TS视图: 使用受限TS语法便于读写的文件.TS与原始BeScript的`.code`文件不同,以零件为范围进行聚合,并映射为一个类.
        - TS视图当中按照实际运行时会持有的内容进行展示,所以链接的独立脚本会直接转为视图中的方法,这些独立脚本被称为coderepo; coderepo在被链接时,相应零件的TS视图当中是完整的代码,与非独立脚本区别不大.
        - CodeRepo有一个独立的只读TS视图,这个视图仅用于辅助理解链接关系和提供语法约束,禁止尝试修改此文件(也不在可修改的范围内)
        - TS视图的路径规则为: 地图在视图目录所对应的位置的直接下级
            - 示例: `temp/toolkit/views/玩法地图.map_/327.ts` 对应`玩法地图.map_/327`零件
            - 示例: `temp/toolkit/views/机械/辅助调度器.mech_/0.ts` 对应`机械/辅助调度器.mech_/0`零件
- BeScript表达式和调用规则:
    - 脚本组: 在零件当中存在多个脚本组,分为事件(`event_*`)和自定义方法(`scripts`),其中事件的签名完全固定,而自定义方法则可以自定义签名.
    - 事件脚本和自定义脚本:
        - 自定义脚本可以作为Act的目标,而事件脚本不可以
        - 事件脚本无权限设置也无法被任何脚本调用(只能由底层触发),自定义脚本可以设置是否私有(在视图中为protected,为了与事件脚本的private区别)
    - 调用关键字: 按照关键字调用主要分为fun/varf/act.
      - fun调用: 在TS视图为`G.*`,并有泛型和可变参数优化
      - varf调用: 在TS视图为类型的成员方法
      - act调用: 在TS视图为Act.*调用,具体分为:
        - Act.self<类型名>(this).方法名 调用自身方法(含底层Act+私有自定义方法+非私有自定义方法)
        - Act.byName<类型名>("零件别名").方法名 按别名调用特定零件的方法(底层Act+非私有自定义方法)
        - Act.byId<类型名>(id号).方法名 按零件ID调用目标零件的方法(底层Act+非私有自定义方法)
    - 格式特性:
        - 代码区块: 为了准确表达逻辑所设立的结构,是不可修改内容.绝对禁止破坏脚本组边界注释等固有结构的尝试和假设.关注点应该在允许修改的全局变量/脚本上
        - 方法内部规则
            - 所有方法禁止重入,禁止一切直接或间接递归
            - 脚本的每一行必须是表达式,且是唯一的方法调用.在TS视图当中,除注释之外每一行都必须是唯一的调用(fromBeConst除外),禁止嵌套(fromBeConst是表达常量所用的特殊格式)
            - 在BeScript当中break()可以表达返回或者跳出循环,在TS当中这会根据循环深度映射为`break`或者`return *`.由于BeScript的语法限制,所以TS视图的循环当中也禁止直接return.
            - BeScript的控制流关键字有限,所以TS视图当中只能使用`if`/`while`/`break`/`continue`/`return`关键字,禁止滥用`for`/`switch`/`else`/`elseif`等无法还原为BeScript脚本内容的结构
            - BeScript当中有`禁用行`格式,相应地在TS视图当中,`/* <语句> */`也作为禁用行格式而不是常规注释.`/* <语句> */`当中只能包裹单一表达式(对应BeScript的一行),但不强制只能是TS视图的一行.(针对内联canvas这个特例,会有多行被一个`/* <语句> */`格式包围)
            - 每个BeScript脚本当中至少有一行有效的代码内容,无论是否禁用,是常规调用还是注释. 完全无内容的脚本会被游戏平台自动抹平.若要删除脚本,应先删除所有调用处,再将整个脚本声明一次性删掉,禁止留下空壳
            - BeScript脚本当中使用缩进和大括号确定层级,在TS视图当中,大括号是唯一标准来源,必须保持完整
            - 返回值规则:
                - BeScript脚本元数据上声明返回值类型和变量名(固定用这个变量返回,无法修改)
                - 在TS视图上,`BeScript({ retVar: "变量名" })`+方法体内的`var`声明+每处返回(含末尾隐式返回)均必须保持一致性
                - 方法体末尾的`return;`/`return 变量名;`是隐式返回,转为code时自动剥离;若TS产生双重return则说明原始code在最后一行写了一个多余的break.(BeScript当中最后一行执行完毕则自然退出无需额外"break"手动返回)
        - 脚本声明格式: 自定义脚本和事件脚本均使用属性赋值+`BeScript`/`BeScriptAsync`柯里化调用的形式
            - `BeScript({ name, color, comment, ... })` 中的字段:
                - name: 原始名称(必须)
                - color: CreatAI上的脚本颜色标记,仅视觉效果
                - debug: 在游戏中运行此脚本会输出详细日志
                - skip: 屏蔽此脚本的调用
                - private: 自定义脚本中使用则其他零件无法调用
                - comment: CreatAI上的脚本用途说明,纯视觉效果
                - repo: 代码仓库脚本的GUID(链接脚本时使用)
                - retVar: 返回值变量名(BeScript当中的写法)
            - 自定义脚本: `public 属性名 = BeScript({ name: "原名" })((参数) => { 方法体 });`
            - 事件脚本: 位于`static { }`块中,按事件类型通过`this.registerEvent_xxx(this, { ... })`注册,成员为`BeScript`调用。lambda使用`function (this: DeviceName, 事件参数...)`形式,其中`this`是注入的设备类型声明,使方法体内`this.xxx`的泛型调用获得正确类型。**不要移除或修改`this`参数**
            - 事件脚本组: `this.registerEvent_xxx(this, { ... })`的外壳禁止修改或删除,表明这是零件类的固有结构!若需删除事件脚本,则必须只能删除脚本本身的定义,而事件注册外壳必须保留!即使是空的`this.registerEvent_xxx(this, { });`也必须保留!
            - 含delay或await的脚本使用`BeScriptAsync`+`async`箭头函数,但这只表示方法体可等待,不等于调用方会等待
            - 调用方是否阻塞只看返回值: 异步无返回值时调用方立即继续; 异步有返回值时调用方必须await
            - 若只想控制时序,可以专门加入一个占位返回值; 即使调用方后续把这个返回值丢弃,也仍然会形成阻塞
        - @events(...)装饰器: 声明零件上的合法事件列表,此装饰器自动生成不得删除
        - 有意设计的缺陷标记:
            - TS视图当中会使用一些明显有语法错误或者语义明确有问题的格式表达BeScript当中的错误.新增内容中不得出现类似的错误
            - 比如调用悬空的Act/类型不匹配时的as强转/undefined强行占据参数位/赋值前使用局部变量等.
            - 可通过读Lint定位TS语法层面的错误具体位置
        - 注释规则:
            - 哪里可以使用(常规的)注释:
                - 全局变量和方法级别可以使用"comment"字段充当注释
                - 方法体内的普通注释直接使用`//`
            - 不可以假设允许自定义注释的常见场景:
                - 方法体中的`var`声明: 不支持在var声明行上添加自定义注释
                - TS类级区段之间: 除固定区段边界结构外,不支持任何注释,无法写入自定义的注释
                - 方法的JSDoc: 这是完全自动生成的,不具备写入信息的能力.不可修改
                - 类定义之外: 顶部固定文档不可编辑.不支持任何的自定义注释
- 脚本和TS属性命名映射:
    - 在BeScript当中`'`符号是原版分组符号(仅第一个生效)。新脚本不建议使用`'`，分组效果可通过重排序近似实现，建议用`_`替代。Lint 会对`'`发出 Warning。
    - TS 属性名规则: 规范化后的标识符(自定义脚本和事件脚本均无额外前缀).
        - 规范化规则: 原始脚本名本身是合法 TS 标识符且不是关键字时，直接用原名/否则由工具生成以 `X_` 开头的标识符，并把非法字符编码成 `_xNN_` 片段
        - `BeScript({ name })` 中的 name 是原始名称,属性名是转义后的标识符,二者必须保持一致性否则将会导致工具拒绝写入
    - 事件列表约束:
        - 零件类型对应着确定性的事件列表.在TS视图当中,新增或修改事件脚本时以 TS 视图中现有 @events(...) 和 `static { }` 块中的 `this.registerEvent_xxx` 声明为唯一准则
        - 事件签名的权威来源是 `temp/toolkit/bescript_reference/sdk.ts` 中自动生成的 `TsViewEventSignatureMap` 与 `TsViewEventMethod`；事件脚本参数必须与该签名一致，禁止自造签名
    - 动态调用参数规则:
        - 指定零件动态调用方法时,直接填方法的`原始BeScript名称`!比如`UI'清空`这个名称绝对不可以乱填什么`X_UI_x27_清空`
        - 使用调用地图方法逻辑比如`callClientMapRet(分机+有返回值)`和`callMapClientCoreAct(分机方块无返回)`和`callMapCoreAct(主机方块无返回)`,都需要使用`零件逻辑名.原始BeScript名称`的格式比如`UI'清空`应写为形如`UI管理模块.UI'清空`
    - 为什么TS和BeScript方法名不一样: 
        - 方法组压缩: 在生成TS视图时有压缩机制,底层方法已经被转义和优化了表达形式.
        - TS转义: 避免TS语法不支持
        - TS视图上方法和对应的原始code不是完全一样是完全正常的,不要在这种事情上面纠结!正常编辑TS视图即可,工具会辅助你把TS视图转回合适的原始code内容
- 变量/常量规则:
    - 变量
        - 变量有状态,可被Struct有状态地绑定.推荐在读Struct时选择复制
        - 变量的值可跨调用保持但不推荐.TS 视图中会出现 TS2454 警告"Variable is used before being assigned".遇到时应该进行调整和重写
        - 为了代码质量,应在需要长期状态时用全局变量,而不是依赖因素状态.
    - 常量
        - 在TS视图当中转为fromBeConst(底层Enum被特殊细分,值不是其他类型那样的String)
        - 无法独立存在,必须作为方法参数填入
        - 有语义值类型（Bool/Float/Long/Color/Vector3/String）填具体值：`BeFloat.fromBeConst("1")`
        - 空值占位类型（UI/Bytes/Canvas/Device/Mech/Player/ListN/Dict/Any/Icon/Struct）无参调用：`BeUI.fromBeConst()`
        - 枚举常量必须带泛型和成员：`BeEnum<Foo>.fromBeConst(Foo.Bar)`
        - Canvas新建用专用方法（fromBase64Const/fromNodeConst/fromGuidConst），非创建场景的空占位用 `BeCanvas.fromBeConst()`；fromNodeConst 的第二参数写 Canvas HTML 模板字符串，根元素为 `MechCanvas`
        - `G.create.*` / `G.creatVariable.*` 用于从变量创建值（如 `G.create.string(w)`），禁止用于包裹常量。有语义值类型（Float/Long/Bool/String/Color/Vector3）常量直接写作 `BeXxx.fromBeConst("值")` 传入参数，无需包一层 `G.create.xxx`
    - 全局变量
        - 在BeScript当中,全局变量只能声明在零件这一层面
        - 在TS视图当中,全局变量在类型顶部`// [Globals] BEGIN/END`区段内,只能在这一块声明不得越界
        - 全局变量上一行使用`@bsVar({ name: "原名" })`标记原名,可选`comment`字段
        - 初始化使用`BeXxx.GlobalVarInitialize()`
        - 全局变量仅在其作用域内可见,零件外无法进行引用
	    - 局部变量
	        - 在BeScript当中,局部变量只能声明在方法这一层面
	        - 在TS视图当中,只有存在局部变量时才需要写`var`.没有返回值且没有局部变量的方法,可以完全没有`var`声明,纯注释方法或只操作全局变量的方法都合规
	        - 返回值变量: 非形参时固定在方法体首行声明`var ret: BeType;`,类型必须与`BeScript({ retVar })`一致
	        - 普通局部变量: 在首次赋值行内联声明`var a: BeType = call(...)`,后续赋值只写`a = call(...)`不重复声明
	        - 局部变量禁止与全局变量或形参重名
        - 任何局部变量均需要被至少使用1次才允许声明.patch_mech_view时需遵守以下规则:
            - 新增变量时,必须确保在一次patch_mech_view调用当中同时完成声明和首次使用.悬空的声明会被抹除所以只写声明而不用无意义;而未声明而直接使用则会直接导致语法错误!
            - 删除变量声明时,若代码里仍保留对该变量的使用则会被拒绝
- 重要避坑:
    - Excel相关:
        - findExcel2 搜索返回的是id本身: 在TS视图当中`G.findExcel2` 禁止用 `"id"` 作为搜索列.正确做法是用普通列查到行 ID，再用 `readExcel2` 按 ID 读取其他列.是严禁不是不推荐!是必然导致报错!
    - Canvas相关:
        - Canvas对象和UI对象:
            - create.canvas传入布局数据时创建真实的UI对象,禁止滥用;传入空值(BeCanvas.fromBeConst())则创建空Canvas,可用于占位
            - 禁止随意丢弃任何UI对象引用.UI对象无GC机制,丢弃引用会导致内存泄漏!
            - 标准的获取UI组件的方案是: 
                1. 创建Canvas后保存Canvas的引用,用Canvas和Canvas内的路径进行查找
                2. 创建字典在创建时保存引用,然后用引用查找
                3. 非常不推荐使用"查找子UI组件"这种确定性极低的方式进行UI对象查找
            - 非常不推荐`findCanvas`,因为Canvas可以被多次实例化而导致部分Canvas丢失引用. 若需传递Canvas,应使用全局变量等等维持引用而不是依赖"按名称查找"
        - Canvas路径和UI父子关系规则:
            - Canvas当中的路径仅取决于其静态内容,动态添加子UI是无效的
            - UI组件可以重新设置父级,相当于从原来父级上移走放到新的父级上
            - UI组件之间的顺序由父子关系+创建顺序决定,为了确定性的效果应该使用图层绑定的设计
        - Canvas默认3D,且3DUI距离设置是每次都调整全局的唯一3DUI距离而不是每个UI组件独立.如果游戏有2D和3D混杂的需求,在每次创建UI组件前应该刷到相应模式
        - 组件特性需注意:
            - 按钮的边框颜色和文本颜色是用的一个属性,全填充情况下必然糊在一起!不推荐使用此处文本字段而是添加`UILabel`类型子级,并在子级上精准设置样式
            - 遮罩元素只有不透明部分会让子组件显示,其他的会裁切掉(预览不够准确,游戏当中是像素级裁切),推荐可以写一个(0,0,0,1)颜色的全填充UI,透明度1/255可最小化干扰,无法完全透明
    - 2D场景相关:
        - 地图中的2D物理等使用的是XY轴而不是Z轴.
        - Z轴位置应该使用0进行填充
    - 布尔值相关
        - 用create.bool和bool.not表示X和非X,而不要滥用`相等`和`与`
    - 延迟和异步相关:
        - delay 是让方法暂停一段时间,至少暂停1帧.通常使用delay表示暂停1帧.(注意这种等待会阻碍方法的返回)
        - delay是BeScript的关键字.在TS视图当中,delay仍然应该视为关键字而不是常规的调用.所以应该是delay(常规调用)而不是delay(BeFloat.fromBeConst("1"))这种错误的直接写常量的格式.乱在这个位置写常量是错误格式
        - 只有带返回值的方法才能阻塞调用者.`BeScriptAsync`/`async`本身不能强制等待,无返回值异步方法仍会立即返回
        - 若要控制异步时序,应通过返回值来制造阻塞.这个返回值可以只是占位,即使后续被丢弃也照样能阻塞
        - 方法是否异步是生成出来的结果,不要试图靠强改这段固定结构控制时序.真正决定因素是: 方法体内是否存在需要等待的内容,以及方法是否带返回值
        - 为了确保异步时序,应在一个机械或地图内使用一个中心化的驱动者统一等待与调度,保证时序可预测
        - 非常不推荐调用其他零件的异步方法,避免重入导致的调用被丢弃和提前返回
        - 非常不推荐使用`loopFun`进行动态的每帧循环,应该改造为静态的每帧循环,由固定的中心化控制者统一调度控制时序
    - CreatAI房间内环境设置:
        - 循环需在启动时用设置上限,否则在500次时会视为死循环(设置的这个上限即阈值)
        - 地图当中可以生成的最大机械数量的最大值是99999,这个值初始是50,可通过设置调整
    - 文本相关:
        - 文本高级拼接当中的占位符是`%+数字`,从%1开始.
        - 建议使用高级拼接输出日志
        - 翻译功能:
            - 需要一个翻译表excel,表头必须有`id`/`name`/`cn`/`en`,由工程`setting.json`指定
            - 静态Canvas翻译: 在HTML视图的文本字段写`@条目名`,运行时按语言表`name`查当前语言并显示到UI文本上.静态预览不解析语言表,会显示原始`@条目名`
            - 运行时翻译: `G.string.translate(key)`直接按语言表`name`查当前语言,key不带`@`,条目不存在会报错
            - 运行时高级翻译: `G.TranslateEx(key, p1..p5)`先翻译key,再替换翻译结果里的`%1`~`%5`.如果某个String参数以`@`开头,该参数会先作为静态翻译key再翻译一次
            - 静态翻译只适合固定UI文案;需要按状态/数量变化的文案应在脚本里用`translate`或`TranslateEx`后赋值
    - 颜色格式:
        - 常量颜色是按0~255计算.0~255范围的颜色文本应尽量抹去小数点后的部分.
        - 易混淆点: 
            - Canvas当中的静态的颜色按[0,255]范围指定值
            - BeScript和TS当中的Color常量也是按[0,255]范围指定值
            - 但是,BeColor的rgba的Get和Set均为0~1范围,而不是标准的0~255,需进行换算
        - `#RRGGBB`这种格式仅在富文本标签里面允许添加,其他地方并不支持
    - 帧相关:
        - 游戏分为图形帧和物理帧
        - 事件`每帧循环`是图形帧,而`物理帧循环`才是物理帧!
        - 物理计算相关的逻辑应该移到物理帧!使用`物理帧循环`的形式将其转为每物理帧调用一次!
        - UI等等用于展示的逻辑应该使用图形帧
        - 物理帧率必须大于等于图形帧,否则会出现严重物理效果bug!!!!!
        - 物理帧比图形帧更为稳定,较少受到帧率影响,而图形帧波动较大不那么稳定
        - 使用`fixedLoopAct`动态指定方法名,实现物理帧循环
        - 目前只有机械核心带`fixedLoop`等物理帧循环(包括事件列表的静态写法和核心静态Act的动态绑定),(机械产生器的虚拟核心注入也行),地图暂无此接口,必要时需桥接.若桥接则必须保持唯一单例,建议地图级启动事件上直接创建和保存避免反复创建浪费内存
        - 可以使用`帧间隔`和`物理帧间隔`确定距离上一帧的时间
    - 机械(零件)的功能:
        - 开启关闭物理: 无物理则不会受到外力移动
        - 开启关闭碰撞: 无碰撞则不会产生碰撞事件可穿模
        - 开启关闭全部功能(Mech.setActive): 这是独立的开关!关掉之后图形/物理/逻辑等均关闭.无需反复单独开关别的功能.
        - 禁止随意丢弃机械对象引用,和字典等类型不一样,机械没有GC,随便乱扔引用会导致内存泄漏!
    - 怎么模仿Null(非推荐做法,非必要禁止使用)
        - 声明一个全局但不使用其赋值.别处只用不赋值,也不要修改其内容
        - 这种做法是对底层缺陷的代偿(比如字典/列表等新建即创建新对象的开销),是明确的Hack,而不是常规做法
        - 这种伪Null可以用来给固定类型成员列表快速填值占位,保持定长列表特性以满足特定场景需求.但禁止在别处滥用
    - 充分利用零件(Device)的Act(用varf补充Act):
        - 大部分的零件Act,直接静态调用即可
        - 部分方法缺少静态路径,则可以利用Device变量类型,先用静态Act获取Device对象再用Device的varf
        - varf方法(即BeDevice类成员方法)提供了静态Act不具备的查询方法存在性/循环调用/禁用方法等等内容,是重要补充,缺失需要的静态Act在SDK定义时,尝试分析varf路径是否可行

### Canvas格式和Html视图规范

#### Canvas文件规范

- 不推荐大量存在内联Canvas,大量内联Canvas不便管理且会导致视图混乱,应尽量导出
- Canvas本身的名称应该与文件名称保持一致性,否则会导致语义上的混乱
- Canvas是二进制内容,禁止直接读和写.一切修改均应该通过Html视图间接完成
- 创建独立Canvas文件的方法:
    1. 当前无直接创建Canvas文件的方法,但可以间接创建
    2. 以`patch_mech_view`在一个方法里面声明内联的canvas行.此时Canvas以内联形式存在
    3. 选择刚创建内联canvas的方法,精准定位那个新增行,将canvas导出
    4. 完成导出后,以独立文件形式存在,继续在HTML文件上`patch_mech_view`
- 编辑后应读图确认: 修改过canvas之后,读图确认真实结果,要整齐美观无错乱!

#### 属性规范

- Schema应参考 HTML 视图头部列出的`temp/toolkit/canvas_render/CanvasTypesNodes.ts`节点定义,而不是无依据地瞎猜!
- 冗余或者缺失必要属性都可能导致错误!
- 属性应符合规范,不要尝试乱猜属性的用处,渲染引擎已明确介绍用处就不要瞎猜!
- Vector3类型参数位置的z轴不要乱填(必须对齐原版格式,虽然那个值并无效果)

#### 节点类型规范(Canvas的UI节点与BeScript的变量类型映射,读取必须遵守此规则)

- Canvas UIRect -> BeScript UIRect (TS BeUIRect)
- Canvas UIButton -> BeScript UIButton (TS BeUIButton)
- Canvas UILabel -> BeScript UILabel (TS BeUILabel)
- Canvas TextPro -> BeScript UILabel (TS BeUILabel)
- Canvas UIInput -> BeScript UIInput (TS BeUIInput)
- Canvas UIRawImage -> BeScript UIRawImage (TS BeUIRawImage)
- Canvas ScrollRect -> BeScript UIRect (TS BeUIRect)

#### 计算规则

##### 布局计算

###### 父空间和对齐方式

- 均设置为9枚举
- 坐标与常见HTML坐标计算完全不同: y轴固定朝上,x轴固定朝右.坐标和尺寸计算中坐标轴均来自父级.在旋转后,子级x轴y轴会有相应变动!
- 父空间决定父级的(0,0)在哪里.但是x轴y轴方向不会变,都是一样的计算方式.不同的父空间会导致可以预测的固定坐标偏移量(取决于父级尺寸)
- 对齐方式决定自身的原点是在哪个位置,同样地x轴y轴不会变.不同的对齐方式会导致可以预测的固定坐标偏移量(取决于自身尺寸)
- 要非常小心canvas的错位问题,父空间和对齐方式如果写错会导致大量的隐蔽问题

###### 尺寸计算

- 绝对: 直接指定像素数
- 比例: 父级的比例(>0&&<=1)
- 边距: 取决于父空间和对齐方式.具体逻辑可以参考`temp/toolkit/canvas_render/CanvasRenderLayout.ts`了解具体底层原理
- 若要做全屏填充,要么父级是全屏时直接按比例计算,要么固定设置远超屏幕大小的绝对尺寸.禁止硬编码(1080*2160)尺寸充当全屏的设计

###### 排版

- 格子排版: 重排全部的子UI的位置,设置子UI组件的尺寸为固定像素值
- 横向排版: 不约束子UI尺寸,但子UI尺寸的计算方式受限,必须是固定像素或者比例值
- 纵向排版: 同样地限制计算方式
- 排版模式的网格模式会强制限制子级长宽.横向/纵向模式则禁用边距模式,子组件本身尺寸仍有效

###### 杂项
- 旋转: 填顺时针旋转角度(任意大小角度值而非"弧度值等等")
- 厚度效果:
    - 方框/按钮等等类型的厚度为-1或者0会导致全填充
    - 正数厚度则按照厚度值向内填充描边,没覆盖到的就是纯透明

##### 颜色

- 颜色决定整个组件的颜色显示,决定方框填充色,也决定九宫格等图片颜色(正片叠底)
- 多个颜色来源时,通常是正片叠底: 组件整体颜色×属性独有颜色=属性本身最终生效值
- 设置UI组件时,颜色范围是[0,255]

##### 其他特性

- 富文本标签支持:
    - <size=XXX> 字号 </size>
    - <color=#RRGGBB> 颜色 </color>

### Excel表格规范

- CreatAI平台在运行时只支持读取文件内第一个表格的内容,其他表格内容无法直接读,但可以用于对第一个表格的辅助运算
- 对于第一个表格有明确格式要求:
    - 第一行是列名
    - 第二行是注释,可空但是这行不可跳过
    - 第三行是类型,明确是`float/long/string/bool/vector/color`之一
    - 第四行及之后是值
    - 第一列的名称是id,类型是long
- 查找ID时,不要在第一列上查id本身,不仅毫无意义还会报错!

### 工程文件结构

- 工程独立实体主要分为: 资源/独立脚本/独立canvas/机械/地图
- 机械和地图平级,地图是扩展后的机械
- 严禁"地图包含机械"假设!!!像"XXX.map_/XXX.mach_/123"这种结构是完全不存在的!禁止这种幻觉!
- 独立实体有guid作为其唯一标识符.但代码当中很少用guid,只有创建canvas时用guid表示文件`.canvas`文件来源,canvas内部用图片的guid指代特定资源
- 禁止把`*.map_`目录和`*.mech`目录及其子目录当作常规文件目录,这些目录是专门放机械和地图本身的内部信息的!
- `*.map_`和`*.mech`的直接子目录是零件目录,零件目录使用纯数字零件id命名
- 禁止一切对`setting.json`/`*.map`/`*.mech`/`map.json`/`mech.json`/`vars.json`/`list.json`/`device.json`/`*.code`/`*.canvas`的直接修改尝试,混乱的尝试通常会直接破坏数据导致游戏中不可用,只有通过MCP间接修改才可以保证安全(目前MCP支持修改TS和Html文件)
- 视图覆盖范围:
    - 零件整体(含全局变量/内联和链接脚本)聚合生成TS视图.
    - 独立canvas文件生成html视图(内联形式的canvas的视图在TS视图内部)
    - 针对coderepo有专门的`temp/toolkit/bescript_reference/CodeRepo.ts`只读索引视图(自动维护,无法主动修改也无需修改)
    - 针对自定义逻辑引用有专门的`temp/toolkit/bescript_reference/custom-logic.d.ts`索引视图(自动维护,无法主动修改也无需修改)
    - 除此之外则无任何其他"视图"概念
- 预览图覆盖范围:
    - 针对独立canvas文件,有按路径映射的同名预览PNG
    - 针对code文件,有按路径映射+脚本内按序号区分的预览PNG,注意这个命名是按原始`.code文件`生成的而不是按视图,但视图当中应该提供了明确的完整路径供查阅
    - 这两种预览图已覆盖全部可能出现的canvas,除此之外无任何其他预览来源

## MCP使用注意事项

### Host 与视图/预览同步

- **Host**（CreatAI/Machinist 工具链侧进程）在工程由工具打开、监听生效时，持续维护 `temp/toolkit/views` 下 TS/HTML视图与工程内原始 `*.code` / `*.canvas` 的一致性，并按规则维护对应的`预览 PNG`。
- **mechtoolkit 等 MCP** 由该 Host 暴露；只要MCP正常运行,文件就不会不同步此时不应该胡乱假设"不同步"而是重新读取最新状态。除非长时间持续下去的异常状态应该通知用户
- 若 MCP 持续异常（超时、断连、工具无响应等），且已排除本机网络不通，则可能Host 已不可用,需重启.提示用户尝试重启 Host 以修复工具链。
- 若频繁遇到MCP的拒绝,应该回顾前面提交的修改是否符合规则
- 当 `patch_mech_view` 或 `patch_mech_view_multiedit` 返回的错误信息说明草稿已保存为 `xxx.ts.draft` 或 `xxx.html.draft` 时，说明本次修改的文本匹配成功但未通过后续校验。此时：
    - MCP 已自动将修改内容保存为 `.draft` 草稿文件
    - 草稿不会干扰 `patch_mech_view` / `patch_mech_view_multiedit` 的正常使用，后续的编辑操作照常进行即可
    - 若要继续这次修改，应读取该草稿文件理解修改内容，修正问题后使用 `commit_draft` 重新提交
    - 若不再继续这次修改，直接忽略该草稿即可，不要为了”收尾”主动调用 `cleanup_drafts`
    - `commit_draft` 会校验基线哈希；若视图内容已变化，会提示该草稿已失效

### 视图换算规则

- temp/toolkit/views 相当于工程根的位置,比如:
    - `temp/toolkit/views/map.map_/54.ts` 对应 `viewRelativePath = map.map_/54.ts`
    - `temp/toolkit/views/layouts/A2.html` 对应 `viewRelativePath = layouts/A2.html`
- 全程使用相对工程根的路径,且禁止`..`这种可能导致越界的表示形式,明确两种路径:
    - ViewPath：相对 `temp/toolkit/views/`，只用 `/`。用于 `mechtoolkit.patch_mech_view.viewRelativePath`
    - WorkspacePath：相对工程根目录，只用 `/`，禁止 `..`，禁止绝对路径。用于 `mechtoolkit.refactor_*` 的各种 `*Path`
- mechtoolkit 不支持任何的worktree 其只维护实际工程根范围内的文件,只会生成在工程本身而不会生成到任何的worktree

### 修改操作的限制

- 注意修改顺序: 必须先有脚本才能添加对脚本的调用,不得先调用再声明
- 为确保修改真实有效,写回后必须复读确认
- `patch_mech_view`为确保准确性,在脚本仍被引用时是禁止更名和删除的,如需更名可以用refactor相关重命名.删除脚本的正确做法:先删除所有调用处(Act调用等),再将整个方法定义(装饰器+签名+花括号+方法体)一次性删掉.禁止只清空方法体而保留空壳
- `patch_mech_view` 校验失败时会自动保存 `.draft` 文件到视图同目录，无需手动备份修改内容。草稿文件与视图共存，视图重建时自动校验草稿有效性（基线哈希匹配则保留，否则丢弃）

## 弹幕互动游戏设计要点

- 测试功能:
    - 受制于CreatAI平台并未提供专门测试框架,所以测试需要设计成正式游戏的一个隐藏部分
    - 惯例是表格配置ID+启动时按住G的逻辑判定.实现当中应与惯例保持一致
    - 测试环境当中的用户本身持有一个虚拟的"弹幕玩家",然后以这个弹幕玩家的身份发送各种模拟消息
    - 为了测试方便,这个特殊弹幕玩家可能需要额外的"切换队伍"支持,即使真正的游戏当中不会这样
    - 测试有时需要模拟机器人玩家,这时应该设计数据结构模拟虚拟的弹幕玩家,并由一个中心化的控制器随机地不定时抽取触发事件模拟消息
- 调参面板:
    - 通常工程中附带调参工具
    - 调参工具是为测试服务的
    - 禁止滥用调参,调参面板不对非测试使用者开放
    - 禁止把调参和设置混淆
- UI设计:
    - 直播间内画面的中上部会有直播间本身的UI,会遮挡.注意避开
    - 需要设计吸引注意力的banner让用户感兴趣.比如制造悬念的标题
    - 礼物图:
        - 通常右下角需设计一个可拖动交互面板,需记忆坐标的功能
        - 交互面板含有指令和礼物效果等信息
        - 交互面板应该简洁明确 展示为`礼物图+描述文字(礼物)`或者`仅描述文字(非礼物)`
    - 交互反馈:
        - 交互或收到礼物时应进行播报
        - 建议设计连击跳动等动态效果增强反馈手感
        - 需要强烈的反馈满足观众虚荣心

Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions
simple and focused.

- Don't add features, refactor code, or make "improvements" beyond what was asked. A bug fix doesn't need surrounding code cleaned up. A simple feature doesn't need extra configurability. Don't add docstrings,
comments, or type annotations to code you didn't change. Only add comments where the logic isn't self-
evident.

- Don't add error handling, fallbacks, or validation for scenarios that can't happen. Trust internal code and
framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.

- Don't create helpers, utilities, or abstractions for one-time operations. Don't design for hypothetical
future requirements. The right amount of complexity is the minimum needed for the current task—three similar
lines of code is better than a premature abstraction.

- Avoid backwards-compatibility hacks like renaming unused _vars, re-exporting types, adding // removed
comments for removed code, etc. If you are certain that something is unused, you can delete it completely.

<!-- mechtoolkit: 以上是mechtoolkit文档 -->
