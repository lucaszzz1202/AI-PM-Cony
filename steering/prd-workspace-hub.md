---
inclusion: manual
---

# Workspace Hub 集成

Cony 通过 workspace-hub MCP 与团队工作流集成，实现需求阶段的自动化流转。

**加载时机**：Workspace 模式启动时（选择模式后立即加载）、Workspace 模式恢复时。

## 角色定位

在 Workspace Hub 的工作流中，Cony 是需求阶段（cony stage）的执行者，角色为 PM（产品经理）。

工作流：`需求（Cony/PM）→ 设计（Archie/开发）→ 用例（Tessy/QA）`

## 双模式工作目录

Cony 支持两种工作模式，由用户在开始时主动选择：

### 模式一：Workspace 模式

当用户选择 Workspace 模式时，Cony 直接在 workspace 的 story 目录下工作，产物直接写入 workspace 规范路径。

工作目录：`workspace/workbench/[sprint]/[story]/`

目录映射关系：

| Cony 独立模式（.prd-refine/） | Workspace 模式（workspace/workbench/[sprint]/[story]/） |
|-------------------------------|------------------------------------------------------|
| `评审概览.md` | `state.md`（复用 workspace 状态机） |
| `origin/PRD原文.md` | `requirements/original.md` |
| `understanding.md` | `.process/understanding.md` |
| `questionnaires/01-愿景对齐.md` | `.process/questionnaires/01-愿景对齐.md` |
| `questionnaires/0N-xxx-澄清.md` | `.process/questionnaires/0N-xxx-澄清.md` |
| `output/完整PRD.md` | `requirements/final.md` |
| `output/变更记录.md` | `.process/变更记录.md`（注意：含义不同，见下方说明） |
| `output/质量报告.md` | `.process/质量报告.md` |
| `output/Confluence回填.html` | `.process/confluence-export.html` |
| `output/wireframes/` | `requirements/wireframes/` |
| `output/prototype/` | `.process/prototype/` |
| —（无对应） | `requirements/features.md`（新增：功能点拆解） |
| —（无对应） | `session.log.md`（新增：会话日志） |

> **变更记录含义差异**：独立模式的 `output/变更记录.md` 是"相对原始 PRD 的内容变更说明"（最终撰写时生成）。Workspace 模式的 `.process/变更记录.md` 是"工作流变更记录"（回退/跳过/修改决策时生成）。Workspace 模式不需要"内容变更说明"，因为 `session.log.md` 已完整记录决策演进。

### 模式二：独立模式

当用户选择独立模式时，保持原有 `.prd-refine/[需求名称]/` 目录结构。

### 模式判断逻辑

工作模式由用户主动选择，而非系统自动检测：

1. **新建项目时**：用户在欢迎语中选择模式（1️⃣ Workspace / 2️⃣ 独立）
   - 用户选 Workspace 模式 → **立即调用** workspace-hub 的 `setup_workspace` 初始化（幂等，已存在也调用）；调用失败或需要补充 GitLab 地址时阻塞流程，待用户补充后重试，成功后继续后续流程
   - 用户选独立模式 → 在 `.prd-refine/` 下创建项目目录
   - 用户选 D（从 workspace story 开始）→ 自动进入 Workspace 模式，同样调用 `setup_workspace`，规则同上
2. **恢复已有项目时**：模式由项目所在目录自动确定
   - 项目在 `workspace/workbench/` 下 → Workspace 模式
   - 项目在 `.prd-refine/` 下 → 独立模式
3. **用户直接给输入但没选模式**：
   - 检测到 `workspace/` 存在 → 提示用户选择（"检测到 workspace 环境，你想用 Workspace 模式还是独立模式？"）
   - `workspace/` 不存在 → 默认进入独立模式

**后续所有 steering 文件中提到的路径，Agent 需根据当前模式自动映射。**

## Workspace 模式详细规范

### Story 目录结构

在 Workspace 模式下，Cony 在 story 目录下产出的完整文件结构：

```
workspace/workbench/[sprint]/[story]/
├── state.md                        # 状态机（Cony 更新需求阶段相关状态）
├── session.log.md                  # 会话日志（Cony 追加需求阶段的会话记录）
├── requirements/                   # 需求阶段产物（Cony 的核心产出）
│   ├── original.md                 # 原始需求（只读，对应 origin/PRD原文.md）
│   ├── final.md                    # 完整需求 PRD（对应 output/完整PRD.md）
│   ├── features.md                 # 功能点拆解（新增产物，模板见 prd-workspace-features.md）
│   └── wireframes/                 # 原型截图（对应 output/wireframes/）
│       ├── {页面名}-{视图}.png
│       └── ...
└── .process/                       # 过程文件（不归档）
    ├── questionnaires/             # 问卷目录（按阶段编号，同独立模式）
    │   ├── 01-愿景对齐.md
    │   ├── 01-愿景对齐-澄清.md    # 矛盾澄清（如有）
    │   ├── 02-场景梳理.md
    │   ├── ...
    │   └── 04-验收对齐.md
    ├── 变更记录.md                 # 变更记录（如有）
    ├── 质量报告.md                 # 质量自检报告
    ├── understanding.md            # 理解积累文档
    ├── confluence-export.html      # Confluence 回填 HTML（如需）
    └── prototype/                  # 原型代码（如需）
        └── ...
```

