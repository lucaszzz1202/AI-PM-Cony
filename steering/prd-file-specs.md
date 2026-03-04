---
inclusion: manual
---

# 文件规范与目录结构

本文件包含 Cony 工作流中所有文件的目录结构、命名规范、文件说明、模板和生命周期。

**加载时机**：输入获取阶段创建文件结构时加载。

## 路径映射表

Agent 根据当前工作模式自动映射文件路径：

| 通用名称 | Workspace 模式路径 | 独立模式路径 |
|----------|-------------------|-------------|
| 原始需求 | `requirements/original.md` | `origin/PRD原文.md` |
| 理解文档 | `.process/understanding.md` | `understanding.md` |
| 问卷文件 | `.process/questionnaires/0N-xxx.md`（独立文件） | `questionnaires/0N-xxx.md`（独立文件） |
| 状态追踪 | `state.md` | `评审概览.md` |
| 最终 PRD | `requirements/final.md` | `output/完整PRD.md` |
| 功能点拆解 | `requirements/features.md` | —（独立模式不生成） |
| 会话日志 | `session.log.md` | —（独立模式不生成） |

## 目录结构模板

### Workspace 模式目录结构

在 workspace 中，Cony 直接在 story 目录下工作。Sprint 和 story 名称由用户指定或从 workspace 已有结构中获取。

```
workspace/workbench/[sprint]/[story]/
├── state.md                          # 状态机（Cony 更新需求阶段部分）
├── session.log.md                    # 会话日志（Cony 追加需求阶段记录）
├── requirements/                     # 需求阶段产物
│   ├── original.md                   # 原始需求（只读）
│   ├── final.md                      # 完整需求 PRD
│   ├── features.md                   # 功能点拆解
│   └── wireframes/                   # 原型截图
│       ├── {页面名}-{视图}.png
│       └── ...
└── .process/                         # 过程文件（不归档）
    ├── questionnaires/               # 问卷目录（按阶段编号，同独立模式）
    │   ├── 01-愿景对齐.md
    │   ├── 01-愿景对齐-澄清.md      # 矛盾澄清（如有）
    │   ├── 02-场景梳理.md
    │   ├── ...
    │   └── 04-验收对齐.md
    ├── 变更记录.md                   # 变更记录（如有）
    ├── 质量报告.md                   # 质量自检报告
    ├── understanding.md              # 理解积累文档
    ├── confluence-export.html        # Confluence 回填 HTML（如需）
    └── prototype/                    # 原型代码（如需）
        └── ...
```

**命名规范**：
- Sprint 名称：由用户指定，如 `sprint-24`、`2024-Q1` 等
- Story 名称：由用户指定或从需求中提取简短标识。如果来自 Confluence，使用 `页面标题-pageId` 格式

### 独立模式目录结构

根目录为 `.prd-refine/[需求名称]/`，支持多需求并行互不干扰。

**需求名称**：从用户输入中提取简短标识（如产品名、功能名）。如果来自 Confluence，使用 `页面标题-pageId` 格式（如 `AR眼镜订单汇集-12345`）。

```
.prd-refine/
└── [需求名称]/
    ├── 评审概览.md                    # 进度追踪（当前阶段、复杂度、各阶段状态）
    │
    ├── origin/                        # 原始输入（只读，不修改）
    │   └── PRD原文.md                 # 原始 PRD / idea 文本 / Confluence 内容副本
    │
    ├── understanding.md               # 理解积累文档（跨阶段持续增量更新）
    │
    ├── questionnaires/                # 问卷目录（按阶段编号）
    │   ├── 01-愿景对齐.md             # 阶段一问卷
    │   ├── 01-愿景对齐-澄清.md        # 阶段一矛盾澄清（如有）
    │   ├── 02-场景梳理.md             # 阶段二问卷
    │   ├── 02-场景梳理-澄清.md        # 阶段二矛盾澄清（如有）
    │   ├── 03-方案细化.md             # 阶段三问卷（低复杂度可能无此文件）
    │   ├── 03-方案细化-澄清.md        # 阶段三矛盾澄清（如有）
    │   ├── 04-验收对齐.md             # 阶段四问卷
    │   └── 04-验收对齐-澄清.md        # 阶段四矛盾澄清（如有）
    │
    └── output/                        # 产出目录
        ├── 完整PRD.md                 # 最终 PRD（基于原始内容 + 问卷补充合并撰写）
        ├── 变更记录.md                # 相对原始 PRD 的变更说明（新增/修改/删除了什么）
        ├── 质量报告.md                # 质量自检报告
        ├── Confluence回填.html        # HTML 格式（回填 Confluence 时生成）
        ├── wireframes/                # 原型截图（原型设计环节生成）
        │   ├── {页面名}-{视图}.png    # 如 order-list-desktop.png
        │   └── ...
        └── prototype/                 # 原型代码（原型设计环节生成）
            └── ...
```


