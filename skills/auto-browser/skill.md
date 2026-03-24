---
name: auto-browser
description: 根据验收标准自动在浏览器中验证
---

**开始时：**
1. 读取 `docs/requirements.md` 文件中的验收标准
2. **按页面分组解析验收标准**，识别每个功能页面及其验收项
3. **必须**使用 `AskUserQuestion` tool 让用户选择，采用**页面维度多选模式**：

### 第一步：选择要验证的页面

```json
{
  "questions": [
    {
      "question": "请选择要验证的功能页面",
      "header": "页面选择",
      "multiSelect": true,
      "options": [
        {"label": "登录页", "description": "包含 3 个验收项"},
        {"label": "首页", "description": "包含 5 个验收项"},
        {"label": "用户管理页", "description": "包含 4 个验收项"}
      ]
    }
  ]
}
```

### 第二步：为每个页面选择验收项

**在一个 AskUserQuestion 调用中，为每个选中的页面创建一个 question**：

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
    }
  ]
}
```

⚠️ **重要**：
- 一次 AskUserQuestion 调用可以包含多个 question，每个页面对应一个 question
- 每个 question 的 `multiSelect: true` 允许选择该页面的多个验收项
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

### 验收标准解析格式

从 requirements.md 中按以下格式解析验收标准：

```markdown
### 12.验收标准

#### 登录页
- [ ] 用户可以通过邮箱和密码登录
- [ ] 密码错误时显示提示信息
- [ ] 登录成功后跳转到首页

#### 首页
- [ ] 展示用户统计数据
- [ ] 支持快速导航到各模块
- [ ] 显示系统通知
```

### 验证步骤

1. 使用 chrome-devtools-mcp 在浏览器中验证用户选择的验收标准
2. 有问题自动分析根本原因，并自动修复
3. 修复完成后在浏览器中重新验证

### 完成后

1. 向用户展示验证总结（按页面分组，仅针对已验证的项目）
2. 使用`code-review/skill.md` review代码