### state.md 状态机集成

Cony 需要读取和更新 story 的 `state.md`。

**读取**：启动时读取 state.md 了解 story 当前状态，确认处于需求阶段。

**更新时机与内容**：

| 时机 | 更新内容 |
|------|----------|
| 开始 PRD 精炼 | 标记需求阶段为"进行中"，记录开始时间 |
| 每阶段完成 | 更新进度信息（如"愿景对齐完成"、"场景梳理完成"） |
| 问卷等待用户填写 | 标记中断点（同原评审概览.md 的中断点逻辑） |
| PRD 完成 + 质量自检通过 | 标记需求阶段为"已完成" |
| 流转到设计阶段 | 调用 advance_stage，状态自动更新 |
| 设计退回 | 标记需求阶段为"退回修正中" |

state.md 格式遵循 workspace 规范，Cony 只更新需求阶段相关的部分，不修改其他阶段的内容。

如果 state.md 不存在，Cony 创建初始版本：

```markdown
# {story-name} — 状态机

> 当前阶段: 需求
> 创建时间: {日期}
> 最后更新: {日期}

## 需求阶段（Cony/PM）

- 状态: 进行中
- 复杂度: {低/中/高}
- 开始时间: {日期}
- 完成时间: —
- 中断点: {精确描述}
- 下一步: {恢复后应执行的操作}

### 阶段进度

| 子阶段 | 状态 | 完成时间 | 备注 |
|--------|------|----------|------|
| 复杂度评估 | ⏳ | — | — |
| 愿景对齐 | ⬜ | — | — |
| 场景梳理 | ⬜ | — | — |
| 方案细化 | ⬜ | — | — |
| 验收对齐 | ⬜ | — | — |
| 最终撰写 | ⬜ | — | — |
| 原型设计 | ⬜ | — | — |
| 质量自检 | ⬜ | — | — |
| 输出回填 | ⬜ | — | — |

> **状态说明**：⬜ 未开始 / ⏳ 进行中 / ✅ 完成 / ⏭️ 跳过（输入已覆盖）/ ⏭️ 跳过（低复杂度）/ ⏭️ 跳过（用户主动跳过）

### 关键决策记录

- {日期}: {决策内容}

## 设计阶段（Archie/开发）

{由 Archie 填写}

## 用例阶段（Tessy/QA）

{由 Tessy 填写}
```

### session.log.md 会话日志

Cony 在每次会话中追加日志记录到 `session.log.md`。

**写入时机**：
- 每次会话开始时（记录恢复/启动信息）
- 每个阶段完成时（记录阶段产出摘要）
- 用户做出关键决策时（记录决策内容）
- 会话结束/中断时（记录中断点）

**格式**：

```markdown
# {story-name} — 会话日志

## [{日期} {时间}] 会话 #{N}

**操作者**: Cony (PM)
**起始状态**: {从哪个阶段/中断点开始}
**结束状态**: {到哪个阶段/中断点结束}

### 活动记录
- {时间} {活动描述}
- {时间} {活动描述}

### 关键决策
- {决策内容及原因}

---
```

Agent 使用 `fsAppend` 追加新的会话记录，不覆盖已有内容。

### .process/questionnaires/ 问卷目录

在 Workspace 模式下，问卷采用与独立模式相同的每阶段独立文件方式，放在 `.process/questionnaires/` 目录下。

**文件命名规范**（同独立模式）：
- `01-愿景对齐.md` — 阶段一问卷
- `01-愿景对齐-澄清.md` — 阶段一矛盾澄清（如有）
- `02-场景梳理.md` — 阶段二问卷
- `03-方案细化.md` — 阶段三问卷（低复杂度可能无此文件）
- `04-验收对齐.md` — 阶段四问卷

问卷格式严格遵循 `prd-question-format.md`（always 已加载），问卷文件末尾的提示语照常附加。用户在文件中填写 `[Answer]:` 后的内容，Agent 等待用户说"完成"后读取答案。

### 配置感知

Cony 在 Workspace 模式下应读取配置文件获取团队信息：

- `workspace/config/team.json` → 获取团队人员配置（PM、ARCHITECT、FE_DEV、BE_DEV、QA），用于流转通知
- `workspace/config/notification.json` → 获取企微通知配置
- `workspace/config/repos.json` → 获取仓库映射，在 features.md 中标注功能涉及的仓库