## 各文件说明

### Workspace 模式文件说明

| 文件 | 用途 | 写入时机 | 可修改性 |
|------|------|----------|----------|
| `state.md` | 状态机，记录需求阶段状态、中断点、阶段进度 | 流程启动时创建/更新，每阶段结束更新 | Agent 更新需求阶段部分 |
| `session.log.md` | 会话日志，记录每次会话的活动和决策 | 每次会话追加 | Agent 追加 |
| `requirements/original.md` | 原始输入的完整副本，作为最终撰写的参考基线 | 输入获取时创建 | **只读，不可修改** |
| `requirements/final.md` | 最终交付的 PRD 文档 | 最终撰写阶段创建 | Agent 撰写 / 质量自检后修补 |
| `requirements/features.md` | 功能点拆解，从 final.md §7 提取 | 最终撰写阶段同步创建 | Agent 撰写 |
| `requirements/wireframes/` | 原型截图 | 原型设计环节生成 | Agent 生成 |
| `.process/questionnaires/0N-*.md` | 各阶段问卷（同独立模式命名规范） | 对应阶段开始时创建 | 用户填写答案 |
| `.process/questionnaires/0N-*-澄清.md` | 矛盾澄清问卷 | 检测到矛盾时创建 | 用户填写答案 |
| `.process/变更记录.md` | 流程变更记录（回退、跳过、修改决策等工作流变更） | 变更发生时创建 | Agent 撰写 |
| `.process/质量报告.md` | 质量自检报告 | 质量自检阶段创建 | Agent 撰写 |
| `.process/understanding.md` | 理解积累，跨阶段传递上下文 | 输入获取时创建，每阶段增量更新 | Agent 增量更新 / 用户可编辑 |
| `.process/confluence-export.html` | Confluence 格式的 PRD | 用户选择回填时生成 | Agent 生成 |
| `.process/prototype/` | 原型代码 | 原型设计环节生成 | Agent 生成 |

> **关于"变更记录"的区分**：
> - **Workspace 模式**的 `.process/变更记录.md` 记录的是工作流变更（回退阶段、跳过阶段、修改已确认决策等流程操作），由 `prd-change-management.md` 规范。Workspace 模式不单独生成"相对原始 PRD 的内容变更说明"，因为 `session.log.md` 已完整记录了所有决策演进过程，且 `requirements/final.md` 本身就是最终交付物。
> - **独立模式**的 `output/变更记录.md` 记录的是最终 PRD 相对原始输入的内容变更（新增了什么、修改了什么、删除了什么），在最终撰写阶段同步生成，帮助用户快速了解 Cony 做了哪些补充和调整。独立模式没有 session.log.md，所以需要这个文件来体现变更全貌。

### 独立模式文件说明

