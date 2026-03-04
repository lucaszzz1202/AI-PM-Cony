---
inclusion: always
---

# PRD 精炼工作流 — 核心流程

## ⚡ 会话连续性（MANDATORY FIRST STEP）

**每次对话开始时，Agent 必须在执行任何其他操作之前，先执行以下检测流程：**

### 检测流程

1. **扫描 `workspace/` 目录**：检查是否存在 workspace hub 环境
2. **扫描 `.prd-refine/` 目录**：检查是否存在独立模式项目
3. **检查是否有进行中的项目**：
   - `workspace/workbench/` 下有进行中的需求阶段 story → 执行 Workspace 恢复流程
   - `.prd-refine/` 下有进行中的项目 → 执行独立模式恢复流程
   - 两边都有进行中的项目 → 列出所有项目，让用户选择要继续的
4. **如果没有进行中的项目** → 正常流程（欢迎语 → 用户选择模式和输入方式）

**关键：新建项目时，工作模式由用户在欢迎语中主动选择（1️⃣ Workspace / 2️⃣ 独立）。恢复已有项目时，模式由项目所在目录自动确定。**

### Workspace 模式恢复流程

1. **Agent 必须读取 steering: `prd-workspace-hub.md`** 获取 Workspace 模式的完整规范
2. 读取 `state.md` → 获取需求阶段的状态、复杂度、中断点、阶段进度
3. 读取 `.process/understanding.md` → 获取已积累的完整理解
4. 扫描 `.process/questionnaires/` → 检查问卷和答案的完成情况
5. 扫描 `requirements/` → 检查已有产出
6. 综合判断进度（state.md 为主，实际文件交叉验证，不一致时以实际文件为准）
7. Smart Context Loading：按恢复阶段加载最小必要产物
8. 展示恢复摘要，等待用户确认后继续
9. 用户确认后，根据中断点加载对应 steering 文件继续执行（参考下方 Steering 加载指引表）

### 独立模式恢复流程

1. 读取 `评审概览.md` → 获取复杂度、当前阶段、中断点
2. 读取 `understanding.md` → 获取已积累的完整理解
3. 扫描 `questionnaires/` 和 `output/` → 检查完成情况
4. 综合判断进度（评审概览为主，实际文件交叉验证）
5. Smart Context Loading：按恢复阶段加载最小必要产物
6. 展示恢复摘要，等待用户确认后继续
7. 用户确认后，根据中断点加载对应 steering 文件继续执行（参考下方 Steering 加载指引表）

### 多项目处理

多个进行中的项目时，列出所有项目及状态，让用户选择。

### 错误恢复

- 状态追踪文件缺失 → 从实际文件反推进度，重建
- understanding.md 缺失 → 从问卷答案重建，标注"从问答记录恢复"
- 原始需求文件缺失 → 告知用户，询问是否重新提供
- 问卷文件损坏 → 告知用户，提供重新生成选项


## 总体流程与 Steering 加载指引

