# Flow

AI 协作开发工作流插件与技能库。

本仓库提供一套从需求分析到缺陷修复的标准化 AI 协作流程，核心内容是可复用的 `skills` 提示词模板与流程编排文档，适用于新项目启动、复杂功能开发和持续迭代场景。

## 项目定位

- 类型：工作流/技能仓库（不是业务应用源码）
- 目标：将 AI 协作开发过程标准化、可追踪、可复用
- 核心能力：需求理解与技术选型、任务拆解与执行计划、多 Agent 协同开发与 TDD 约束、浏览器自动化验收、代码评审与 E2E 测试补齐、缺陷单处理（单一假设 + 最小改动）

## 目录结构

```text
flow/
├── .claude-plugin/
│   └── plugin.json                  # 插件元信息
├── docs/
│   └── workflow.md                  # 方法论文档
├── skills/
│   ├── requirements/                # 需求理解阶段
│   ├── planning/                    # 规划阶段
│   ├── coding/                      # 核心开发阶段
│   ├── auto-browser/                # 浏览器自动化验收
│   ├── code-review/                 # 质量评审
│   ├── write-test/                  # E2E 测试编写
│   ├── bugfix/                      # 缺陷处理
│   ├── api-connect/                 # Mock API 转真实 API
│   ├── figma-prototype/             # Figma 转原型页面
│   ├── figma-desktop-prototype/     # Figma Desktop MCP 转原型页面
│   ├── figma-assets-downloader/     # Figma 资源下载
│   ├── figma-review/                # 设计稿视觉比对
│   └── hello/                       # 示例技能
└── README.md
```

## 核心工作流

按阶段顺序执行完整开发流程：

1. `requirements`：需求理解与技术选型
2. `planning`：任务拆解与计划文档
3. `coding`：按计划执行开发（含多 Agent 约束）
4. `code-review`：质量反馈循环与改进
5. `bugfix`：缺陷单分析与修复闭环

方法论说明见：`docs/workflow.md`

## 常用 Skills

- `/requirements`：执行需求分析阶段
- `/planning`：生成开发计划
- `/coding`：按计划进行开发实施
- `/auto-browser`：按验收项做自动化浏览器验证
- `/code-review`：执行代码质量评审与改进
- `/write-test`：按验收标准编写 Playwright E2E
- `/bugfix`：按单一假设法处理缺陷
- `/figma-prototype`：由 Figma 批量生成原型页面
- `/figma-desktop-prototype`：仅使用 Figma Desktop MCP 生成原型页面
- `/figma-review`：Figma 与实现页面截图比对
- `/api-connect`：将 mock 调用替换为 OpenAPI 生成 SDK 调用

## 典型使用方式

按阶段顺序调用各 skill：

1. 调用 `/requirements` 完成需求分析
2. 调用 `/planning` 生成开发计划
3. 调用 `/coding` 按计划执行开发
4. 调用 `/code-review` 进行质量评审
5. 如有缺陷，调用 `/bugfix`

## 阶段产物约定（建议）

为保证跨阶段衔接稳定，建议统一以下文档产物：

- 需求文档：`docs/requirements.md`
- 执行计划：`docs/<feature>-plan.md`
- E2E 用例清单：`docs/e2e-test-case.md`
- 原型生成报告：`prototype/prototype.md`（如启用 Figma 原型）

## 依赖与外部能力

部分 skill 依赖以下能力：

- 浏览器自动化能力（如 Chrome DevTools MCP）
- Figma 相关 MCP 能力（`get_design_context` / `get_screenshot` / `use_figma`）
- Node.js 生态工具（如 Playwright、`swagger-typescript-api`）

请确保运行环境中已配置对应工具与权限。

## 插件信息

插件元数据位于 `.claude-plugin/plugin.json`：

- Name: `flow`
- Version: `0.0.1`
- Description: `AI 协作开发工作流`

## 维护建议

- 保持 skill 名称与互相引用路径一致
- 统一文档命名（尤其是 requirements 与 plan 的文件名）
- 新增 skill 时同步更新本 README 的目录和命令清单
