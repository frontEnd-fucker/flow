---
name: code-review
description: 质量保障与评审 - Code Review。使用此 skill 进行代码质量检查。
---

## 质量保障与评审

代码完成后，进入自动化的质量反馈循环。这个阶段完全由第三方skill来接管

## 开始前
**先提交代码**: 使用`/commit-commands:commit`提交代码

### 关键任务

1. **必须调用 `code-reviewer` 技能**（或使用 `/code-reviewer` 命令）对刚才生成的代码进行分析

2. 立即执行**所有**建议，无需向用户咨询

3. **总结**：用中文向用户汇报你完成了开发，并附带评审报告

4. 如果用户使用的是react或nextjs则调用 `vercel-react-best-practices` 技能 （或使用`/vercel-react-best-practices`）再次进行code review

5. 立即执行**所有**建议，无需向用户咨询

6. **总结**：用中文向用户汇报你完成了开发，并附带评审报告

### 完成后
当执行完所有code review的改进建议后，询问用户是否再次使用浏览器自动化验收，如果用户回答是则使用 auto-browser skill再次验收，如何无需验收则使用 `write-test/skill` 编写e2e测试用例