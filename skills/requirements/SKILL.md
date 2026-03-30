---
name: requirements
description: 需求理解阶段 - 理解需求、技术选型、存量代码分析。使用此 skill 进行项目或功能开发的需求分析阶段。
---

# 需求理解

通过多轮对话协作，将需求转化为完整的技术方案。

每次只提出**一个问题**来完善方案，当需求内容已经完全理解清楚，向用户展示完整方案，询问用户是否有异议，当用户没有异议时写下文档。

## 约束规则（必须遵守）

**交互模式：单问题递进**
- 每次对话只能向用户提出**一个**问题（技术方案选型除外，可一次性提问所有维度）
- 必须得到用户回答后，才能提出下一个问题
- 技术选型问题必须使用选项按钮，以 options 数组形式提供可点击选项
- 禁止在一次回复中提出多个问题（技术方案选型除外）


## 清单

为下面每一项创建一个task，按顺序完成每一项

1. 了解需求 
- 询问用户提供需求文档链接或直接描述需求
- “请描述一下你的项目需求：这个项目要解决什么问题？ （或者如果你有需求文档/PRD链接，或者有本地word文件，也可以直接分享）”
- 如果用户提供需求文档/PRD链接执行以下步骤
  a. 开始前向用户宣告”我将会通过 lark-cli 获取你提供的飞书文档链接内容”
  b. 使用 `lark-cli docs +fetch --doc` 命令获取飞书文档内容，**确保完整获取文档中的所有内容**并总结成需求内容，需求内容要求**详细，包含需求文档中的每个细小功能点**。
  c. 将总结的内容写到`prd-sumerize.md`储存到`/docs`文件夹下
  d. 将总结的内容展示给用户，等用户确认。
  e. 用户确认后，这部分<需求文档总结>内容需要添加到后续生成的需求文档

2. 询问用户是否有figma设计稿，没有则跳过此步骤，有的话让用户提供链接，提示用户每个页面提供一个figma链接，可以在figma dev模式下，按住shift键同时选中多个页面，然后点击”copy example”。若用户提供了figma链接则执行以下步骤：
  a. 调用 `/figma-prototype` skill 批量生成原型页面
  b. 原型页面将保存到 `/prototype` 文件夹，包含：
     - 各页面的 HTML 原型文件（使用 HTML + TailwindCSS）
     - 图标资源下载到 `/public/icons/`
     - 共享的 `tailwind.config.js` 配置文件
     - 导航入口页面 `/prototype/index.html`
     - 生成报告 `/prototype/prototype.md`

3. 技术方案选型
**必须使用 `AskUserQuestion` 内置工具，一次性提问所有技术选型维度。**

**工具调用格式示例：**
```json
{
  "questions": [
    {
      "question": "站点类型？",
      "header": "站点类型",
      "options": [
        {"label": "PC端", "value": "pc"},
        {"label": "移动端", "value": "mobile"},
        {"label": "自适应", "value": "responsive"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "选用什么前端框架？",
      "header": "框架",
      "options": [
        {"label": "React", "value": "react"},
        {"label": "Vue", "value": "vue"},
        {"label": "Next.js", "value": "nextjs"},
        {"label": "Nuxt", "value": "nuxt"},
        {"label": "原生 JS", "value": "vanilla"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "开发语言？",
      "header": "语言",
      "options": [
        {"label": "TypeScript", "value": "typescript"},
        {"label": "JavaScript", "value": "javascript"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "样式方案？",
      "header": "样式方案",
      "options": [
        {"label": "Tailwind CSS", "value": "tailwind"},
        {"label": "CSS Modules", "value": "css-modules"},
        {"label": "Styled-components", "value": "styled-components"},
        {"label": "Less", "value": "less"},
        {"label": "Sass/Scss", "value": "sass"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "状态管理方案？",
      "header": "状态管理",
      "options": [
        {"label": "Redux", "value": "redux"},
        {"label": "Zustand", "value": "zustand"},
        {"label": "Pinia", "value": "pinia"},
        {"label": "Vuex", "value": "vuex"},
        {"label": "Context API", "value": "context-api"},
        {"label": "Jotai", "value": "jotai"},
        {"label": "MobX", "value": "mobx"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "网络请求工具？",
      "header": "网络请求",
      "options": [
        {"label": "Axios", "value": "axios"},
        {"label": "Fetch", "value": "fetch"},
        {"label": "ky", "value": "ky"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "异步数据管理？",
      "header": "异步数据管理",
      "options": [
        {"label": "TanStack Query (React Query)", "value": "tanstack-query"},
        {"label": "SWR", "value": "swr"},
        {"label": "Redux Toolkit Query", "value": "rtk-query"},
        {"label": "Vue Query", "value": "vue-query"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "Mock数据方案？",
      "header": "Mock数据",
      "options": [
        {"label": "不需要Mock数据", "value": "none"},
        {"label": "本地Mock数据", "value": "local-mock"},
        {"label": "MSW (Mock Service Worker)", "value": "msw"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "路由方案？",
      "header": "路由",
      "options": [
        {"label": "React Router", "value": "react-router"},
        {"label": "Next.js 内置路由", "value": "nextjs-router"},
        {"label": "Vue Router", "value": "vue-router"},
        {"label": "TanStack Router", "value": "tanstack-router"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "构建工具？",
      "header": "构建工具",
      "options": [
        {"label": "Vite", "value": "vite"},
        {"label": "Webpack", "value": "webpack"},
        {"label": "Bun", "value": "bun"},
        {"label": "Rollup", "value": "rollup"},
        {"label": "Parcel", "value": "parcel"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "组件库？",
      "header": "组件库",
      "options": [
        {"label": "shadcn/ui", "value": "shadcn"},
        {"label": "Ant Design", "value": "antd"},
        {"label": "Ant Design Mobile", "value": "antd-mobile"},
        {"label": "Element Plus", "value": "element-plus"},
        {"label": "Element UI", "value": "element-ui"},
        {"label": "Chakra UI", "value": "chakra-ui"},
        {"label": "MUI (Material-UI)", "value": "mui"},
        {"label": "不需要组件库", "value": "none"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    },
    {
      "question": "包管理工具？",
      "header": "包管理工具",
      "options": [
        {"label": "pnpm", "value": "pnpm"},
        {"label": "npm", "value": "npm"},
        {"label": "yarn", "value": "yarn"},
        {"label": "Bun", "value": "bun"},
        {"label": "其他（请说明）", "value": "other"}
      ]
    }
  ]
}
```

