---
inclusion: manual
---

# 原型设计环节

## 目标

基于最终 PRD 的功能需求和 UI 描述，生成可视化原型页面，截图后填入 PRD 的 §9 UI/UX Requirements 章节与并在对应FR新增行：Prototype、UI Specs，填入截图与界面交互需求。

## 前置条件

- 最终 PRD 已撰写完成（Workspace 模式: `requirements/final.md` / 独立模式: `output/完整PRD.md`）

## 用户确认（必须步骤）

**不要自动跳过原型设计环节，必须询问用户。**

撰写完最终 PRD 后，Agent 必须询问用户：

### 情况一：原始 PRD 已有原型/截图

如果原始需求文件中包含原型截图、Figma 链接或界面设计图：

```
📐 我注意到原始 PRD 中已包含原型界面截图/设计稿，已引用到最终 PRD 的 §9 章节。

你希望：
A) 保持现有原型，不需要重新生成
B) 基于完善后的需求重新生成原型页面
C) 补充生成原始 PRD 中没有覆盖的页面原型
```

### 情况二：没有已有原型

```
📐 PRD 撰写完成。接下来是否需要我生成原型线框图？

我可以基于 PRD 中的功能需求，使用组件库生成核心页面的原型并截图。

A) 需要，帮我生成原型
B) 不需要，跳过原型设计
C) 我有 Figma 设计稿，给你链接
```

用户明确选择后才继续。

## judy power 可用性检查

当用户选择需要生成原型（情况一选 B/C，或情况二选 A）时，Agent 必须检查 judy power 是否可用：

1. 尝试激活 judy power（通过 kiroPowers action="activate" powerName="judy"）
2. 如果激活成功 → 继续原型设计流程
3. 如果激活失败（power 未安装）→ 在聊天中告知用户：
   ```
   ⚠️ 原型设计需要 judy power（MCD IT 前端工程师），但当前未安装。
   
   你可以：
   A) 安装 judy power 后继续（安装地址：https://gitlab-ex.mcd.com.cn/hub/powers/mcd.power-judy）
   B) 不使用 judy power 直接帮我设计
   C) 我有 Figma 设计稿，给你链接直接引用
   D) 跳过原型设计，直接进入质量自检
   ```
4. 等待用户选择后再继续

## 执行流程

### 1. 分析 UI 需求

从
- 最终 PRD（Workspace 模式: `requirements/final.md` / 独立模式: `output/完整PRD.md`）中了解需求
- 理解文档（understanding.md）中关于布局、列表、表单的决策
确定需要生成哪些页面的原型。

### 2. 生成原型页面代码

基于 PRD 需求，优先使用 power judy 生成每个页面的原型代码：
- 原型代码放在原型代码目录（Workspace 模式: `.process/prototype/` / 独立模式: `output/prototype/`）
- 使用 Mock 数据填充页面内容

### 3. 预览与截图

启动本地预览后，使用 chrome-devtools MCP 截图：
- `navigate_page` 导航到各个原型页面
- `take_screenshot` 截取页面截图
- 截图保存到原型截图目录（Workspace 模式: `requirements/wireframes/` / 独立模式: `output/wireframes/`）
- 命名规范：`{页面名}-{视图}.png`，如 `list-page-desktop.png`

如需展示不同状态：
- 使用 `click`、`fill` 等操作模拟用户交互
- 截取不同状态下的页面（空状态、加载中、数据填充后等）

如需展示移动端：
- 使用 `emulate` 设置移动端 viewport
- 截取移动端视图

### 4. 更新 PRD

- 在截图对应FR新增行：Prototype、UI Specs，分别填入对应截图引用与界面交互需求。
- 将截图引用填入最终 PRD 的 §9 章节：

```markdown
## 9. UI/UX Requirements (界面需求)

### 9.1 Wireframes (线框图)

#### 列表页
![列表页](wireframes/list-page-desktop.png)

#### 详情页
![详情页](wireframes/detail-page-desktop.png)

#### 表单页
![表单页](wireframes/form-page-desktop.png)

### 9.2 Design Specifications (设计规格)
- ...
```

**注意**：Workspace 模式下截图路径使用相对于 `requirements/` 的路径（如 `wireframes/list-page-desktop.png`），独立模式下使用相对于 `output/` 的路径。

如果最终要回填 Confluence，截图需要作为附件上传：
- 使用 confluence-mcp 的 `upload_attachment` 上传截图
- 在页面内容中使用 `<ac:image>` 标签引用附件

## 注意事项

- 每个功能至少一张核心页面截图
- 如果用户已有 Figma 设计稿，跳过此环节，直接在 PRD 中引用 Figma 链接

**下一步衔接**：原型设计完成（或跳过）后，**Agent 必须读取 steering: `prd-quality-check.md`** 进入质量自检环节。