如果配置文件不存在，不影响核心流程，仅跳过相关功能（如通知）。

## Workspace 感知

### 启动时检测

Agent 在会话连续性检测时，按以下顺序扫描：

1. **扫描 `workspace/` 和 `.prd-refine/` 目录** → 检查是否有进行中的项目
2. **如果有进行中的项目** → 模式由项目所在目录自动确定，展示恢复摘要
3. **如果没有进行中的项目** → 显示欢迎语，由用户选择模式（1️⃣ Workspace / 2️⃣ 独立）

### 从 Story 启动 PRD 精炼

当用户选择 Workspace 模式或提到某个 story 时：

1. **初始化 workspace**：**必须**调用 workspace-hub 的 `setup_workspace` 工具（幂等，无论 `workspace/` 是否已存在都调用）：
   ```
   setup_workspace(
     workspace_path: <当前工作目录的绝对路径，通过 pwd 获取>
   )
   ```
   - 该工具会检查并创建缺失的目录和配置文件，已存在的内容不会被覆盖
   - **调用失败时必须阻塞流程**，不得跳过，向用户说明失败原因并等待解决
   - **如果工具提示需要 GitLab 仓库地址**（即当前目录不存在 `.git`），必须暂停并向用户询问：
     ```
     ⚠️ 初始化 workspace 需要 GitLab 仓库地址（当前目录尚未关联 Git 仓库）。
     请提供：
     - GitLab 仓库 HTTPS URL（如 https://gitlab-ex.mcd.com.cn/your-team/your-repo.git）
     - 分支名称（默认 main，如有不同请告知）
     ```
     获取到用户提供的信息后，重新调用：
     ```
     setup_workspace(
       workspace_path: <绝对路径>,
       repo_url: <用户提供的 GitLab HTTPS URL>,
       branch: <分支名，默认 main>
     )
     ```
   - 重新调用仍失败 → 告知用户具体错误，继续等待，直到初始化成功才能继续
2. **读取团队配置**：读取 `workspace/config/`（team.json、notification.json、repos.json），获取团队人员、通知渠道、仓库映射（如存在）
3. **确定 Sprint 和 Story 名称**（如用户未指定）：
   - 如果用户选了 D（从 workspace story 开始）→ 列出 `workspace/workbench/` 下已有的 sprint 和 story，让用户选择或创建新的
   - 如果用户选了 1A/1B/1C（Workspace 模式 + 非 story 输入）→ Agent 必须主动询问：
     ```
     📂 Workspace 模式需要指定 Sprint 和 Story 名称来创建工作目录。
     
     Sprint 名称（如 sprint-24、2024-Q1）：
     Story 名称（如 AR眼镜订单汇集、门店巡检优化）：
     
     或者你可以直接告诉我，比如"sprint-24 下的订单汇集"。
     ```
   - 获取到名称后才能创建 `workspace/workbench/[sprint]/[story]/` 目录并继续
4. 读取 story 目录下已有文件，提取需求描述、关联的 Confluence 链接等信息
5. 将信息作为输入，在该 story 目录下直接开始 PRD 精炼流程
6. 创建/更新 `state.md`，标记需求阶段为"进行中"
7. 创建 `requirements/original.md`（原始需求）
8. 创建 `.process/understanding.md`（理解积累）
9. 创建 `.process/questionnaires/` 目录（问卷）
10. 创建 `session.log.md`（会话日志）

### 恢复流程（Workspace 模式）

当检测到 story 目录下有进行中的 PRD 精炼时：

1. **读取 `state.md`** → 获取需求阶段的状态、中断点、阶段进度
2. **读取 `.process/understanding.md`** → 获取已积累的理解
3. **扫描 `.process/questionnaires/`** → 检查问卷和答案的完成情况
4. **扫描 `requirements/`** → 检查是否已有 final.md、features.md 等产出
5. **综合判断当前进度**（逻辑同独立模式）
6. **向用户展示恢复摘要**（格式同独立模式，但路径使用 workspace 路径）

## 注意事项

- workspace-hub MCP 在 Workspace 模式下是必要依赖，`setup_workspace` 调用失败时必须阻塞流程，不得降级为独立模式（用户已明确选择 Workspace 模式）
- Workspace 模式下，Cony 不创建 `.prd-refine/` 目录，所有文件都在 story 目录下
- Cony 只操作 `requirements/`、`.process/`、`state.md`、`session.log.md`，不触碰 `design/`、`tasks/`、`cases/` 等其他阶段的目录
- `.process/` 下的文件不会被归档，仅作为过程记录保留
- 阶段流转和知识沉淀：PRD 完成后，**Agent 必须读取 steering: `prd-workspace-knowledge.md`** 获取知识沉淀和阶段流转规范
- features.md 生成：最终撰写时，**Agent 必须读取 steering: `prd-workspace-features.md`** 获取 features.md 模板和生成规范
