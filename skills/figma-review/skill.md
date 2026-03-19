---
name: figma-review
description: 根据组件名到 Figma node 的映射表，使用 Figma MCP 审查并修改每个组件的样式
---

# Figma 设计审查

根据用户提供的组件名到 Figma node 的映射表，审查并修改 Figma 中每个组件的样式。

## 流程

1. **获取映射表**
   - 询问用户提供组件名到 Figma node 的映射表
   - 格式示例：
     ```
     Button: https://figma.com/design/xxx/xxx?node-id=1-2
     Card: https://figma.com/design/xxx/xxx?node-id=3-4
     ```
   - 或者接受用户直接提供 node ID 列表

2. **获取 Figma Token**
   - 询问用户是否已配置 Figma Token（通常已作为 Figma MCP 的环境变量配置）
   - 确保 Figma MCP 连接正常

3. **逐个审查组件**
   - 对于映射表中的每个组件：
     a. 使用 Figma MCP 的 `get_design_context` 获取该 node 的设计信息
     b. 分析组件的样式（颜色、字体、间距、圆角等）
     c. 根据组件名推断期望的样式规范
     d. 如有需要，使用 Figma MCP 修改组件样式

4. **记录修改建议**
   - 对于每个审查的组件，记录：
     - 当前样式
     - 建议修改
     - 已执行的修改
   - 生成审查报告

## 工具使用

- **Figma MCP**:
  - `get_design_context` - 获取组件设计信息
  - `get_variable_defs` - 获取设计 token/变量
  - 其他 Figma MCP 提供的样式修改工具

## 输出

- 审查报告，包含每个组件的：
  - 当前样式快照
  - 修改建议
  - 实际修改记录