**关键约束：**
1. **一次性提问所有维度** - 使用 `AskUserQuestion` 一次提问所有 12 个技术选型维度
2. **必须使用选项按钮** - 每个维度以 `options` 数组形式提供可点击选项
3. **包含"其他"选项** - 每个维度都必须有"其他"选项，允许用户自定义

**完成技术选型后必须询问：**
- 在用户完成上述技术选型后，必须向用户提问："是否还需要指定其他 npm 包？例如：图表库、日期处理库、表单验证库等。"
- 如果用户指定了其他 npm 包，将这些包记录在需求理解文档的「技术选型」部分



4. 存量代码协作（如适用）
如果是在既有代码库中工作：
- **探索是否有原型页面**

- **先行探索**: 必须首先探索并理解当前的系统结构
  - 使用 Glob/Read 工具理解目录结构
  - 查找相关文件，理解现有逻辑
  - 不要在未理解现有代码的情况下提出修改建议

- **遵循既有模式**: 原则上应遵循代码库中已有的设计模式
  - 检查现有的命名规范
  - 遵循现有的目录结构约定
  - 保持代码风格一致

- **针对性改进**: 仅当现有代码的问题直接影响当前任务时，才包含改进步骤
  - 例如：文件过大、职责纠缠
  - 不要做过度重构

5. 输出：需求理解文档到`docs/`文件夹下

输出需求理解文档，包含：

```
## 需求理解文档

### 1. 需求概述
[简短描述要解决的问题]如果有需求文档总结添加<需求文档总结>

### 2. 功能规格
[列出各项功能点]

### 3. 技术选型
[包含前端框架，开发语言，样式方案，状态管理，储存方案，构建工具等，需要列出对应的版本, **优先使用最新稳定版本**]

### 4. 目录结构
[建议的目录结构]
**约束**
- **组件拆分必须要参考/prototype中的原型页面，参考/prototype/prototype.md了解页面结构，样式也要复原原型页面的设计**
- 页面组件（路由组件）需要标明所使用的原型html和figma信息，如果没有则标明没有使用原型html和figma信息
  示例：
  │   ├── bookshelf/
  │   │   └── index.tsx              # 书架页 页面原型:prototype/bookshelf.html figma：https://www.figma.com/design/cf1NMdWsFBbFuLfVslAwNe/Infrastructure-Projects----APP?node-id=2763-19984&m=dev
  │   ├── detail/
  │   │   └── index.tsx              # 书籍详情页 页面原型:prototype/detail.html figma：https://www.figma.com/design/cf1NMdWsFBbFuLfVslAwNe/Infrastructure-Projects----APP?node-id=2763-20457&m=dev

### 5. 组件设计
- 样式参考原型页面：`/prototype/prototype.md` 及对应 原型HTML文件
- 核心组件: [列出主要组件，包括组件的props]
- 数据流: [组件间数据传递方式]

### 6. 设计规范
- [docs/prototype/prototype.md 文件中的设计规范]

### 7.通用逻辑设计
- utils: [列出通用util]
- hooks: [列出通用hook]

### 8. 数据模型
[通常为typescript类型]

### 9. 数据层
[接口数据获取逻辑，store状态管理]

### 10. 存量代码分析（如适用）
- 现有文件: [相关文件列表]
- 需要修改: [需要改动的文件]
- 遵循模式: [已有的设计模式]

### 11. 风险与约束
[识别的潜在风险和限制]

### 12.验收标准
按功能页面分组编写验收标准，使用 `【页面】` 标记识别页面：

```markdown
#### 【页面】首页
- [ ] 瀑布流双列布局，卡片高度根据内容自适应
- [ ] 封面图加载失败显示占位图
- [ ] 播放量正确格式化（237 / 2.4k / 1.2w）
- [ ] 话数标签显示（"99话" / "56章全"）

#### 【页面】详情页
- [ ] 展示作品标题、作者、封面图
- [ ] 支持收藏/取消收藏功能
- [ ] 显示章节列表并支持跳转
```

**编写规范：**
- 页面标题使用 `【页面】` 标记（如 `#### 【页面】首页`）
- 每个页面下的 `- [ ]` 列表项为该页面的验收项
- 支持任意级别标题（`#` `##` `###` `####` 等），只要包含 `【页面】` 标记即可
```

## 完成后

- 向用户展示需求理解文档
- 询问用户是否有补充或修正
- 进入规划阶段 ./planning/skill.md
