# MachinistDev

CreateAI(Machinist)平台开发工作流
基于Toolkit提供的TS视图的开发模板与参考资料。

## 相关链接

- CreateAI平台(Machinist) Steam链接 https://store.steampowered.com/app/1265510/Machinist/
- Toolkit仓库 https://github.com/2302680972/MechToolkit
- Skills仓库 https://github.com/2302680972/MachinistDev
- Toolkit使用教程 https://sx16dhdgjdw.feishu.cn/wiki/AdLGwxUuViJ3zpkt9CPcWhPznGf

## 目录结构

```
.
├── skills/
│   ├── .curated/
│   │   └── machinist/     # 唯一 Skill
│   │       ├── SKILL.md   # Skill 定义文档
│   │       ├── assets/    # 代码模板
│   │       └── references/# 参考资料
│   └── .experimental/
│       └── .gitkeep       # 预留实验通道目录
├── LICENSE
└── README.md
```

## 快速开始

### 安装

这是一个公开的 Skill 仓库，核心 Skill 位于：

- `skills/.curated/machinist`

如果你的工具支持从 GitHub 仓库安装 Skills，可直接指向本仓库，再选择上述目录作为 Skill 根目录。

### AGENTS模板

`agents_claude.template.md`是一个AGENTS.md的模板,提供了平台的重要注意事项

### 代码模板

位于 `skills/.curated/machinist/assets/` 目录，按功能分类：

| 分类 | 说明 |
|------|------|
| `mech/` | 机械注册、对象池申请与回收 |
| `physics/` | 物理属性、碰撞层初始化 |
| `unit/` | 单位数据、属性计算、目标选择 |
| `danmu/` | 礼物处理、排行榜、结算逻辑 |
| `canvas/` | Canvas 引用、动画、缓动效果 |

使用方式：打开对应模板，将代码片段迁移到目标工程。

### 参考资料

位于 `skills/.curated/machinist/references/` 目录：

- `concepts/` - 核心概念说明（Canvas 组件、事件机制、存档等）
- `tips/` - 实践技巧（动画机制、性能优化、调色板等）

### 导入路径

Marketspace/安装脚本使用的 Skill 路径：

- `skills/.curated/machinist`

### 元数据

面向 Skill 列表展示的元数据位于：

- `skills/.curated/machinist/agents/openai.yaml`

## 标准来源

- Agent Skills 规范：<https://agentskills.io/specification>
- Codex Skills 文档：<https://developers.openai.com/codex/skills>
- OpenAI skills 仓库目录约定（`.curated` / `.experimental`）：<https://github.com/openai/skills>

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件
