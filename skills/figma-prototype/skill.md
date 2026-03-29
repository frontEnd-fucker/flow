---
name: figma-prototype
description: 根据多个 Figma 设计稿批量生成完整的 HTML 原型页面，自动下载图标资源，提取设计 token 生成统一的 Tailwind 配置
---

## 概述

本 skill 支持批量处理多个 Figma 设计稿，生成完整的 HTML 原型页面系统，包含以下核心功能：

1. **批量处理多页面**：支持一次输入多个 Figma URL，并行处理
2. **共享设计 Token**：同 Figma 文件的页面共享 design token，生成统一的 Tailwind 配置
3. **批量下载资源**：自动下载所有页面的 icon、图片等资源，智能去重
4. **汇总报告**：生成完整报告，包含所有页面的 token、资源、路径信息

---

## 执行流程

**严格按照 skill 流程的每一步执行，不要跳过任何步骤，完成一步确认后再进行下一步**

### 第 1 步：收集 Figma 链接（支持多 URL）

**询问用户：**

```
请提供 Figma 设计稿链接（支持多个）：
- 每行一个链接，或用逗号分隔
- 在 Figma Dev 模式下，按住 Shift 可选中多个页面同时复制链接
```

**接收用户输入后，解析所有 URL：**
从用户提供的 Figma URL 中提取参数：

```
https://figma.com/design/AbCdEf123/ProjectName?node-id=1-2
                    └─────┬─────┘              └────┬────┘
                     fileKey                     nodeId
```

| 参数 | 提取规则 | 示例 |
|------|---------|------|
| **fileKey** | URL 中 `/design/` 后的第一段 | `AbCdEf123` |
| **nodeId** | `node-id` 参数的值，将 `-` 替换为 `:` | `1:2` |

**解析结果示例：**

| # | URL | fileKey | nodeId | 状态 |
|---|-----|---------|--------|------|
| 1 | `https://figma.com/design/AbCdEf123/Project?node-id=1-2` | `AbCdEf123` | `1:2` | ✅ |
| 2 | `https://figma.com/design/AbCdEf123/Project?node-id=3-4` | `AbCdEf123` | `3:4` | ✅ |
| 3 | `https://figma.com/design/AbCdEf123/Project?node-id=5-6` | `AbCdEf123` | `5:6` | ✅ |

---

### 第 2 步：获取设计 Token

a.带上获取到的fileKey和nodeId作为参数，并行调用`get_design_context`.
  **调用规范**:
  必须传入以下参数：
  - fileKey: 必须是字符串（例如 "Abc123456"）
  - nodeId: 必须是字符串（例如 "10:140"）

  调用示例：
  `get_design_context(fileKey="实际的fileKey", nodeId="实际的nodeId")`

b.从{返回的context信息}中提取颜色值、字体大小等作为 token。

| Token 类型 | 用途 | Tailwind 映射 |
|-----------|------|--------------|
| `color` | 颜色变量 | `colors` 主题 |
| `number` | 间距、圆角、字体大小 | `spacing`, `borderRadius`, `fontSize` |
| `string` | 字体族 | `fontFamily` |

**Token 命名规范转换：**
- Figma: `color/primary/500` → Tailwind: `primary-500`
- Figma: `spacing/medium` → Tailwind: `spacing-medium`
- Figma: `font/size/heading-1` → Tailwind: `font-heading-1`

---

### 第 3 步：生成统一的 Tailwind 配置

#### 3.1 创建 tailwind.config.js

