---
inclusion: manual
---

# Workspace 模式：知识沉淀与阶段流转

**加载时机**：PRD 完成后（质量自检通过 + 输出回填完成），执行知识沉淀和阶段流转时加载。

## 知识沉淀

PRD 完成后，Cony 应将可复用的业务知识沉淀到 workspace 的 knowledge 目录：

1. **检查目录和文件**：如果 `workspace/knowledge/product/` 不存在，创建目录并初始化：
   - `landscape.md`（空产品全景模板）
   - `glossary.md`（空术语表模板）
   - `modules/` 目录
2. **更新产品全景** → 更新 `workspace/knowledge/product/landscape.md`：
   - 模块清单：新模块 → 追加条目；已有模块有变更 → 更新状态和描述。参考 `requirements/features.md` 中标注为 `🆕 新增模块` 的条目，将其作为新模块追加到全景的功能模块清单
   - 迭代变更摘要：追加本次 story 的变更摘要（一句话），只保留最近 10 条
   - 使用 `strReplace` / `fsAppend` 增量更新，禁止 `fsWrite` 覆盖
3. **更新模块详情** → 如果 PRD 涉及新的业务模块（参考 features.md 中 `🆕 新增模块` 标注），在 `workspace/knowledge/product/modules/` 下创建 `{模块名}.md`；已有模块则增量追加新功能点（参考 features.md 中该模块关联的 FR 列表）
4. **业务术语** → 检查 `requirements/final.md` §1.5 中的术语，增量追加到 `workspace/knowledge/product/glossary.md`（去重，不覆盖已有条目）
5. **更新 state.md** → 记录知识沉淀完成
6. **追加 session.log.md** → 记录沉淀了哪些术语和模块

知识沉淀是可选步骤，在阶段流转之前执行。Agent 在聊天中简要告知："已将本次 PRD 中的业务术语和模块知识沉淀到 workspace/knowledge/product/。"

### landscape.md 模板

```markdown
# 产品全景

> 最后更新: {日期}
> 更新来源: {story 名称}

## 系统概述

- 系统名称: {名称}
- 系统定位: {一句话描述}
- 目标用户: {用户群体}
- 技术栈: {前端/后端/基础设施}
- 终端覆盖: {企微小程序 / RN / PC / 服务端}

## 功能模块清单

| 模块名称 | 一句话描述 | 状态 | 负责人 | 详情 |
|----------|-----------|------|--------|------|
| {模块名} | {描述} | {活跃/稳定/规划中} | {负责人} | [详情](modules/{模块名}.md) |

## 模块依赖关系

{模块间的关键依赖，文字或简图}

## 迭代变更摘要

| 日期 | Story | 变更内容 |
|------|-------|----------|
| {日期} | {story名} | {一句话描述变更} |
```

## 阶段流转集成

### PRD 完成后的流转

当 Cony 完成 PRD 精炼（质量自检通过 + 输出回填完成）后：

1. **执行知识沉淀**（可选，见上方）

2. **询问用户是否流转**：
```
✅ PRD 已完成并通过质量自检。

产物清单：
- requirements/original.md（原始需求）
- requirements/final.md（完整 PRD）
- requirements/features.md（功能点拆解，共 {N} 个功能点）
- requirements/wireframes/（原型截图，{N} 张）

是否将需求流转到设计阶段？流转后会自动通知架构师和开发同学。

A) 流转到设计阶段
B) 暂不流转，我还需要再看看
```

3. **用户确认流转后**：
   - 调用 workspace-hub 的 `advance_stage` 工具
   - 传递产物路径信息（requirements/final.md、requirements/features.md）
   - 更新 state.md，标记需求阶段为"已完成"，记录完成时间
   - 追加 session.log.md 流转记录
   - 在聊天中确认："已流转到设计阶段，架构师和开发同学会收到通知。"

### 设计阶段退回处理

当设计阶段（archie）通过 `return_stage` 将 story 退回需求阶段时：

1. Agent 检测到 story 状态变更（通过 state.md 变化或用户告知）
2. 读取退回原因（从 state.md 的设计阶段部分获取，退回原因由 Archie 写入 state.md）
3. 更新 state.md，标记需求阶段为"退回修正中"
4. 追加 session.log.md 退回记录
5. 进入变更管理流程（**Agent 必须读取 steering: `prd-change-management.md`**）：
   - 小问题：直接修改 requirements/final.md + requirements/features.md，重跑质量自检
   - 大问题：回退到对应阶段重新走问卷（在 `.process/questionnaires/` 下生成新问卷）
6. 修复完成后，再次询问是否流转

## 注意事项

- 阶段流转必须用户确认，不自动触发
- 退回处理复用已有的变更管理机制，不引入新流程
