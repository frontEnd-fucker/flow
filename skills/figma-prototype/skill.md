---
name: figma-prototype
description: 根据 Figma 设计稿生成完整的 HTML 原型页面，自动下载图标资源，提取设计 token 生成 Tailwind 配置
---

## 概述

本 skill 根据 Figma 设计稿生成完整的 HTML 原型页面，包含以下核心功能：

1. **自动下载 Figma 资源**：通过 Figma MCP server 的 assets endpoint 下载所有 icon、图片等资源
2. **提取设计 Token**：使用 `get_variable_defs` 获取 Figma 变量，生成 Tailwind CSS 配置文件
3. **生成原型页面**：基于设计稿生成 HTML + Tailwind CSS 页面，使用设计 token 实现

---

## 执行流程

### 第 1 步：收集 Figma 链接

**询问用户：**
- 是否有 Figma 设计稿？如果有，请提供每个页面的 Figma 链接

**提示用户获取链接的方法：**
- 在 Figma Dev 模式下，按住 Shift 键同时选中多个页面
- 点击 "Copy link to selection" 或 "Copy example prompt"

---

### 第 2 步：提取 fileKey 和 nodeId

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

**为每个 Figma 页面执行此提取操作。**

---

### 第 3 步：获取设计 Token

**优先使用 `get_variable_defs`，失败时使用 `get_design_context` 作为兜底方案。**

#### 3.1 方案 A：调用 get_variable_defs（优先）

```json
{
  "fileKey": "从URL提取的fileKey",
  "nodeId": "从URL提取的nodeId"
}
```

**成功时：** 从返回结果中提取以下 token 类型：

| Token 类型 | 用途 | Tailwind 映射 |
|-----------|------|--------------|
| `color` | 颜色变量 | `colors` 主题 |
| `number` | 间距、圆角、字体大小 | `spacing`, `borderRadius`, `fontSize` |
| `string` | 字体族 | `fontFamily` |

**Token 命名规范转换：**
- Figma: `color/primary/500` → Tailwind: `primary-500`
- Figma: `spacing/medium` → Tailwind: `spacing-medium`
- Figma: `font/size/heading-1` → Tailwind: `font-heading-1`

#### 3.2 方案 B：使用 get_design_context 兜底

当 `get_variable_defs` 失败或返回空时，调用 `get_design_context`：

```json
{
  "fileKey": "从URL提取的fileKey",
  "nodeId": "从URL提取的nodeId"
}
```

**从返回结果提取 Token：**

| 来源 | 提取方式 | 用途 |
|------|---------|------|
| `code` 中的颜色值 | 正则匹配 `#RRGGBB` 或 `rgb()` | 颜色 Token |
| `code` 中的字体大小 | 匹配 `text-{size}` 或 `fontSize` | 字体大小 Token |
| `code` 中的间距值 | 匹配 `p-{size}`, `m-{size}` 等 | 间距 Token |
| `assetDownloadUrls` 键名 | 分析资源命名规律 | 推断颜色/尺寸命名 |

**兜底策略：**
1. 提取代码中出现的所有颜色值，去重后生成颜色调色板
2. 使用常见间距值（4, 8, 12, 16, 24, 32, 48）作为间距 Token
3. 使用常见字体大小（12, 14, 16, 18, 20, 24, 32）作为字体 Token

---

### 第 4 步：生成 Tailwind 配置文件

#### 4.1 创建 tailwind.config.js

