---
name: auto-browser
description: 根据验收标准自动在浏览器中验证
---

**开始时：**
1. 读取 `docs/requirements.md` 文件中的验收标准
2. **按页面分组解析验收标准**：

   **Requirements.md 格式 → Questions JSON 映射关系：**

   ```markdown
   #### 【页面】登录页              ← question.header = "登录页"
   - [ ] 用户可以通过邮箱和密码登录   ← options[0].label
   - [ ] 密码错误时显示提示信息      ← options[1].label
   - [ ] 登录成功后跳转到首页        ← options[2].label

   #### 【页面】首页               ← question.header = "首页"
   - [ ] 展示用户统计数据          ← options[0].label
   - [ ] 支持快速导航到各模块       ← options[1].label

   #### 【页面】用户管理页          ← question.header = "用户管理页"
   - [ ] 展示用户列表             ← options[0].label
   - [ ] 支持分页和搜索            ← options[1].label
   ```

   **映射规则：**
   - 标题中包含 `【页面】` 标记的识别为页面标题（支持任意级别标题 `#` `##` `###` `####` 等）
   - `header` = `【页面】` 后的内容
   - `question` = `【{页面标题}】请选择要验证的验收项`
   - `multiSelect: true`（每个页面可多选）
   - 该页面标题下的每个 `- [ ]` 列表项 → `options` 数组中的一个 `{label: "验收项内容"}`

3. **按上述映射规则生成 AskUserQuestion 调用**：

### 选择要验证的验收项

**在一个 AskUserQuestion 调用中，为每个功能页面创建一个 question**：

```json
{
  "questions": [
    {
      "question": "【登录页】请选择要验证的验收项",
      "header": "登录页",
      "multiSelect": true,
      "options": [
        {"label": "用户可以通过邮箱和密码登录"},
        {"label": "密码错误时显示提示信息"},
        {"label": "登录成功后跳转到首页"}
      ]
    },
    {
      "question": "【首页】请选择要验证的验收项",
      "header": "首页",
      "multiSelect": true,
      "options": [
        {"label": "展示用户统计数据"},
        {"label": "支持快速导航到各模块"},
        {"label": "显示系统通知"}
      ]
    },
    {
      "question": "【用户管理页】请选择要验证的验收项",
      "header": "用户管理页",
      "multiSelect": true,
      "options": [
        {"label": "展示用户列表"},
        {"label": "支持分页和搜索"},
        {"label": "可以禁用/启用用户"}
      ]
    }
  ]
}
```

⚠️ **重要**：
- 一次 AskUserQuestion 调用包含多个 question，每个功能页面对应一个 question
- 每个 question 的 `multiSelect: true` 允许选择该页面的多个验收项
- 用户可以为每个页面独立选择要验证的验收项
- 禁止直接输出文本列表

**用户选择后：**
向用户确认，按页面分组展示：

```
我来在浏览器中验证以下验收标准：

【登录页】
- [ ] 用户可以通过邮箱和密码登录
- [ ] 密码错误时显示提示信息

【首页】
- [ ] 展示用户统计数据
- [ ] 支持快速导航到各模块
```

### 验证步骤

1. 使用 chrome-devtools-mcp 在浏览器中验证用户选择的验收标准
2. 有问题自动分析根本原因，并自动修复
3. 修复完成后在浏览器中重新验证

### 完成后

1. 向用户展示验证总结（按页面分组，仅针对已验证的项目）
2. 使用`code-review/skill.md` review代码