```
用户激活 Cony
        │
        ▼
┌─ 欢迎引导 ──────────────────────────┐
│  ⚡ 读取 steering: prd-welcome.md    │
└───────────────────────────────────┘
        │
        ▼
用户提供输入 + 选择工作模式
        │
        ▼
┌─ Workspace 模式初始化 ──────────────┐
│  （仅 Workspace 模式，阻塞性步骤）   │
│  ⚡ 读取 steering:                   │
│     prd-workspace-hub.md            │
│  ⚡ 调用 workspace-hub.setup_workspace│
│     (workspace_path=<pwd绝对路径>)   │
│  ⚠️ 失败则阻塞，等待用户解决         │
│  ⚠️ 需要 repo_url 则向用户询问       │
└───────────────────────────────────┘
        │
        ▼
┌─ 输入获取 ──────────────────────────┐
│  ⚡ 读取 steering: prd-file-specs.md │
│  ⚡ 读取 steering:                   │
│     prd-understanding-template.md   │
│     （创建 understanding.md v0.0）   │
│  ⚡ Confluence 输入：                │
│     读取 steering: prd-confluence.md│
└───────────────────────────────────┘
        │
        ▼
┌─ 覆盖度分析 + 自适应规划 ───────────┐
│  ⚡ 读取 steering:                   │
│     prd-phase0-assessment.md        │
│  ⚡ Workspace 模式：                 │
│     读取 landscape.md               │
│     （了解系统现状作为推导背景）      │
└───────────────────────────────────┘
        │
        ▼
┌─ 阶段一~四（EXECUTE / SKIP）────────┐
│  ⚡ 读取 steering: prd-phases.md    │
│  ⚡ 阶段二/三问卷生成前：            │
│     读取 steering:                   │
│     prd-param-rules.md              │
│     （参数规则参考库，交叉验证）      │
│  ⚡ 更新 understanding.md 前：       │
│     读取 steering:                   │
│     prd-understanding-template.md   │
│  （prd-question-format.md 已常驻）   │
│  ⚡ Workspace 模式（按需）：          │
│     读取 modules/{模块名}.md        │
│     （涉及已有模块时深入了解）        │
└───────────────────────────────────┘
        │
        ▼
┌─ 撰写就绪检查（门禁）───────────────┐
│  扫描 understanding.md：             │
│  1. "假设与推导"中是否有未确认的      │
│     关键假设（⚠️ 标记项）            │
│  2. "开放问题"中是否有 🔴 项         │
│  3. "待深入"中是否有未解决项          │
│                                      │
│  如果存在上述任一情况：               │
│  → 生成补充确认问卷                   │
│    （Workspace 模式:                 │
│     .process/questionnaires/         │
│     05-撰写前确认.md                 │
│     独立模式:                        │
│     questionnaires/05-撰写前确认.md）│
│  → 让用户确认关键假设 /              │
│    解决开放问题                       │
│  → 用户确认后更新 understanding.md   │
│    才进入最终撰写                     │
│                                      │
│  如果全部清零：                       │
│  → 直接进入最终撰写                   │
└───────────────────────────────────┘
        │
        ▼
┌─ 最终撰写 ──────────────────────────┐
│  ⚡ 读取 steering: prd-template.md   │
│  ⚡ Workspace 模式：                 │
│     读取 steering:                   │
│     prd-workspace-features.md       │
│     （features.md 模板）             │
└───────────────────────────────────┘
        │
        ▼
┌─ 原型设计确认 ──────────────────────┐
│  ⚡ 读取 steering:                   │
│     prd-wireframes.md               │
└───────────────────────────────────┘
        │
        ▼
┌─ 质量自检 ──────────────────────────┐
│  ⚡ 读取 steering:                   │
│     prd-quality-check.md            │
└───────────────────────────────────┘
        │
        ▼
┌─ 输出回填 ──────────────────────────┐
│  ⚡ 询问用户输出方式：               │
│     - Confluence 回填（原页面/新页面）│
│       → 读取 steering:              │
│         prd-confluence.md           │
│     - 仅保留本地 → 跳过此步骤       │
│  ⚡ 非 Confluence 输入时也可选择     │
│     创建 Confluence 新页面           │
└───────────────────────────────────┘
        │
        ▼
┌─ 知识沉淀 + 阶段流转 ───────────────┐
│  （仅 Workspace 模式）               │
│  ⚡ 读取 steering:                   │
│     prd-workspace-knowledge.md      │
└───────────────────────────────────┘
```

> ⚡ 标记表示 **Agent 必须在该步骤主动读取对应 steering 文件**，不能依赖 inclusion 头部声明自动加载。每个 ⚡ 都是强制动作。

### 按需加载的 Steering 文件索引

| Steering 文件 | 加载时机 | inclusion |
|---------------|----------|-----------|
| `prd-workflow-core.md` | 每次对话自动加载 | always |
| `prd-question-format.md` | 每次对话自动加载 | always |
| `prd-welcome.md` | 首次激活，显示欢迎语 | manual |
| `prd-file-specs.md` | 输入获取阶段，创建文件结构 | manual |
| `prd-confluence.md` | 用户提供 Confluence 链接 / 选择回填 | manual |
| `prd-phase0-assessment.md` | 覆盖度分析 + 自适应规划 | manual |
| `prd-phases.md` | 阶段一~四执行时 / 变更管理回退时 | manual |
| `prd-understanding-template.md` | 输入获取阶段创建 understanding.md / 每次更新 understanding.md 前 | fileMatch + 显式 |
| `prd-template.md` | 最终撰写 | manual |
| `prd-wireframes.md` | 原型设计环节 | manual |
| `prd-quality-check.md` | 质量自检 | manual |
| `prd-workspace-hub.md` | Workspace 模式：选择模式后立即调用 setup_workspace / 恢复时 | manual |
| `prd-workspace-features.md` | Workspace 模式：最终撰写阶段生成 features.md | manual |
| `prd-workspace-knowledge.md` | Workspace 模式：知识沉淀 + 阶段流转 | manual |
| `prd-param-rules.md` | 阶段二/三问卷生成前、参数补充检测时、质量自检 §1.5 时 | manual |
| `prd-change-management.md` | 用户请求变更时（回退/跳过/修改决策）/ 设计阶段退回时 | manual |

