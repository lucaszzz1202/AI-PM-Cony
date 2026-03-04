---
inclusion: manual
---

# Confluence 集成

通过 confluence-mcp 工具实现文档读写。

**加载时机**：用户提供 Confluence 链接时（输入获取），或用户选择回填 Confluence 时（输出回填）。

**路径映射**：以下规范中的文件路径根据工作模式不同而不同。如需查看完整映射关系，**Agent 必须读取 steering: `prd-file-specs.md`**（输入获取阶段已加载；如未加载则需显式读取）。

## 输入：从 Confluence 读取

用户提供 Confluence 页面链接时：
1. 从链接中提取 page ID
2. 调用 confluence-mcp 的 `get_page` 读取内容
3. 保存到原始需求文件（Workspace 模式: `requirements/original.md` / 独立模式: `origin/PRD原文.md`）作为只读工作副本
4. 尝试用 confluence-mcp 的 `search` 在同一 space 下搜索关联文档（如历史 PRD、技术架构、接口文档），作为推导的额外上下文
5. 后续流程基于本地副本 + 关联文档进行

## 输出：回填到 Confluence

最终 PRD 撰写完成后：
1. 先生成本地最终 PRD（Workspace 模式: `requirements/final.md` / 独立模式: `output/完整PRD.md`）
2. 询问用户：回填到原页面 / 创建新页面 / 仅保留本地
3. 如果回填：生成 Confluence 回填 HTML（Workspace 模式: `.process/confluence-export.html` / 独立模式: `output/Confluence回填.html`），调用 confluence-mcp 的 `update_page` 更新原文档
4. 如果新建：调用 confluence-mcp 的 `create_page`，需要用户提供 space key 和可选的 parent page ID
5. 回填时将 markdown 转换为 Confluence 支持的 HTML 格式