| 文件 | 用途 | 写入时机 | 可修改性 |
|------|------|----------|----------|
| `评审概览.md` | 进度追踪，记录当前阶段、复杂度判断、各阶段完成状态 | 流程启动时创建，每阶段结束更新 | Agent 更新 |
| `origin/PRD原文.md` | 原始输入的完整副本，作为最终撰写的参考基线 | 输入获取时创建 | **只读，不可修改** |
| `understanding.md` | 理解积累，跨阶段传递上下文 | 输入获取时创建，每阶段增量更新 | Agent 增量更新 / 用户可编辑 |
| `questionnaires/0N-*.md` | 各阶段问卷 | 对应阶段开始时创建 | 用户填写答案 |
| `questionnaires/0N-*-澄清.md` | 矛盾澄清问卷 | 检测到矛盾时创建 | 用户填写答案 |
| `output/完整PRD.md` | 最终交付的 PRD 文档 | 最终撰写阶段创建 | Agent 撰写 / 质量自检后修补 |
| `output/变更记录.md` | 相对原始 PRD 的内容变更说明（新增/修改/删除了什么） | 最终撰写阶段同步创建 | Agent 撰写 |
| `output/质量报告.md` | 质量自检结果 | 质量自检阶段创建 | Agent 撰写 |
| `output/Confluence回填.html` | Confluence 格式的 PRD | 用户选择回填时生成 | Agent 生成 |

## 评审概览模板（仅独立模式）

Workspace 模式使用 `state.md`（模板见 steering: `prd-workspace-hub.md`，Workspace 模式输入获取时已加载；如未加载，**Agent 必须读取 steering: `prd-workspace-hub.md`** 获取 state.md 模板），不使用评审概览.md。

```markdown
# {需求名称} — 评审概览

> 创建时间: {日期}
> 复杂度: {低/中/高}
> 当前阶段: {阶段名称}
> 来源: {Confluence 链接 / 用户输入 / 文件}
> Story ID: {workspace story 标识，无则留空}
> 工作流状态: {需求阶段 / 已流转设计 / 设计退回，无 workspace 则留空}

## 中断点

> 最后活跃: {日期时间}
> 中断位置: {精确描述，如"阶段二问卷已生成，等待用户填写"}
> 下一步: {恢复后应执行的具体操作，如"读取 questionnaires/02-场景梳理.md 的用户答案"}

## 阶段进度

| 阶段 | 状态 | 完成时间 | 备注 |
|------|------|----------|------|
| 复杂度评估 | ✅ 完成 | {日期} | {复杂度}，覆盖度 {N}/15 |
| 阶段一：愿景对齐 | ⏳ 进行中 | — | — |
| 阶段二：场景梳理 | ⬜ 未开始 | — | — |
| 阶段三：方案细化 | ⬜ 未开始 | — | {低复杂度或全覆盖标注"计划跳过"} |
| 阶段四：验收对齐 | ⬜ 未开始 | — | — |
| 最终撰写 | ⬜ 未开始 | — | — |
| 原型设计 | ⬜ 未开始 | — | — |
| 质量自检 | ⬜ 未开始 | — | — |
| 输出回填 | ⬜ 未开始 | — | — |
| 阶段流转 | ⬜ 未开始 | — | {无 workspace 则标注"不适用"} |

## 关键决策记录

- {日期}: {决策内容}
```

## 中断点更新规范

**Agent 必须在以下时机更新状态追踪文件（Workspace 模式: state.md / 独立模式: 评审概览.md）的中断点信息：**

1. **生成问卷后** → `中断位置: 阶段{N}问卷已生成，等待用户填写` / `下一步: 用户填写后读取答案`
2. **读取答案后** → `中断位置: 阶段{N}答案已读取，正在更新理解文档` / `下一步: 更新 understanding.md 并请用户确认`
3. **用户确认理解后** → `中断位置: 阶段{N}已完成，准备进入阶段{N+1}` / `下一步: 读取 steering: prd-phases.md 并生成问卷`
4. **最终撰写中** → `中断位置: 正在撰写最终 PRD` / `下一步: 继续撰写或进入原型设计确认`
5. **质量自检中** → `中断位置: 质量自检进行中，{N}个🔴项待修复` / `下一步: 解决🔴项后重新检查`
6. **每次对话即将结束时**（Agent 感知到回复较长、即将触发上下文限制时）→ 主动更新中断点

`最后更新` 字段每次更新中断点时同步更新为当前日期时间。

**Workspace 模式额外要求**：每次更新中断点时，同步追加 session.log.md 活动记录。

## 文件生命周期

### Workspace 模式

