---
inclusion: manual
---

# Workspace 模式：features.md 功能点拆解

**加载时机**：最终撰写阶段，Workspace 模式下生成 `requirements/features.md` 时加载。

## 概述

这是 Workspace 模式下的新增产物。在最终撰写 `requirements/final.md` 的同时，Agent 必须同步生成 `requirements/features.md`。

**内容来源**：从 `requirements/final.md` 的 §7 功能需求中提取。

## 生成步骤

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
7. **生成 features.md**：按下方模板输出功能点总览表和每个功能的详情
8. **统计优先级分布**：在文件末尾汇总 Must Have / Should Have / Nice to Have 的数量
9. **交叉验证**：确认 features.md 中的功能列表与 final.md §7 完全一致（不多不少）

## features.md 模板

```markdown
# {story-name} — 功能点拆解

> 版本: v1.0
> 最后更新: {日期}
> 来源: requirements/final.md §7

## 功能点总览

| 编号 | 功能名称 | 优先级 | 复杂度 | 涉及模块 | 涉及终端 | 简要描述 |
|------|----------|--------|--------|----------|----------|----------|
| FR-001 | {名称} | Must Have | {高/中/低} | {模块名} | {终端} | {一句话描述} |
| FR-002 | {名称} | Should Have | {高/中/低} | {模块名} | {终端} | {一句话描述} |
| ... | | | | | | |

## 功能点详情

### FR-001: {功能名称}

- 优先级: {Must Have / Should Have / Nice to Have}
- 复杂度: {高/中/低}
- 涉及模块: {模块名称，对应 landscape.md 功能模块清单中的模块；新模块标注"🆕 新增模块"}
- 涉及终端: {企微小程序 / RN / PC / 服务端}
- 用户故事: {As a [角色], I want [功能], so that [价值]}
- 验收标准:
  - [ ] {AC-1}
  - [ ] {AC-2}
- 依赖: {依赖的其他功能或外部系统}
- 备注: {补充说明}

### FR-002: {功能名称}
...

## 优先级分布

- Must Have: {N} 个
- Should Have: {N} 个
- Nice to Have: {N} 个
```

## 质量自检同步更新规则

如果质量自检阶段修改了 `requirements/final.md` 的 §7 功能需求（新增/删除/修改 FR）或 §7.3 用户故事/验收标准，必须同步更新 features.md。其他章节的修改不需要同步。