基于获取的 design token 生成配置文件：

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./prototype/**/*.html'],
  theme: {
    extend: {
      // 颜色 Token
      colors: {
        // 从 get_variable_defs 提取的颜色变量
        'primary-50': '#f0f9ff',
        'primary-100': '#e0f2fe',
        'primary-500': '#0ea5e9',
        'primary-600': '#0284c7',
        'primary-900': '#0c4a6e',
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
        // ... 其他间距
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

#### 3.2 创建基础 CSS 文件

```css
/* /prototype/styles/tailwind.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 基础样式 */
@layer base {
  body {
    @apply font-sans text-neutral-900 bg-white;
  }
}

/* 组件样式 */
@layer components {
  .btn-primary {
    @apply px-md py-sm bg-primary-500 text-white rounded-md hover:bg-primary-600 transition-colors;
  }

  .btn-secondary {
    @apply px-md py-sm bg-neutral-100 text-neutral-900 rounded-md hover:bg-neutral-200 transition-colors;
  }

  .input-field {
    @apply px-md py-sm border border-neutral-200 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500;
  }

  .card {
    @apply bg-white rounded-lg shadow-md p-lg;
  }
}
```

---

### 第 5 步：下载 Figma 资源（Icons 和图片）

#### 5.1 获取设计上下文

使用 `get_design_context` 获取页面完整信息：

```json
{
  "fileKey": "从URL提取的fileKey",
  "nodeId": "从URL提取的nodeId"
}
```

#### 5.2 提取资源下载链接

从返回结果中提取 `assetDownloadUrls`：

```json
{
  "assetDownloadUrls": {
    "icon-search": "https://assets.figma.com/.../icon-search.svg",
    "icon-menu": "https://assets.figma.com/.../icon-menu.svg",
    "icon-close": "https://assets.figma.com/.../icon-close.svg",
    "logo-brand": "https://assets.figma.com/.../logo-brand.png",
    "hero-image": "https://assets.figma.com/.../hero-image.png"
  }
}
```

#### 5.3 创建资源目录结构

```bash
mkdir -p /prototype/assets/icons
mkdir -p /prototype/assets/images
```

#### 5.4 下载所有资源

**使用 Bash + curl 下载：**

```bash
#!/bin/bash

# 图标资源
icons=(
  "icon-search|https://..."
  "icon-menu|https://..."
  "icon-close|https://..."
  "icon-arrow-right|https://..."
  "icon-chevron-down|https://..."
  "icon-user|https://..."
  "icon-settings|https://..."
  "icon-notification|https://..."
)

for item in "${icons[@]}"; do
  IFS='|' read -r name url <<< "$item"
  curl -L --retry 3 --retry-delay 2 "$url" -o "/prototype/assets/icons/${name}.svg"
  echo "Downloaded: ${name}.svg"
done

# 图片资源
images=(
  "logo-brand|https://..."
  "hero-banner|https://..."
  "user-avatar|https://..."
)

for item in "${images[@]}"; do
  IFS='|' read -r name url <<< "$item"
  curl -L --retry 3 --retry-delay 2 "$url" -o "/prototype/assets/images/${name}.png"
  echo "Downloaded: ${name}.png"
done
```

#### 5.5 验证下载结果

检查所有资源是否成功下载，生成清单文件：

```json
{
  "pageName": "登录页",
  "figmaUrl": "https://figma.com/design/xxx/Project?node-id=1-2",
  "downloadTime": "2026-03-27T10:30:00Z",
  "assets": {
    "icons": [
      { "name": "icon-search", "file": "icons/icon-search.svg", "type": "image/svg+xml" },
      { "name": "icon-menu", "file": "icons/icon-menu.svg", "type": "image/svg+xml" },
      { "name": "icon-close", "file": "icons/icon-close.svg", "type": "image/svg+xml" }
    ],
    "images": [
      { "name": "logo-brand", "file": "images/logo-brand.png", "type": "image/png" },
      { "name": "hero-banner", "file": "images/hero-banner.png", "type": "image/png" }
    ]
  },
  "totalCount": 5
}
```

---

### 第 6 步：生成原型页面

#### 6.1 分析设计稿结构

从 `get_design_context` 返回的代码中提取：

- **页面布局**：Header、Sidebar、Main Content、Footer 等
- **组件列表**：Button、Input、Card、Modal 等
- **使用的图标**：根据组件推断需要的图标

#### 6.2 生成 HTML 页面

**页面结构模板：**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{页面名称}</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- 自定义配置 -->
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            // 从 design token 提取的颜色
            'primary-50': '#f0f9ff',
            'primary-500': '#0ea5e9',
            'primary-600': '#0284c7',
            'neutral-100': '#f5f5f5',
            'neutral-900': '#171717',
          },
          spacing: {
            'xs': '4px',
            'sm': '8px',
            'md': '16px',
            'lg': '24px',
            'xl': '32px',
          }
        }
      }
    }
  </script>
  <!-- 项目样式 -->
  <link rel="stylesheet" href="./styles/prototype.css">
</head>
<body class="bg-neutral-50">
  <!-- 页面内容 -->

  <!-- 图标使用示例 -->
  <img src="./assets/icons/icon-search.svg" class="w-5 h-5" alt="Search">
  <img src="./assets/icons/icon-menu.svg" class="w-6 h-6" alt="Menu">

</body>
</html>
```

#### 6.3 图标使用规范

**所有图标必须从下载的资源中获取：**

```html
<!-- ✅ 正确：使用下载的图标资源 -->
<button class="btn-primary flex items-center gap-sm">
  <img src="./assets/icons/icon-search.svg" class="w-4 h-4" alt="">
  <span>搜索</span>
</button>

<!-- ✅ 正确：使用下载的 SVG 图标 -->
<div class="flex items-center gap-md">
  <img src="./assets/icons/icon-user.svg" class="w-6 h-6 text-primary-500" alt="User">
  <span class="font-body">用户名</span>
