---
inclusion: manual
---

# PRD 标准模板

Agent 基于全部阶段积累的理解，按此模板撰写最终 PRD。

- **Workspace 模式**：写入 `requirements/final.md`，同步生成 `requirements/features.md`
- **独立模式**：写入 `output/完整PRD.md`，同步生成 `output/变更记录.md`（相对原始输入的内容变更说明）

## 撰写原则

- 以专业产品经理视角填充内容
- 问卷中直接获得的信息如实体现
- 可以合理推导的内容主动补充
- 无法确定的内容标注 `[待确认]`，**特别是功能的关键业务参数——不能一笔带过，必须显式标注缺什么**
- 精简版重点填充 §2、§4、§7、§8.1，其余简要或标注"不适用"
- 标准版完整填充 §1-§8，§9-§12 按需
- 详尽版所有章节完整填充

**版本选择与复杂度映射**：Agent 根据 Phase 0 判定的复杂度自动选择撰写深度：
- 低复杂度 → 精简版
- 中复杂度 → 标准版
- 高复杂度 → 详尽版

**重要例外**：§7（功能需求）不受撰写深度限制。不论精简版、标准版还是详尽版，每个 FR 的 Business Rules 表都必须完整填充——条件、动作、参数三要素不能省略。精简版只精简 §1、§3、§5、§8.2-§8.5、§9-§12 等章节，§7 的业务规则细节不打折。

用户可在执行计划确认时或最终撰写前要求调整撰写深度。

### understanding.md → PRD 映射检查（撰写后必做）

**撰写完成后，Agent 必须执行以下检查，确保问卷阶段收集的所有信息都体现在最终 PRD 中：**

1. 逐条扫描 understanding.md "已明确"区域的所有条目
2. 对每条检查：是否在 final PRD 的某个章节中有对应体现
3. 如果某条"已明确"内容在 PRD 中找不到对应：
   - 属于功能需求 → 补入 §7 对应 FR
   - 属于非功能需求 → 补入 §8
   - 属于业务规则 → 补入对应 FR 的 Business Rules 表
   - 属于假设/约束 → 补入 §11
   - 确实不适用 → 在检查记录中标注原因
4. 检查完成后，在聊天中报告："understanding.md 共 {N} 条已明确内容，{M} 条已映射到 PRD，{K} 条补充映射，{J} 条标注为不适用。"

**此检查在质量自检（`prd-quality-check.md`）之前执行，确保质量自检的输入是完整的 PRD。**

### 必须参考原始输入

**撰写最终 PRD 时，必须同时参考理解文档（understanding.md）和原始需求文件。**

- **Workspace 模式**：参考 `requirements/original.md` + `.process/understanding.md`，并可选参考 `workspace/knowledge/product/landscape.md`（撰写 §6 Business Process 和 §10 Dependencies 时，了解模块间依赖；生成 features.md 时，为每个 FR 标注涉及模块）
- **独立模式**：参考 `origin/PRD原文.md` + `understanding.md`

具体规则：
1. **先读取原始需求文件**：检查原始 PRD/输入中是否包含以下内容，如有则必须保留或迁移到最终 PRD 中：
   - 原型设计图、截图、Figma 链接
   - 已有的业务流程图
   - 数据埋点定义
   - 接口文档引用
   - 试点上线策略、回滚方案
   - 验收标准、毕业标准
   - 变更记录
   - 任何其他有价值的具体内容（如具体的数值指标、具体的技术方案细节）
2. **内容迁移原则**：
   - 原始输入中已有的具体内容（如原型截图链接、具体指标数值），直接迁移到最终 PRD 对应章节
   - 原始输入中的图片/截图引用，保留原始链接或引用方式，放入 §9 或相关章节
   - 原始输入中的 Confluence 附件引用（如 `<ac:image>`），保持原格式
   - 如果原始 PRD 来自 Confluence，其中的原型截图应在 §9.1 中以"参见原始页面"方式引用
3. **不能丢失的内容**：
   - 原始 PRD 中已有的原型设计图/截图/Figma 链接 → 必须出现在 §9
   - 原始 PRD 中已有的验收标准 → 必须出现在 §7.3 的 Acceptance Criteria 或独立章节
   - 原始 PRD 中已有的变更记录 → 必须出现在文档末尾的变更记录中

### Workspace 模式额外产物：features.md

在 Workspace 模式下，撰写 `requirements/final.md` 的同时，必须同步生成 `requirements/features.md`（功能点拆解）。

#### 生成步骤

1. **提取功能列表**：从 `requirements/final.md` §7 中提取所有 FR-xxx 功能需求，包括名称、描述、优先级
2. **提取用户故事和验收标准**：从 §7.3 的用户故事表中，将每个功能的 User Story 和 Acceptance Criteria 对应到功能点
3. **判定复杂度**：基于功能描述、涉及的场景数量、依赖关系，为每个功能标注复杂度（高/中/低）
4. **标注涉及模块**：
   - 读取 `workspace/knowledge/product/landscape.md`（如存在）获取功能模块清单
   - 基于每个 FR 的功能描述、业务流程、依赖关系，判断其涉及 landscape 中的哪些已有模块
   - 如果某个 FR 不属于任何已有模块（全新业务领域），标注为 `🆕 新增模块` 并给出建议的模块名称
   - 如果 landscape.md 不存在（全新 workspace），基于功能描述推断模块归属，全部标注为 `🆕 新增模块`
5. **标注涉及终端**：
   - 如果 `workspace/config/repos.json` 存在 → 读取仓库映射，为每个功能标注涉及的仓库和终端（如企微小程序、RN、PC、服务端）
   - 如果不存在 → 基于功能描述推断终端类型
