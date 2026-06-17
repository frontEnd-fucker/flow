# 执行计划：将 `flow` 发布为 Codex + Claude Code 双兼容插件

## Summary

在当前仓库 `/Users/a1/development/flow` 中新增 `docs/` 文档目录，保存双兼容发布方案；随后把仓库改造成同时支持 Codex 和 Claude Code 的 GitHub marketplace 仓库。最终发布到 `frontEnd-fucker/flow` 后，Claude Code 可通过 `/plugin marketplace add frontEnd-fucker/flow` 安装，Codex 可通过 `codex plugin marketplace add frontEnd-fucker/flow --ref master` 安装。

## Key Changes

- 新建文档：
  - 创建 `docs/dual-compatible-plugin-plan.md`
  - 写入本方案，包含目标结构、安装方式、验证方式、发布方式
- 调整仓库结构：
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
  ├── docs/
  │   └── dual-compatible-plugin-plan.md
  └── README.md
  ```
- 移动现有插件内容：
  - `skills/` 移动到 `plugins/flow/skills/`
  - `.claude-plugin/plugin.json` 移动到 `plugins/flow/.claude-plugin/plugin.json`
  - 当前根目录 `README.md` 的插件说明内容保留到 `plugins/flow/README.md`
- 新增 Claude Code marketplace：
  - 文件：`.claude-plugin/marketplace.json`
  - marketplace 名称：`flow`
  - 插件 source：`./plugins/flow`
- 新增 Codex marketplace：
  - 文件：`.agents/plugins/marketplace.json`
  - marketplace 名称：`flow`
  - 插件 source path：`./plugins/flow`
- 新增 Codex manifest：
  - 文件：`plugins/flow/.codex-plugin/plugin.json`
  - 指向 `./skills/`
  - 补齐 Codex 校验必需的 `interface` 元数据
- 统一 skill 文件名：
  - 所有 `skill.md` 改名为 `SKILL.md`
  - 已有 `SKILL.md` 保持不变
- 更新根目录 `README.md`：
  - 改为 GitHub 安装说明
  - 分别列出 Claude Code 和 Codex 的安装命令
  - 说明两端调用方式差异

## Execution Steps

1. 创建 `docs/dual-compatible-plugin-plan.md`，写入完整双兼容方案。
2. 创建 `plugins/flow/`。
3. 移动 `skills/` 到 `plugins/flow/skills/`。
4. 移动 `.claude-plugin/plugin.json` 到 `plugins/flow/.claude-plugin/plugin.json`。
5. 将旧根 README 的插件说明复制或迁移到 `plugins/flow/README.md`。
6. 在根目录重新创建 `.claude-plugin/marketplace.json`。
7. 创建 `.agents/plugins/marketplace.json`。
8. 创建 `plugins/flow/.codex-plugin/plugin.json`。
9. 将所有 `plugins/flow/skills/*/skill.md` 统一重命名为 `SKILL.md`。
10. 更新根目录 `README.md` 为双平台安装文档。
11. 运行本地校验命令。
12. 测试 Codex 本地安装。
13. 测试 Claude Code 本地安装。
14. 提交并推送到 GitHub。
15. 打 tag `v0.0.1` 并推送 tag。

## Install UX

Claude Code，GitHub 安装：

```text
/plugin marketplace add frontEnd-fucker/flow
/plugin install flow@flow
/reload-plugins
```

Claude Code 调用示例：

```text
/flow:requirements
/flow:planning
/flow:coding
```

Codex，GitHub 安装：

```bash
codex plugin marketplace add frontEnd-fucker/flow --ref master
codex plugin add flow@flow
```

Codex 调用示例：

```text
使用 flow 的 requirements skill 帮我做需求分析
使用 flow 的 planning skill 生成执行计划
```

稳定版本安装：

```bash
codex plugin marketplace add frontEnd-fucker/flow --ref v0.0.1
codex plugin add flow@flow
```

## Test Plan

- Codex manifest 校验：
  ```bash
  python3 /Users/a1/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py /Users/a1/development/flow/plugins/flow
  ```
- Codex 本地 marketplace 验证：
  ```bash
  codex plugin marketplace add /Users/a1/development/flow
  codex plugin list --available --json
  codex plugin add flow@flow
  codex plugin list --json
  ```
- Claude Code 本地验证：
  ```text
  /plugin marketplace add /Users/a1/development/flow
  /plugin install flow@flow
  /reload-plugins
  /flow:hello
  ```
- GitHub 发布验证：
  ```bash
  git status --short
  git push origin master
  git tag v0.0.1
  git push origin v0.0.1
  ```
- 功能验证：
  - Claude Code 中 `/flow:requirements` 能触发 requirements skill
  - Claude Code 中 `/flow:planning` 能触发 planning skill
  - Codex 新 thread 中自然语言提到 `flow requirements skill` 能触发对应 skill
  - Figma 相关 skill 只验证可加载，不在安装阶段要求外部 Figma MCP 必须可用

## Assumptions

- Claude Code 侧以 marketplace 安装为主，不要求仓库根目录继续支持直接作为插件根使用。
- Codex 和 Claude Code 共用 `plugins/flow/skills/`。
- GitHub remote 继续使用 `frontEnd-fucker/flow`。
- 默认分支继续使用 `master`。
- Marketplace 名称和插件名都使用 `flow`，因此安装选择器为 `flow@flow`。
- 不使用 symlink，避免 GitHub、Windows、Codex 缓存安装、Claude Code marketplace 安装时出现路径差异。
- 正式发布时同步更新：
  - `plugins/flow/.claude-plugin/plugin.json`
  - `plugins/flow/.codex-plugin/plugin.json`
  - `.claude-plugin/marketplace.json`
