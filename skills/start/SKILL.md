---
name: start
description: AI 协作开发工作流 - 引导完整的 5 步标准化开发流程。使用此 skill 开始一个新项目或重大功能开发，它会引导你完成从需求理解到缺陷修复的完整流程。
---

This skill orchestrates the complete AI collaboration workflow, guiding you through 5 standardized steps:

1. **需求理解 (Requirements)** - 需求文档 prompt 化、技术选型、存量代码分析
2. **规划阶段 (Planning)** - 任务拆解、范围检查、目录结构设计、人机共识
3. **核心开发 (Development)** - TDD、原子化提交、git worktree 隔离、Chrome DevTool MCP
4. **质量保障 (QA & Review)** - Code Review、自动化测试
5. **缺陷处理 (Bugfix)** - 工单分析、单一假设、最小化测试

## 启动工作流

当用户要开始一个新项目或重大功能开发时，使用 `/start` 命令或直接调用此 skill。

## 工作流流程

### Step 1/5: 需求理解
调用 `requirements` skill:
- 解析需求文档为初始化 prompt
- 技术方案选型（架构设计、组件定义、数据层设计）
- 探索现有代码库结构（如适用）
- 输出：需求理解文档

**完成后询问用户确认，然后进入 Step 2**

### Step 2/5: 规划阶段
调用 `planning` skill:
- 任务拆解（使用 `[ ] Step N` 格式）
- 范围检查（确保计划涵盖所有子功能）
- 目录结构设计
- 输出：执行计划（需用户 Review & Approve）

**必须获得用户 "Review & Approve" 才能进入 Step 3**

### Step 3/5: 核心开发
调用 `development` skill:
- 遵循 TDD 原则（先写测试）
- 使用 git worktree 隔离开发分支
- 利用 Chrome DevTool MCP 进行实时调试
- 原子化提交

**每个原子任务完成后确认，继续下一个直到全部完成，然后进入 Step 4**

### Step 4/5: 质量保障
调用 `qa-review` skill:
- 执行 Code Review（参考 Google/React 规范）
- 引入技术框架 best-practices
- 运行 Playwright e2e 测试
- 输出：Review 报告

**如有问题，返回 Step 3 修复；如无问题，进入 Step 5**

### Step 5/5: 缺陷处理
调用 `bugfix` skill:
- 工单分析（明确假设）
- 单一假设验证（每次只验证一个变量）
- 最小化改动验证假设
- 失败时撤销并形成新假设

## 进度追踪

在每一步开始时，显示进度：
- Step X/5: [步骤名称]
- 进度：[████████░░] 40%

## 关键原则

1. **人机共识**: Step 2 完成后必须获得用户批准才能开始开发
2. **TDD 优先**: Step 3 必须先写测试再写实现
3. **原子化提交**: 每个子任务单独提交
4. **单一假设**: Step 5 每次只验证一个变量
5. **最小化改动**: 只做验证假设所需的最小改动

## 可单独调用的子 Skill

如需单独使用某一步骤，可直接调用：
- `/requirements` - 需求理解
- `/planning` - 规划阶段
- `/development` - 核心开发
- `/qa-review` - 质量保障
- `/bugfix` - 缺陷处理