</div>

<!-- ❌ 错误：不要内联 SVG 或使用外部图标库 -->
<!-- 不要使用 <svg>...</svg> 内联 -->
<!-- 不要使用 Font Awesome 等外部图标库 -->
```

#### 6.4 使用设计 Token

**颜色：**
```html
<!-- 使用 Tailwind 配置中的颜色 -->
<button class="bg-primary-500 hover:bg-primary-600 text-white">
  主要按钮
</button>

<div class="bg-neutral-100 text-neutral-900">
  卡片背景
</div>
```

**间距：**
```html
<!-- 使用自定义间距 -->
<div class="p-md gap-sm">
  <div class="mb-lg">...</div>
</div>
```

**字体：**
```html
<!-- 使用自定义字体大小 -->
<h1 class="text-heading-1">标题</h1>
<h2 class="text-heading-2">副标题</h2>
<p class="text-body">正文内容</p>
```

---

### 第 7 步：输出结果

#### 7.1 生成项目结构

```
/prototype/
├── index.html                 # 入口页面（导航到各原型页面）
├── {页面1}/
│   └── index.html            # 页面1原型
├── {页面2}/
│   └── index.html            # 页面2原型
├── styles/
│   ├── tailwind.css          # Tailwind 基础样式
│   └── prototype.css         # 项目自定义样式
├── assets/
│   ├── icons/                # 图标资源（从 Figma 下载）
│   │   ├── icon-search.svg
│   │   ├── icon-menu.svg
│   │   ├── icon-close.svg
│   │   └── ...
│   └── images/               # 图片资源（从 Figma 下载）
│       ├── logo-brand.png
│       ├── hero-banner.png
│       └── ...
├── tailwind.config.js        # Tailwind 配置（含 design token）
└── prototype.md              # 原型说明文档
```

#### 7.2 生成 prototype.md 报告

```markdown
# Figma 原型生成报告

## 概述

- **生成时间**: 2026-03-27 10:30:00
- **Figma 源文件**: https://figma.com/design/xxx/Project

## 设计 Token

### 颜色系统
| Token | 值 | 用途 |
|-------|-----|------|
| primary-500 | #0ea5e9 | 主色 |
| primary-600 | #0284c7 | 主色悬停 |
| neutral-100 | #f5f5f5 | 背景色 |
| neutral-900 | #171717 | 文字色 |

### 间距系统
| Token | 值 |
|-------|-----|
| xs | 4px |
| sm | 8px |
| md | 16px |
| lg | 24px |
| xl | 32px |

## 下载的资源

### 图标 (8个)
- icon-search.svg
- icon-menu.svg
- icon-close.svg
- icon-arrow-right.svg
- icon-chevron-down.svg
- icon-user.svg
- icon-settings.svg
- icon-notification.svg

### 图片 (3个)
- logo-brand.png
- hero-banner.png
- user-avatar.png

## 生成的页面

1. **登录页** (`/login/index.html`)
   - Figma: https://figma.com/design/xxx/Project?node-id=1-2
   - 包含图标: icon-user, icon-lock

2. **首页** (`/home/index.html`)
   - Figma: https://figma.com/design/xxx/Project?node-id=3-4
   - 包含图标: icon-search, icon-menu, icon-notification

## 使用说明

1. 打开 `index.html` 浏览所有原型页面
2. 所有图标资源位于 `/assets/icons/`
3. 所有图片资源位于 `/assets/images/`
4. Tailwind 配置已集成设计 Token

## 注意事项

- 所有图标均从 Figma 设计稿下载，确保与设计一致
- 颜色、间距等样式使用提取的 design token
- 如需修改样式，请编辑 `tailwind.config.js`
```

---

## 错误处理

### get_variable_defs 失败

**原因**：
- Figma 文件没有定义变量
- 权限不足

**处理**：
- 使用 `get_design_context` 返回的颜色值作为备选
- 生成基础的 Tailwind 配置

### 资源下载失败

**处理**：
- 重试 3 次
- 记录失败的资源
- 在报告中标注缺失资源

### 图标缺失

**处理**：
- 使用占位符图标（带红色边框提示）
- 在报告中标注需要补充的图标

---

## 约束和注意事项

1. **必须使用下载的图标资源**：不允许使用内联 SVG 或外部图标库
2. **必须使用 design token**：颜色、间距等样式必须使用提取的 token
3. **不要包含移动端状态栏**：HTML 原型不包含顶部时间、电池信息
4. **资源路径**：使用相对路径 `./assets/icons/` 和 `./assets/images/`
5. **响应式设计**：根据 Figma 设计稿实现响应式布局
