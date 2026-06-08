# Flow

Flow 是一个 AI 协作开发工作流插件仓库，同时支持 Claude Code 和 Codex 客户端。

它提供从需求分析、计划拆解、编码执行、浏览器验收到代码评审、E2E 测试和缺陷修复的一组可复用 skills。插件本体位于 `plugins/flow/`，仓库根目录提供 Claude Code 与 Codex 各自的 marketplace 清单。

## Repository Layout

```text
flow/
├── .claude-plugin/
│   └── marketplace.json
├── .agents/
│   └── plugins/
│       └── marketplace.json
├── plugins/
│   └── flow/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── .codex-plugin/
│       │   └── plugin.json
│       ├── skills/
│       └── README.md
└── docs/
    └── dual-compatible-plugin-plan.md
```

## Claude Code

### Install From GitHub

```text
/plugin marketplace add frontEnd-fucker/flow
/plugin install flow@flow
/reload-plugins
```

### Install Locally

```text
/plugin marketplace add /Users/a1/development/flow
/plugin install flow@flow
/reload-plugins
```

### Usage

```text
/flow:requirements
/flow:planning
/flow:coding
/flow:bugfix
```

## Codex

### Install From GitHub

```bash
codex plugin marketplace add frontEnd-fucker/flow --ref master
codex plugin add flow@flow
```

For a pinned release:

```bash
codex plugin marketplace add frontEnd-fucker/flow --ref v0.0.1
codex plugin add flow@flow
```

### Install Locally

```bash
codex plugin marketplace add /Users/a1/development/flow
codex plugin add flow@flow
```

### Usage

Codex uses natural-language skill triggering. Examples:

```text
使用 flow 的 requirements skill 帮我做需求分析
使用 flow 的 planning skill 生成执行计划
使用 flow 的 bugfix 流程处理这个缺陷
```

Open a new Codex thread after installing or updating the plugin so the updated skills are loaded.

## Skills

- `requirements`: 需求理解与技术选型
- `planning`: 任务拆解与执行计划
- `coding`: 核心开发阶段
- `auto-browser`: 浏览器自动化验收
- `code-review`: 代码质量评审
- `write-test`: Playwright E2E 测试编写
- `bugfix`: 缺陷工单处理
- `api-connect`: Mock API 转真实 API
- `figma-prototype`: Figma 转原型页面
- `figma-desktop-prototype`: Figma Desktop MCP 转原型页面
- `figma-assets-downloader`: Figma 资源下载
- `figma-review`: 设计稿视觉比对
- `hello`: 示例技能

## Validation

Validate the Codex plugin manifest:

```bash
python3 /Users/a1/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py /Users/a1/development/flow/plugins/flow
```

Validate the Claude Code marketplace and plugin:

```bash
claude plugin validate /Users/a1/development/flow
claude plugin validate /Users/a1/development/flow/plugins/flow
```

## Release

Before publishing, validate both plugin surfaces, commit the changes, push `master`, then create and push a release tag:

```bash
git push origin master
git tag v0.0.1
git push origin v0.0.1
```