基于获取的 design token 生成共享配置文件：

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./pages/**/*.html', './index.html'],
  theme: {
    extend: {
      // 从 Figma 文件提取的颜色 Token
      colors: {
        // 主色板
        'primary-50': '#f0f9ff',
        'primary-100': '#e0f2fe',
        'primary-500': '#0ea5e9',
        'primary-600': '#0284c7',
        'primary-900': '#0c4a6e',
        // 中性色
        'neutral-100': '#f5f5f5',
        'neutral-200': '#e5e5e5',
        'neutral-800': '#262626',
        'neutral-900': '#171717',
        // ... 其他颜色
      },
      // 间距 Token
      spacing: {
        'xs': '4px',
        'sm': '8px',
        'md': '16px',
        'lg': '24px',
        'xl': '32px',
        '2xl': '48px',
      },
      // 圆角 Token
      borderRadius: {
        'sm': '4px',
        'md': '8px',
        'lg': '12px',
        'xl': '16px',
        'full': '9999px',
      },
      // 字体大小 Token
      fontSize: {
        'heading-1': ['32px', { lineHeight: '40px', fontWeight: '700' }],
        'heading-2': ['24px', { lineHeight: '32px', fontWeight: '600' }],
        'heading-3': ['20px', { lineHeight: '28px', fontWeight: '600' }],
        'body': ['16px', { lineHeight: '24px', fontWeight: '400' }],
        'caption': ['12px', { lineHeight: '16px', fontWeight: '400' }],
      },
      // 字体族 Token
      fontFamily: {
        'sans': ['Inter', 'system-ui', 'sans-serif'],
        'display': ['Poppins', 'sans-serif'],
      },
    },
  },
  plugins: [],
}
```

---

### 第 4 步：批量下载 Figma 资源（icon，logo），**绝不能跳过此步骤**

#### 4.1 收集资源链接

**从{返回的context信息}中提取资源链接，筛选图标资源并去重：**

**执行步骤：**
a. **筛选图标资源，确保只包含图标资源，不包含图片资源**

b. **按 URL 去重**
   - 使用资源 URL 作为唯一标识
   - 相同 URL 的资源只保留一条记录

c. **输出资源链接列表**
   - 总资源数量和去重后的唯一资源数量
   - 生成包含下载链接的结构化资源列表

**输出格式：**

去重后的 URL 列表（仅包含下载链接）：

```
https://s3.figma.com/.../icon-search.svg?...
https://s3.figma.com/.../icon-close.svg?...
https://s3.figma.com/.../logo.png?...
...
```

#### 4.3 **输出摘要信息：**
```
✅ 资源链接收集完成

📊 统计：
   - 总资源数：12
   - 去重后：10（2个重复）
   - 通用图标：5
   - 页面专属：7

🔗 下载链接已准备，共 10 个唯一 URL
```

#### 4.4 下载图标资源

通过 Figma MCP 服务内置的 assets endpoint 下载所有图标资源：
步骤：
a. 创建输出目录：`/public/icons`
b. 遍历收集的图标资源列表
c. 对每个资源的下载 URL，调用下载工具指定输出路径为 `./public/icons/文件名.svg`，参考以下示例。

**下载示例**
```
# 第一步：强制创建物理目录
mkdir -p ./public/icons

# 第二步：使用完整路径进行原子化下载
curl -sL "https://figma-alpha.s3..." -o "./public/icons/icon-arrow-left.svg"
curl -sL "https://figma-alpha.s3..." -o "./public/icons/icon-search.svg"
```

---

### 第 5 步：生成图标入口文件，**绝不能跳过此步骤**

生成图片的引用入口文件，该文件需要配合`SVGR插件`使用

**文件示例**
`**/components/icons/index.ts`

```typescript
// 通用图标
export { default as IconSearch } from './common/IconSearch';
export { default as IconClose } from './common/IconClose';
export { default as IconArrowRight } from './common/IconArrowRight';

// 登录页图标
export { default as IconUser } from './login/IconUser';
export { default as IconLock } from './login/IconLock';

// 首页图标
export { default as IconMenu } from './home/IconMenu';
export { default as IconNotification } from './home/IconNotification';
```

---

### 第 6 步：批量生成原型页面,保存到项目`/prototype`文件夹

**开始前：** 开始前先必须检查第4步和第5步是否已经完成，如未完成则重新执行第4步或第5步，直到它们都完成
**实现原则：** 不使用inline svg icon，使用下载好的资源


#### 6.1 使用figma mcp实现用户提供的页面，为每个页面生成一个 HTML（使用html + tainwindcss，保存到项目`/prototype`文件夹

#### 6.2 页面 HTML 模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{页面名称} - 原型</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- 共享配置 -->
  <script src="../../tailwind.config.js"></script>
  <!-- 项目样式 -->
  <link rel="stylesheet" href="../../styles/tailwind.css">
</head>
<body class="bg-neutral-50">
  <!-- 页面内容 -->

  <!-- 图标引用示例 -->
  <img src="../../assets/icons/common/icon-search.svg" class="w-5 h-5" alt="">
  <img src="../../assets/icons/login/icon-user.svg" class="w-6 h-6" alt="">
</body>
</html>
```

---

### 第 7 步：生成导航入口页面

创建 `/prototype/index.html` 作为所有原型页面的导航入口：

#### 7.1 导航页内容

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>原型导航</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="./tailwind.config.js"></script>
  <link rel="stylesheet" href="./styles/tailwind.css">
