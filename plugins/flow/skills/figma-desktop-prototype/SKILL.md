---
name: figma-desktop-prototype
description: 根据多个 Figma 设计稿批量生成完整的 HTML 原型页面，自动下载图标资源，提取设计 token 生成统一的 Tailwind 配置；全流程使用 Figma Desktop MCP（get_design_context + get_variable_defs + use_figma）
---

## 概述

本 skill 与 `figma-prototype` 的目标一致：批量生成 HTML 原型页面、提取设计 token、下载图标资源并输出汇总报告。  
**关键区别**：本 skill **只使用 Figma Desktop MCP**，不使用远程 Figma MCP。

核心能力：
1. 批量处理多页面原型
2. 提取并合并设计 Token（颜色、间距、字体等）
3. 导出并去重图标资源（SVG/PNG）
4. 生成页面、导航入口和汇总报告

## 编写原则（重要）

本 skill 是“流程引导型”，不是“脚本模板型”。

- 重点是引导 AI 按步骤完成任务与校验结果。
- 不提供大段 JavaScript 示例，不要求复制/粘贴长代码。
- 当工具需要代码参数时，只允许最小必要表达，优先描述目标、输入、输出和验收条件。
- 每一步都要先说明意图，再执行，再回报结果。

---

## Desktop MCP 与远程 MCP 交互差异（必须遵守）

| 维度 | Figma Desktop MCP | 远程 Figma MCP |
|---|---|---|
| 主要工具 | `get_design_context` + `get_variable_defs` + `use_figma` | `get_design_context` / `get_variable_defs` |
| 交互方式 | 优先传 `fileKey + nodeId + dirForAssetWrites`；必要时用 `use_figma` 补充 | 传 `fileKey + nodeId` 拉取结构化上下文 |
| 资源获取 | `get_design_context` 写盘到本地目录；必要时 `use_figma` 补导出 | `assetDownloadUrls` 返回预签名下载链接 |
| 上下文来源 | `figma.currentPage` / `figma.currentPage.selection` | 指定节点上下文 |
| 前置要求 | 必须打开 Figma Desktop 并打开目标文件 | 可仅凭 URL 远程读取 |

**硬性约束：**
- 必须优先使用 `get_design_context` 获取页面上下文与资产信息。
- 每次调用 `get_design_context` 时，`dirForAssetWrites` 必须传入绝对路径。
- `get_variable_defs` 仅允许“单节点调用”，每次只传一个 `nodeId`。
- `use_figma` 仅用于补充：当 `get_design_context` 返回信息不足或个别资源导出失败时再调用。
- 禁止调用“远程 Figma MCP 实例”；只允许调用 Figma Desktop MCP 实例。

### MCP 调用白名单（必须遵守）

- 允许：`get_design_context`、`get_variable_defs`、`use_figma`
- 禁止：远程 Figma MCP 实例上的同名工具调用

执行要求：
1. `get_design_context` 必须带 `fileKey`、`nodeId`、`dirForAssetWrites`（绝对路径）。
2. `nodeId` 必须使用冒号格式（如 `1340:14515`），不能使用短横线格式。
3. `dirForAssetWrites` 默认值使用 `ASSET_WRITE_DIR`。
4. 调用 `get_variable_defs` 时必须显式传入单个 `nodeId`，禁止依赖“当前多选状态”。
5. 多个页面节点时，`get_variable_defs` 必须按节点串行调用并汇总结果。
6. 如需补充导出，再调用 `use_figma`，但不能跳过前置的 `get_design_context`。

### 重点报错预防

历史报错：
`Error: Path for asset writes as tool argument is required for write-to-disk option`

预防规则：
1. 任何 `get_design_context` 调用都必须传 `dirForAssetWrites`。
2. `dirForAssetWrites` 必须是绝对路径，不能是相对路径。
3. 调用前先确保目录存在。
4. 调用失败时先检查参数，不要直接替换成其它工具调用。
5. 任何 `get_variable_defs` 调用都必须确保“当前仅处理一个 nodeId”。

---

## 执行流程

**严格按步骤执行，不跳步。完成一步后再进入下一步。**

### 第 0 步：初始化本地目录（兜底）

开始前先确认以下目录存在：
- `/Users/a1/development/flow/public/figma-mcp-assets`
- `/Users/a1/development/flow/public/icons`