| 阶段 | 创建的文件 | 更新的文件 |
|------|-----------|-----------|
| 输入获取 | `requirements/original.md`, `state.md`, `session.log.md`, `.process/questionnaires/`（目录）, `.process/understanding.md` | — |
| 复杂度评估 + 阶段一 | `.process/questionnaires/01-愿景对齐.md`（如 EXECUTE） | `.process/understanding.md`, `state.md`, `session.log.md` |
| 阶段一完成 | — | `.process/understanding.md`, `state.md`, `session.log.md` |
| 阶段二 | `.process/questionnaires/02-场景梳理.md`（如 EXECUTE） | `.process/understanding.md`, `state.md`, `session.log.md` |
| 阶段三 | `.process/questionnaires/03-方案细化.md`（如 EXECUTE） | `.process/understanding.md`, `state.md`, `session.log.md` |
| 阶段四 | `.process/questionnaires/04-验收对齐.md`（如 EXECUTE） | `.process/understanding.md`, `state.md`, `session.log.md` |
| 最终撰写 | `requirements/final.md`, `requirements/features.md` | `state.md`, `session.log.md` |
| 原型设计 | `requirements/wireframes/*.png`, `.process/prototype/*` | `requirements/final.md`（§9）, `state.md`, `session.log.md` |
| 质量自检 | `.process/质量报告.md` | `requirements/final.md`（自动补全）, `requirements/features.md`（同步更新）, `state.md`, `session.log.md` |
| 输出回填 | `.process/confluence-export.html`（如需） | `state.md`, `session.log.md` |
| 知识沉淀 | `workspace/knowledge/product/landscape.md`（如首次）, `workspace/knowledge/product/glossary.md`（如首次）, `workspace/knowledge/product/modules/*.md`（如需） | `workspace/knowledge/product/landscape.md`（更新模块清单+变更摘要）, `workspace/knowledge/product/glossary.md`（追加术语）, `state.md`, `session.log.md` |
| 阶段流转 | — | `state.md`（需求阶段标记完成）, `session.log.md` |

### 独立模式

| 阶段 | 创建的文件 | 更新的文件 |
|------|-----------|-----------|
| 输入获取 | `origin/PRD原文.md`, `评审概览.md`, `understanding.md` | — |
| 复杂度评估 + 阶段一 | `questionnaires/01-愿景对齐.md`（如 EXECUTE） | `understanding.md`, `评审概览.md` |
| 阶段一完成 | — | `understanding.md`, `评审概览.md` |
| 阶段二 | `questionnaires/02-场景梳理.md` | `understanding.md`, `评审概览.md` |
| 阶段三 | `questionnaires/03-方案细化.md`（如不跳过） | `understanding.md`, `评审概览.md` |
| 阶段四 | `questionnaires/04-验收对齐.md` | `understanding.md`, `评审概览.md` |
| 最终撰写 | `output/完整PRD.md`, `output/变更记录.md` | `评审概览.md` |
| 原型设计 | `output/wireframes/*.png`, `output/prototype/*` | `output/完整PRD.md`（§9）, `评审概览.md` |
| 质量自检 | `output/质量报告.md` | `output/完整PRD.md`（自动补全）, `评审概览.md` |
| 输出回填 | `output/Confluence回填.html`（如需） | `评审概览.md` |

## 注意事项

**通用规则**：
- 原始需求文件（`requirements/original.md` 或 `origin/PRD原文.md`）是只读的，任何时候都不要修改
- 不要创建规范之外的文件（如 summary.md、notes.md 等）
- 问卷内容一旦用户填写完成，不要覆盖（用户的回答在里面）
- 如果需要重新生成某阶段问卷（如回退），先备份旧内容

**Workspace 模式额外规则**：
- 所有文件都在 `workspace/workbench/[sprint]/[story]/` 目录下
- Cony 只操作 `requirements/`、`.process/`、`state.md`、`session.log.md`，不触碰 `design/`、`tasks/`、`cases/` 等其他阶段的目录
- `.process/` 下的文件不会被归档到 `workspace/archive/`
- 不要创建 `.prd-refine/` 目录
- 多 story 并行时，每个 story 有独立的目录，互不干扰

**独立模式额外规则**：
- 所有文件都在 `.prd-refine/[需求名称]/` 目录下，不要散落在项目根目录
- 多需求并行时，每个需求有独立的目录，互不干扰