</head>
<body class="bg-neutral-50 min-h-screen">
  <div class="max-w-4xl mx-auto py-12 px-6">
    <h1 class="text-3xl font-bold text-neutral-900 mb-2">Figma 原型</h1>
    <p class="text-neutral-600 mb-8">共生成 {N} 个页面</p>

    <div class="grid gap-4">
      <!-- 页面卡片 -->
      <a href="./pages/login/index.html"
         class="block bg-white rounded-lg shadow-sm border border-neutral-200 p-6 hover:shadow-md transition-shadow">
        <div class="flex items-center justify-between">
          <div>
            <h2 class="text-lg font-semibold text-neutral-900">登录页</h2>
            <p class="text-sm text-neutral-500 mt-1">./pages/login/index.html</p>
          </div>
          <span class="text-neutral-400">→</span>
        </div>
      </a>

      <a href="./pages/home/index.html"
         class="block bg-white rounded-lg shadow-sm border border-neutral-200 p-6 hover:shadow-md transition-shadow">
        <div class="flex items-center justify-between">
          <div>
            <h2 class="text-lg font-semibold text-neutral-900">首页</h2>
            <p class="text-sm text-neutral-500 mt-1">./pages/home/index.html</p>
          </div>
          <span class="text-neutral-400">→</span>
        </div>
      </a>
    </div>

    <div class="mt-12 pt-8 border-t border-neutral-200">
      <h3 class="text-sm font-medium text-neutral-900 mb-4">设计 Token</h3>
      <div class="flex flex-wrap gap-2">
        <span class="px-3 py-1 bg-primary-100 text-primary-700 rounded-full text-sm">primary-500</span>
        <span class="px-3 py-1 bg-neutral-100 text-neutral-700 rounded-full text-sm">neutral-100</span>
      </div>
    </div>
  </div>
</body>
</html>
```

#### 7.2 生成规则

1. **页面标题**：显示 "Figma 原型 - {项目名}"
2. **页面列表**：
   - 每个页面一个卡片
   - 显示页面名称和文件路径
   - 点击卡片跳转到对应页面
   - 按页面顺序排列
3. **设计 Token 预览**：
   - 显示主要颜色 token 的色块
   - 可选：显示字体、间距等关键 token
4. **资源统计**：
   - 图标总数
   - 图片总数

---

### 第 8 步：生成汇总报告

#### 8.1 创建 /prototype/prototype.md

```markdown
# Figma 原型生成报告

## 概述

- **生成时间**: 2026-03-27 10:30:00
- **Figma 源文件**: {fileKey}
- **生成页面**: {N} 个页面

## 处理摘要

| # | 页面名称 | 状态 |
|---|---------|------|
| 1 | 登录页 | ✅ 成功 |
| 2 | 首页 | ✅ 成功 |
| 3 | 设置页 | ✅ 成功 |

## 设计 Token

### 颜色系统

| Token | 值 |
|-------|-----|
| primary-500 | #0ea5e9 |
| primary-600 | #0284c7 |
| neutral-100 | #f5f5f5 |
| neutral-900 | #171717 |

### 间距系统

| Token | 值 |
|-------|-----|
| xs | 4px |
| sm | 8px |
| md | 16px |
| lg | 24px |
| xl | 32px |

## 下载的资源

### 图标统计

| 位置 | 数量 |
|------|------|
| common/ | 5 个 |
| login/ | 3 个 |
| home/ | 4 个 |
| **总计** | **12 个** |

### 图片统计

| 位置 | 数量 |
|------|------|
| login/ | 1 个 |
| home/ | 3 个 |
| **总计** | **4 个** |

## 生成的页面

### 登录页 (`/pages/login/index.html`)
- **Figma**: https://figma.com/design/{fileKey}/Project?node-id=1-2
- **图标**: icon-user, icon-lock
- **图片**: bg-pattern.png

### 首页 (`/pages/home/index.html`)
- **Figma**: https://figma.com/design/{fileKey}/Project?node-id=3-4
- **图标**: icon-menu, icon-notification, icon-search
- **图片**: hero-banner.png, feature-1.png
```
---

## 完成后
- 检查是否有未完成的步骤，若有则**必须将它完成**

## 约束和注意事项

1. **必须使用下载的图标资源**：不允许使用内联 SVG 或外部图标库
2. **共享 Tailwind 配置**：所有页面使用统一的 tailwind.config.js
3. **不要包含移动端状态栏**：HTML 原型不包含顶部时间、电池信息