并在执行上下文记录：
- `ASSET_WRITE_DIR=/Users/a1/development/flow/public/figma-mcp-assets`

### 第 1 步：收集 Figma 链接与页面范围（支持多 URL）

询问用户提供 Figma URL（可多个），并说明：
- 每行一个链接，或逗号分隔
- 建议从 Figma Dev 模式复制页面链接

解析每个 URL（用于记录和校验）：
- `fileKey`：`/design/{fileKey}/...`
- `nodeId`：`node-id=1-2` 需转换为 `1:2`

将所有 `nodeId` 归一化后写入 `NODE_QUEUE`（数组），后续涉及 `get_variable_defs` 时严格按 `NODE_QUEUE` 串行处理，一次只处理一个 `nodeId`。

> 说明：Desktop MCP 主要依赖当前打开文件和选择节点，URL 在本 skill 中用于“记录、校验、命名映射”，不是远程拉取入口。

---

### 第 2 步：确认 Desktop 运行前提

执行前必须确认：
1. 用户已安装并登录 Figma Desktop
2. 用户已打开目标 Figma 文件
3. 用户已选中要生成原型的页面根节点/Frame（可多选）

若用户提供的 URL 存在多个 `fileKey`，必须提示：
- Desktop 模式一次仅处理当前打开文件
- 需要用户分批切换文件后重复执行

---

### 第 3 步：提取页面结构与设计 Token（get_design_context + get_variable_defs）

优先使用 `get_design_context` 获取以下信息：
- 选中节点的页面结构（名称、层级、尺寸、布局关键属性）
- 本地变量（Variables）与样式（Styles）

调用参数规范（必须满足）：
```text
{
  fileKey: "<从 URL 解析>",
  nodeId: "<例如 1340:14515>",
  dirForAssetWrites: "/Users/a1/development/flow/public/figma-mcp-assets"
}
```

执行要点：
1. 对每个页面节点都调用一次 `get_design_context`，并始终附带 `dirForAssetWrites`。
2. 输出应包含：`fileName`、`pageName`、`nodes`、`variables` 四类数据。
3. 结构信息至少覆盖：节点类型、名称、尺寸、层级关系。
4. 变量定义细节优先通过 `get_variable_defs` 获取，但必须按 `NODE_QUEUE` 一次一个 `nodeId` 调用。
5. 多个节点时严禁一次性调用 `get_variable_defs`（会触发多选报错）。
6. 变量信息至少覆盖：变量名称、类型、可用模式值。
7. 遇到数据缺失时，先记录缺失项，再使用 `use_figma` 做最小补充，不要整体切换流程。

期望输出结构（示意）：
```text
{
  fileName: "...",
  pageName: "...",
  nodes: [...],
  variables: [...]
}
```

从返回结果提取 token 并映射到 Tailwind：
- `color/*` -> `theme.extend.colors`
- `spacing/*` -> `theme.extend.spacing`
- `radius/*` -> `theme.extend.borderRadius`
- `font/*` -> `theme.extend.fontFamily/fontSize`

---

### 第 4 步：生成统一 Tailwind 配置

基于第 3 步输出的 token 生成 `tailwind.config.js`。

要求：
- 配置为所有生成页面共享
- 命名统一为 kebab-case
- 保留原变量名到 Tailwind 名的映射关系（用于报告）

---

### 第 5 步：使用 get_design_context 导出图标/Logo 资源（不能跳过）

优先使用 `get_design_context` + `dirForAssetWrites` 输出资源到本地目录，再做筛选与去重。

调用规则：
1. 每个页面节点都要调用 `get_design_context`。
2. 每次调用都必须传 `dirForAssetWrites=ASSET_WRITE_DIR`。
3. 资产写盘后，从写入目录与返回结果中共同汇总资源清单。
4. 仅当部分资源未成功写盘时，才使用 `use_figma` 对缺失项补导出。

执行要点：
1. 导出项需记录：节点 ID、节点名、页面标签、格式、写盘路径、导出结果。
2. 导出失败时记录错误信息但不中断全流程。
3. 完成后输出“原始导出清单”，供筛选与去重步骤使用。

期望输出结构（示意）：
```text
[
  { nodeId, nodeName, pageLabel, format, filePath },
  { nodeId, nodeName, pageLabel, format, error }
]
```