**恢复时的 Steering 加载映射**（根据中断点判断）：

| 恢复到的阶段 | 必须加载的 Steering 文件 |
|-------------|------------------------|
| 覆盖度评估 | `prd-phase0-assessment.md` |
| 阶段一~四（问卷等待填写） | `prd-phases.md` + `prd-understanding-template.md`（阶段二/三额外加载 `prd-param-rules.md`） |
| 阶段一~四（答案已填，待更新理解） | `prd-phases.md` + `prd-understanding-template.md`（阶段二/三额外加载 `prd-param-rules.md`） |
| 撰写就绪检查 | `prd-understanding-template.md` |
| 最终撰写 | `prd-template.md`（Workspace 模式额外加载 `prd-workspace-features.md`） |
| 原型设计 | `prd-wireframes.md` |
| 质量自检 | `prd-quality-check.md` |
| 输出回填 | `prd-confluence.md`（如需 Confluence 回填） |
| 知识沉淀 / 阶段流转 | `prd-workspace-knowledge.md` |

## 核心理念

- 问卷是挖掘和澄清需求的过程工具，PRD 模板是最终交付物的格式规范
- 两者之间是"理解 → 撰写"的关系，不是字段映射
- Agent 通过问卷充分理解用户意图后，以专业产品经理视角撰写 PRD
- 问卷没直接问到但可以合理推导的内容，Agent 应主动补充

## 理解积累文档（understanding.md）

每轮问卷回答后，Agent 必须更新 `understanding.md`。

- **Workspace 模式**：位于 `workspace/workbench/[sprint]/[story]/.process/understanding.md`
- **独立模式**：位于 `.prd-refine/[需求名称]/understanding.md`

作用：上下文传递、用户确认锚点、最终撰写依据、理解演进追溯。

模板和更新规范见 steering 文件 `prd-understanding-template.md`（读取 understanding.md 时自动加载；如未自动加载，**Agent 必须读取 steering: `prd-understanding-template.md`**）。

核心原则：**先推导，再验证，最后才问新问题。**

**关键约束：更新 understanding.md 时必须增量追加，严禁覆盖或删除已有的"已明确"内容。禁止使用 fsWrite 更新 understanding.md（首次创建除外），必须使用 strReplace / fsAppend / editCode 进行精确修改。**

每个阶段结束后，告知用户："已更新 understanding.md（v{N}.0），请确认理解是否准确，确认后进入下一阶段。"

## 每阶段执行规范

### 阶段 SKIP 时

1. ✅ 维度内容已在覆盖度评估阶段写入 understanding.md 的"已明确"
2. 更新状态追踪：标记为 `⏭️ 跳过（输入已覆盖）`
3. Workspace 模式：追加 session.log.md 跳过记录
4. 聊天中简要告知用户，直接进入下一个 EXECUTE 阶段

### 阶段 EXECUTE 时

1. **Agent 必须读取 steering: `prd-phases.md`** 获取对应阶段的维度、挖掘方向和功能完备性检查清单
2. 读取理解文档 + 原始需求（每个阶段都要参考原始输入）
3. 只为 🔶 和 ❌ 维度生成问卷，✅ 维度不出现，同时纳入功能完备性检查清单识别的缺失项
   - Workspace 模式：生成 `.process/questionnaires/0N-xxx.md`
   - 独立模式：生成 `questionnaires/0N-xxx.md`
4. 更新状态追踪中断点，等待用户填写
5. 读取答案 → 矛盾检测 → 如有矛盾生成澄清问卷（最多 2 轮）
6. **参数完备性检测**：对照功能完备性检查清单（见 `prd-phases.md`），检查用户回答是否产生了新的未定义参数 → 如有缺失生成参数补充问卷（最多 1 轮，与矛盾澄清独立计数）
7. 增量更新 understanding.md（**Agent 必须读取 steering: `prd-understanding-template.md`** 获取更新规范和来源标注格式）
8. 更新状态追踪，Workspace 模式追加 session.log.md
9. 告知用户确认理解，确认后进入下一阶段
