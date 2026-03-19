---
name: figma-review
description: 根据组件名到 Figma node 的映射表，使用 Figma MCP 获取设计稿样式并修改代码中的组件样式
---

# Figma 样式同步

根据用户提供的组件名到 Figma node 的映射表，获取 Figma 设计稿中的样式，并自动修改代码中对应组件的样式。

## 流程

1. **获取映射表**
   - 询问用户提供组件名到 Figma node 的映射表
   - 格式示例：
     ```
     Button: https://figma.com/design/xxx/xxx?node-id=1-2
     Card: https://figma.com/design/xxx/xxx?node-id=3-4
     ```
   - 或接受简化的 `组件名: nodeId` 格式

2. **获取 Figma 设计信息**
   - 对于映射表中的每个组件：
     a. 使用 Figma MCP 的 `get_design_context` 获取该 node 的设计信息
     b. **提取所有可用的样式属性**，包括但不限于：
        - **颜色**：背景色、文字色、边框色、填充色、渐变色
        - **字体**：字号、字重、行高、字间距、字体家族、文本装饰、文本对齐
        - **间距**：内边距（padding）、外边距（margin）、间隙（gap）
        - **尺寸**：宽度、高度、最小/最大宽高、宽高比
        - **布局**：display 类型、flex 属性（direction、justify、align、wrap）、grid 属性
        - **位置**：position 类型、top/left/right/bottom、z-index、transform（旋转、缩放、位移）
        - **效果**：阴影（box-shadow、text-shadow）、透明度（opacity）、混合模式、滤镜、裁剪
        - **边框**：边框宽度、样式、颜色、圆角（border-radius）、轮廓
        - **交互状态**：hover、focus、active、disabled 等状态的样式差异
        - **响应式**：不同断点的样式变化
     c. **AI 自主决策**：根据目标项目的样式方案（Tailwind、CSS Modules、Styled-components 等），智能选择需要提取和应用的样式属性，无需提取所有属性

3. **查找代码中的对应组件**
   - 在项目中搜索与组件名匹配的文件
   - 常见命名模式：
     - `Button.tsx` / `button.tsx`
     - `Button/index.tsx`
     - `components/Button.tsx`
   - 如果找到多个匹配，询问用户确认

4. **修改组件样式**
   - 根据 Figma 设计稿提取的样式，修改代码中的样式：
     - **Tailwind**: 更新 className 中的工具类
     - **CSS Modules**: 修改 .module.css 中的样式规则
     - **Styled-components**: 更新样式模板字符串
     - **CSS-in-JS**: 修改样式对象
   - 保持组件结构和交互逻辑不变，仅修改视觉样式

5. **验证修改**
   - 检查修改后的代码语法是否正确
   - 如有 TypeScript 类型错误，尝试自动修复或提示用户
   - 询问用户是否需要预览修改效果

6. **生成修改报告**
   - 记录每个组件的修改内容：
     - 修改的文件路径
     - 样式变更摘要（颜色、字体、间距等）
     - 修改前后的对比

## 工具使用

- **Figma MCP**:
  - `get_design_context` - 获取组件设计信息
  - `get_variable_defs` - 获取设计 token/变量
  - `get_screenshot` - 获取组件截图用于对比

- **代码修改**:
  - 使用 `Read` 和 `Edit` 工具修改代码文件
  - 使用 `Grep` 搜索组件文件

## 输入

- 组件名到 Figma node 的映射表
- 项目中的组件代码文件

## 输出

- 修改后的组件代码文件
- 修改报告，包含：
  - 处理的组件列表
  - 每个组件的样式变更详情
  - 修改的文件路径

## 注意事项

- 修改前建议用户先提交当前代码，以便必要时回滚
- 如果 Figma 设计稿使用了设计 token，尝试映射到项目中的对应 token
- 对于复杂的样式（如渐变、动画），可能需要人工介入
- 保持组件的可访问性（Accessibility）属性不变
