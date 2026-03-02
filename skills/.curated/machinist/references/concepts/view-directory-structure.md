# 视图的目录结构

## 本篇范围

- 视图目录位置与结构
- TS 视图与 HTML 视图的文件分布

## 不包含

- TS 视图语法与写回规则（见"TS 视图编写规范"）
- UI/Canvas 的运行时使用方式（见"UI 组件管理指引"）

## 总体位置

视图目录固定在"工程根目录"的 `temp/toolkit/views/` 下。

- 工程根目录：通常是 `setting.json` 所在的那一层目录。
- 视图是自动生成/刷新出来的：首次打开工程、或工具刷新视图之前，`temp/toolkit/views/` 可能不存在。

## TS 视图（脚本相关）

位于 `temp/toolkit/views/device/` 目录：

- `temp/toolkit/views/device/sdk.base.d.ts`：TS 视图基础库声明（通常只读，用于类型提示）。
- `temp/toolkit/views/device/sdk.ext.d.ts`：TS 视图扩展库声明（通常只读）。
- `temp/toolkit/views/device/custom-logic.d.ts`：全工程脚本索引（用于跳转/引用关系/类型提示，通常只读）。
- `temp/toolkit/views/device/CodeRepo.ts`：代码仓库脚本视图（可编辑；方法体会以 `//|` 形式保留原版 BeScript 命令行）。
- `temp/toolkit/views/device/tsconfig.json`：TS 配置文件。
- `temp/toolkit/views/device/<载体目录>/<零件ID>.ts`：具体零件 TS 视图文件（通常主要编辑这些文件）。

### 零件视图文件命名

零件视图路径形如：

- `temp/toolkit/views/device/<载体目录>/<零件ID>.ts`

示例：

- `temp/toolkit/views/device/abc.map_/123.ts`：表示 `abc.map` 里 ID 为 `123` 的零件的完整 TS 视图。
- `temp/toolkit/views/device/某个机械.mech_/0.ts`：表示某个机械里 ID 为 `0` 的零件的完整 TS 视图。

说明：

- `device/` 下通常会按"载体"分组（地图/机械等），载体目录名常见以 `.map_` / `.mech_` 结尾。
- 每个载体目录下是一批 `<零件ID>.ts` 文件。

## HTML 视图（Canvas 相关）

位于 `temp/toolkit/views/canvas/`：

- `temp/toolkit/views/canvas/canvas-view.css`：Canvas 基础样式文件。
- `temp/toolkit/views/canvas/canvas-palette.js`：Canvas 调色板索引。
- `temp/toolkit/views/canvas/CanvasLayoutEngine.DO_NOT_READ_OR_EDIT.generated.js`：布局渲染引擎脚本。
- `temp/toolkit/views/canvas/canvas-guid-map.js`：独立模式 GUID 图片映射索引。
- `temp/toolkit/views/canvas/**/*.html`：具体 Canvas 的 HTML 视图文件。

说明：

- `canvas/` 目录不提供 `tsconfig*.json` 或 `*.d.ts`；Canvas 预览侧不走 tsc 类型检查。

### Canvas 视图路径的对应关系

- 对于独立 `.canvas` 文件：在 `canvas/` 下保留相对路径，并将扩展名改为 `.html`。
  - 示例：工程里有 `UI_主UI.canvas` → 视图为 `temp/toolkit/views/canvas/UI_主UI.html`
  - 示例：工程里有 `layouts/A2.canvas` → 视图为 `temp/toolkit/views/canvas/layouts/A2.html`
- 对于脚本/变量中的"内联 Canvas"：会在 `canvas/` 下展开为 `InlineCanvas*.html` 一类的文件，便于单独编辑与写回。

## 相关文档

- `tsview-writing-spec.md`
- `canvas-component-guide.md`