6. **提取依赖关系**：从 §10 Dependencies 和功能描述中提取每个功能的依赖（其他功能、外部系统）
7. **生成 features.md**：**Agent 必须读取 steering: `prd-workspace-features.md`** 获取 features.md 模板格式，输出功能点总览表和每个功能的详情
8. **统计优先级分布**：在文件末尾汇总 Must Have / Should Have / Nice to Have 的数量
9. **交叉验证**：确认 features.md 中的功能列表与 final.md §7 完全一致（不多不少）

如果 `workspace/config/repos.json` 存在，在 features.md 的功能点详情中标注每个功能涉及的仓库（如企微小程序、RN、PC、服务端）。

**下一步衔接**：最终 PRD 撰写完成后，**Agent 必须读取 steering: `prd-wireframes.md`** 进入原型设计确认环节。

## 模板

```markdown
# Product Requirements Document (产品需求文档)

**Pipeline Name**: [产品名称]
**Pipeline ID**: [Pipeline ID]
**Pipeline Type**: [A类/B类/C类]
**Product Manager**: [PM姓名]
**Version**: v1.0
**Last Updated**: YYYY-MM-DD

---

## 1. 文档介绍

### 1.1 Document Purpose (文档目的)

### 1.2 Document Scope (文档范围)

### 1.3 Readers (读者对象)

### 1.4 Reference Documents (参考文档)

格式：[标识符] 作者，文献名称，出版单位（或归属单位），日期

### 1.5 Terms and Abbreviation (术语与缩写解释)

| Terms and Abbreviation | Explanation |
| :--------------------- | :---------- |
|                        |             |

---

## 2. Product Overview (产品概述)

### 2.1 Background (产品背景)

### 2.2 Target Users (目标用户)

- **Primary Users**:
- **Secondary Users**:

### 2.3 Success Metrics (成功指标)

---

## 3. Standards or Specifications (标准或规范)

---

## 4. Product Range (产品范围)

---

## 5. Product Roles (产品角色)

| Role Name | Responsibility Description |
| :-------- | :------------------------- |
|           |                            |

---

## 6. Business Process (业务流程说明)

---

## 7. Functional Requirements (功能需求)

### 7.1 功能性需求分类

| Function Type | Sub-Function |
| :------------ | :----------- |
|               |              |

### 7.2 FR-001: [功能名称]

| Item            | Detail                               |
| :-------------- | :----------------------------------- |
| **Description** | [功能描述]                            |
| **Priority**    | [Must Have/Should Have/Nice to Have] |
| **Trigger**     | [触发方式：用户操作/系统自动/外部调用/定时] — [触发条件] |
| **User Flow**   | [步骤1] → [步骤2]                    |
| **Remarks**     |                                      |

#### Business Rules for FR-001

| 规则编号 | 条件（When） | 动作（Then） | 参数/数值 | 依据（Why，可选） |
| :------- | :----------- | :----------- | :-------- | :---------------- |
| BR-001-1 | [什么条件下] | [执行什么动作] | [具体参数值/范围/默认值] | [为什么这样定] |
| BR-001-2 | ... | ... | ... | ... |

> 每条业务规则必须包含"条件-动作-参数"三要素。缺少任何一个要素的规则视为不完整。
> 如果参数值未经用户确认，标注 `[待确认]`。
> "依据"列用于记录关键决策的背景（如"业务方要求"、"参考竞品X"、"性能考虑"），非必填但建议关键规则填写。

### 7.3 User Stories (用户故事)

| Feature | User Story | Priority | Story Points | Acceptance Criteria |
| :------ | :--------- | :------- | :----------- | :------------------ |
|         |            |          |              |                     |

---

## 8. Non-Functional Requirements (非功能需求)

### 8.1 Performance (性能)

| Content       | Requirements |
| :------------ | :----------- |
| Response Time |              |
| Throughput    |              |

### 8.2 Software and Hardware Environment (软硬件环境)

| Content  | Requirements |
| :------- | :----------- |
| Software |              |
| Hardware |              |

### 8.3 Product Quality (产品质量)

| Content    | Requirements |
| :--------- | :----------- |
| 正确性     |              |
| 健壮性     |              |
| 可靠性     |              |
| 性能，效率 |              |
| 易用性     |              |
| 清晰性     |              |
| 安全性     |              |
| 可扩展性   |              |
| 兼容性     |              |
| 可移植性   |              |

### 8.4 Security (安全)

| Content        | Requirements |
| :------------- | :----------- |
| Authentication |              |
| Authorization  |              |

### 8.5 Usability (可用性)

| Content   | Requirements |
| :-------- | :----------- |
| Usability |              |

---

## 9. UI/UX Requirements (界面需求)

### 9.1 Wireframes (线框图)

{由原型设计环节生成，引用截图目录下的截图}
{Workspace 模式: requirements/wireframes/ / 独立模式: output/wireframes/}
{如果回填 Confluence，截图作为附件上传并用 ac:image 引用}
{如果用户已有 Figma 设计稿，直接引用 Figma 链接}

### 9.2 Design Specifications (设计规格)

---

## 10. Dependencies (依赖)

| Dependency | Type | Owner | Status |
| :--------- | :--- | :---- | :----- |
|            |      |       |        |

---

## 11. Assumptions & Constraints (假设与约束)

### 11.1 Assumptions (假设)

### 11.2 Constraints (约束)

---

## 12. Appendix (附录)

---
```
