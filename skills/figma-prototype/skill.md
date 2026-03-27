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

**使用任意一个页面的 nodeId 获取该文件的设计变量：**

**调用 `get_variable_defs`：**

```json
{
  "fileKey": "从URL提取的fileKey",
  "nodeId": "任意一个页面的nodeId"
}
```

#### 兜底方案：get_design_context

当 `get_variable_defs` 失败时，调用 `get_design_context`：

```json
{
  "fileKey": "AbCdEf123",
  "nodeId": "1:2"
}
```

从返回的信息中提取颜色值、字体大小等作为 token。

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

### 第 4 步：批量下载 Figma 资源（icon，logo）

#### 4.1 并行获取所有页面的设计上下文

**对每个 URL 并行调用 `get_design_context`：**

```json
{
  "fileKey": "AbCdEf123",
  "nodeId": "1:2"
}
```
收集所有页面的资源链接。

#### 4.2 提取和合并资源下载链接
通过Figma MCP 服务内置的 assets endpoint下载所有icon，将icon保存到项目的 `/public/icons`文件夹下

**约束**
- 去重：相同 URL 的资源只下载一次

---

### 第 5 步：生成图标组件入口

**使用SVGR插件**

**示例**
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

#### 5.2 使用方式

```tsx
import { IconSearch, IconUser } from '@/components/icons';

// 在组件中使用
<button>
  <IconSearch className="w-5 h-5 text-primary-500" />
  搜索
</button>
```

---

### 第 6 步：批量生成原型页面,保存到项目`/prototype`文件夹

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

### 第 7 步：生成汇总报告

#### 7.1 创建 /prototype/prototype.md

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

## 约束和注意事项

1. **必须使用下载的图标资源**：不允许使用内联 SVG 或外部图标库
2. **共享 Tailwind 配置**：所有页面使用统一的 tailwind.config.js
3. **不要包含移动端状态栏**：HTML 原型不包含顶部时间、电池信息
