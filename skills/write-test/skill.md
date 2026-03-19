---
name: write-test
description: 编写 E2E 测试 - 根据验收清单使用 /e2e-testing-patterns skill 编写 Playwright E2E 测试
---

# 编写 E2E 测试

根据项目的验收清单，使用 Playwright 编写端到端测试。

## 流程

1. **获取验收标准**
   - 首先查看项目 docs/ 目录下的需求理解文档，找到验收标准部分, 向用户展示你从文档中找到的[验收标准]
   - 如果没有找到文档，询问用户提供验收清单

2. **选择验收条目**
   - **必须使用 AskUserQuestion 工具**，以多选形式让用户选择要编写 E2E 测试的验收条目
   - 将所有验收标准列作选项，设置 `multiSelect: true`
   - 示例格式：
     ```json
     {
       "questions": [
         {
           "question": "请选择要编写 E2E 测试的验收条目（可多选）",
           "header": "选择验收条目",
           "multiSelect": true,
           "options": [
             {"label": "验收条目 1", "description": "..."},
             {"label": "验收条目 2", "description": "..."},
             {"label": "全部", "description": "为所有验收条目编写测试"}
           ]
         }
       ]
     }
     ```

3. **调用 E2E 测试模式 skill**
   - 使用 `/e2e-testing-patterns` skill 来指导测试编写
   - 将用户选中的验收条目作为上下文提供给该 skill

4. **编写测试用例**
   - 按照 Playwright 最佳实践编写测试
   - 每个选中的验收项对应至少一个测试用例
   - 确保测试覆盖核心用户流程

5. **验证测试**
   - 运行测试确保它们能正常执行
   - 向用户展示测试结果