筛选规则（与 `figma-prototype` 一致）：
- 包含：`.svg`、`.png`，或名称包含 `icon/logo/symbol`
- 排除：`.jpg/.jpeg/.gif/.webp`

去重规则：
- 基于 `{normalizedName + format + bytesLength}` 去重（Desktop 无 URL 可用）
- 统计总数、去重后数量、失败数量

---

### 第 6 步：保存资源到本地目录

将 `get_design_context` 已写盘资源整理并规范化到目标目录：
- 读取目录：`ASSET_WRITE_DIR`
- 输出目录：`/public/icons`
- 可选结构：`/public/icons/common`、`/public/icons/{页面名}`

保存规范：
- 文件名清洗（仅保留字母、数字、`-`、`_`）
- 格式映射：`SVG -> .svg`，`PNG -> .png`，`JPG -> .jpg`
- 失败项写入清单，最终汇总

---

### 第 7 步：生成图标入口文件

生成 `components/icons/index.ts`（配合 SVGR 使用），导出通用图标和页面图标。

要求：
- 仅引用第 6 步已落盘文件
- 名称统一 PascalCase（例如 `IconSearch`）

---

### 第 8 步：批量生成 HTML 原型页面（/prototype）

为每个选中页面/Frame 生成一个 HTML 页面到 `/prototype`。

约束：
- 使用 `html + tailwindcss`
- 必须优先复用第 6 步导出的图标资源，不使用 inline SVG
- 页面中禁止加入移动端状态栏（时间/电池等）

建议模板（与 `figma-prototype` 保持一致）：
- `<script src="https://cdn.tailwindcss.com"></script>`
- 引入共享 `tailwind.config.js`
- 引入项目样式文件

---

### 第 9 步：生成导航入口页

创建 `/prototype/index.html`：
- 展示页面列表（名称 + 路径）
- 支持点击跳转到每个原型页
- 展示 token 预览与资源统计

---

### 第 10 步：生成汇总报告

创建 `/prototype/prototype.md`，至少包含：
1. 输入信息（URL、页面范围、处理时间）
2. Desktop 执行信息（当前文件名、当前页面、选择节点）
3. Token 提取结果（含映射关系）
4. 资源导出统计（成功/失败/去重）
5. 生成页面清单与路径
6. 已知限制与后续建议

---

## 错误处理与回退策略

1. `get_design_context` 执行失败：
- 优先检查 `fileKey`、`nodeId`、`dirForAssetWrites` 是否齐全
- 确认 `nodeId` 为冒号格式（如 `1340:14515`）
- 确认 `dirForAssetWrites` 是绝对路径且目录真实存在

2. `get_variable_defs` 执行失败：
- 若报错为 `Multiple nodes selected. Only selecting a single node is supported.`，根因是当前调用在多选上下文触发
- 立即改为“按 `NODE_QUEUE` 串行调用”，并在每次调用中显式传单个 `nodeId`
- 不要使用“当前选中多个节点”的默认上下文进行重试

3. `use_figma` 执行失败：
- 检查 Figma Desktop 是否打开、是否登录、是否打开正确文件
- 检查是否有选中节点；若无，提示用户重新选择 Frame 后重试

4. 命中以下报错：
- `Path for asset writes as tool argument is required for write-to-disk option`

恢复步骤：
1. 保留 `get_design_context` 调用链路，不要切换工具
2. 补齐 `dirForAssetWrites=ASSET_WRITE_DIR` 后重试
3. 若路径仍无效，改用绝对路径 `/Users/a1/development/flow/public/figma-mcp-assets` 重试
4. 仅在单个节点持续失败时，才对该节点使用 `use_figma` 补充
5. 在报告中记录“已触发参数修复重试”

5. 多文件输入：
- 若 URL 覆盖多个 `fileKey`，按文件分批执行，避免在一次执行中混用

6. 资源导出失败：
- 单资源最多重试 3 次
- 失败项保留到报告，不阻塞其余资源导出

7. token 缺失：
- 若未读取到 Variables，则回退到节点显式样式提取（fill/text/spacing）

---

## 完成后输出

向用户汇报：
1. 生成了哪些文件（`/prototype`、`/public/icons`、`tailwind.config.js`、`prototype.md`）
2. 导出资源统计与失败项
3. 与远程 MCP 流程的差异点（本次是否完全遵循 Desktop 模式）
