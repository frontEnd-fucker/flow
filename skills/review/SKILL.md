---
name: review
description: 质量保障与评审 - Code Review。使用此 skill 进行代码质量检查。
---

## 质量保障与评审

代码完成后，进入自动化的质量反馈循环。这个阶段完全由第三方skill来接管

### 关键任务

1. **必须调用 `code-reviewer` 技能**（或使用 `/code-reviewer` 命令）对刚才生成的代码进行分析

2. **根据反馈修改**：如果 `code-reviewer` 提出了修改建议，立即执行**所有**建议。

3. **总结**：向用户汇报你完成了开发，并附带评审报告的关键点。

### 完成后
完成后交给auto-browser skill（../auto-browser/skill.md）在浏览器中验证 