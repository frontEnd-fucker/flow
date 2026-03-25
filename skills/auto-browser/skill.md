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

**⚠️ 重要限制**：`AskUserQuestion` 工具每个 question 的 `options` 最多只能有 **4 个选项**。当一个页面的验收项超过 4 个时，需要将该页面拆分为多个 question。

**在一个 AskUserQuestion 调用中，为每个功能页面创建一个或多个 question**：

| 页面验收项数量 | 处理方式 |
|-------------|---------|
| ≤ 4 个 | 创建一个 question |
| > 4 个 | 拆分为多个 question，每个最多 4 个选项，用 "1/2", "2/2" 区分 |

**示例 1：单个页面验收项 ≤ 4 个**（创建一个 question）
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
    }
  ]
}
```

**示例 2：单个页面验收项 > 4 个**（拆分为多个 question）
```json
{
  "questions": [
    {
      "question": "【登录页 - 1/2】请选择要验证的验收项",
      "header": "登录页",
      "multiSelect": true,
      "options": [
        {"label": "用户可以通过邮箱和密码登录"},
        {"label": "密码错误时显示提示信息"},
        {"label": "登录成功后跳转到首页"},
        {"label": "支持记住密码功能"}
      ]
    },
    {
      "question": "【登录页 - 2/2】请选择要验证的验收项",
      "header": "登录页",
      "multiSelect": true,
      "options": [
        {"label": "支持第三方登录"},
        {"label": "显示登录次数统计"}
      ]
    },
    {
      "question": "【首页】请选择要验证的验收项",
      "header": "首页",
      "multiSelect": true,
      "options": [
        {"label": "展示用户统计数据"},
        {"label": "支持快速导航到各模块"}
      ]
    }
  ]
}
```

**规则总结**：
- 每个 `options` 数组最多 4 个选项
- 拆分后的 question 使用相同 `header`，`question` 文本添加 "- 1/2", "- 2/2" 序号
- 一次 AskUserQuestion 调用可包含多个 question（多个页面或拆分的子页面）
- 禁止直接输出文本列表让用户选择

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
