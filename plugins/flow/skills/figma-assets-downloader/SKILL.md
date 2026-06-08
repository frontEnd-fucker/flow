---
name: figma-assets-downloader
description: 从 Figma 设计稿中自动下载所有图片、图标、SVG 等资源文件到本地项目
---

**职责：**
专门负责解析 Figma 设计稿，识别所有可导出的资源（图片、图标、SVG 等），并下载到项目指定目录。

## 开始：询问用户连接方式

**必须先询问用户使用哪种方式连接 Figma：**

```json
{
  "questions": [
    {
      "question": "请选择 Figma 连接方式",
      "header": "连接方式",
      "multiSelect": false,
      "options": [
        {"label": "Figma Desktop MCP（推荐，更稳定）", "value": "desktop"},
        {"label": "远程 Figma MCP（无需安装 Desktop）", "value": "remote"}
      ]
    }
  ]
}
```

**根据用户选择，执行对应的分支流程：**

| 选择 | 使用工具 | 说明 |
|-----|---------|------|
| **Desktop MCP** | `use_figma` | 通过本地 Figma Desktop 应用导出资源 |
| **远程 MCP** | `get_design_context` 等 | 通过 Figma API 获取下载链接 |

⚠️ **重要**：一旦用户选择 Desktop MCP，后续所有步骤**只使用 `use_figma` 工具**，禁止使用远程 Figma MCP 工具。

---

## 分支 A：使用 Figma Desktop MCP

### A1. 通过 Desktop 导出资源

使用 `use_figma` 工具执行 Plugin API 代码导出所有可导出资源：

```javascript
async function exportAllAssets() {
  const results = [];

  // 递归遍历节点查找可导出资源
  async function traverse(node) {
    if (node.exportSettings && node.exportSettings.length > 0) {
      for (const setting of node.exportSettings) {
        try {
          const bytes = await node.exportAsync(setting.format);
          let ext = 'png';
          if (setting.format === 'SVG') ext = 'svg';
          else if (setting.format === 'PDF') ext = 'pdf';
          else if (setting.format === 'JPG') ext = 'jpg';

          results.push({
            name: node.name.replace(/[^a-zA-Z0-9]/g, '_'), // 清理文件名
            extension: ext,
            bytes: Array.from(bytes), // 转为普通数组便于序列化
            format: setting.format,
            nodeId: node.id,
            width: node.width,
            height: node.height
          });
        } catch (error) {
          console.error(`导出失败: ${node.name}`, error);
        }
      }
    }

    if ('children' in node) {
      for (const child of node.children) {
        await traverse(child);
      }
    }
  }

  // 从当前选中页面或指定 node 开始
  await traverse(figma.currentPage);
  return results;
}

exportAllAssets();
```

**use_figma 调用参数：**

```json
{
  "fileKey": "文件 Key（如果用户已打开可省略）",
  "code": "上述 JavaScript 代码",
  "description": "从 Figma Desktop 导出所有标记为可导出的资源"
}
```

### A2. 处理返回的资源数据

`use_figma` 返回的结果包含资源的字节数组，需要转换为文件：

```javascript
// 将返回的 bytes 数组写入文件
const fs = require('fs');
const path = require('path');

function saveAssets(exportedAssets, outputDir) {
  for (const asset of exportedAssets) {
    const filePath = path.join(outputDir, `${asset.name}.${asset.extension}`);
    const buffer = Buffer.from(asset.bytes);
    fs.writeFileSync(filePath, buffer);
  }
}
```

### A3. Desktop 方式前置要求

**必须确认以下条件满足：**
- ✅ 用户已安装 Figma Desktop 应用
- ✅ 用户已登录 Figma Desktop
- ✅ 用户在 Desktop 中打开了目标设计文件

**如果不满足：**
- 告知用户需要先安装并打开 Figma Desktop
- 询问是否切换到远程 MCP 方式

---

## 分支 B：使用远程 Figma MCP

### B1. 获取设计信息

使用 `get_design_context` 工具获取设计稿完整信息：

```json
{
  "fileKey": "从URL提取的fileKey",
  "nodeId": "从URL提取的nodeId（-替换为:）"
}
```

### B2. 提取资源下载链接

从 `get_design_context` 返回结果中提取：
- `assetDownloadUrls`: 预签名的资源下载 URL 映射表
  - Key: 资源名称或节点 ID
  - Value: 可下载的 URL

### B3. 创建输出目录

```bash
mkdir -p /public/assets/figma/{页面名称}
```

### B4. 下载资源文件

**方式 1：使用 curl（推荐）**

```bash
# 遍历所有 assetDownloadUrls 下载资源
for url in "${urls[@]}"; do
  curl -L "$url" -o "/public/assets/figma/{页面名称}/{资源名}.{扩展名}"
done
```

**方式 2：使用 Node.js**

```javascript
const fs = require('fs');
const https = require('https');
const path = require('path');

async function downloadAssets(assetDownloadUrls, outputDir) {
  for (const [name, url] of Object.entries(assetDownloadUrls)) {
    const ext = url.includes('svg') ? 'svg' : 'png';
    const filePath = path.join(outputDir, `${name}.${ext}`);

    const file = fs.createWriteStream(filePath);
    https.get(url, (response) => {
      response.pipe(file);
    });
  }
}
```

### B5. 资源分类存储

按资源类型分类存放：

```
/public/assets/figma/
├── {页面名称}/
│   ├── images/          # 图片资源
│   │   ├── hero-banner.png
│   │   └── user-avatar.jpg
│   ├── icons/           # 图标资源
│   │   ├── search.svg
│   │   ├── menu.svg
│   │   └── close.svg
│   └── logos/           # Logo 资源
│       └── brand-logo.svg
```

### B6. 生成资源清单

创建 `assets-manifest.json` 文件，记录所有下载的资源：

```json
{
  "pageName": "登录页",
  "figmaUrl": "https://figma.com/design/xxx/Project?node-id=1-2",
  "downloadTime": "2026-03-26T10:30:00Z",
  "assets": [
    {
      "name": "hero-banner",
      "file": "images/hero-banner.png",
      "type": "image/png",
      "figmaNodeId": "123:456"
    },
    {
      "name": "search-icon",
      "file": "icons/search.svg",
      "type": "image/svg+xml",
      "figmaNodeId": "123:789"
    }
  ],
  "totalCount": 2
}
```

### B7. 输出结果

向用户汇报下载结果：

```markdown
## Figma 资源下载完成

**来源：** https://figma.com/design/{fileKey}/Project?node-id={nodeId}
**输出目录：** /public/assets/figma/{页面名称}/

**下载统计：**
- 图片资源：3 个
- 图标资源：5 个
- Logo 资源：1 个
- **总计：9 个文件**

**资源清单：** /public/assets/figma/{页面名称}/assets-manifest.json

**使用方式：**
```html
<img src="/assets/figma/{页面名称}/images/hero-banner.png" />
<img src="/assets/figma/{页面名称}/icons/search.svg" />
```
```

## 错误处理

### 下载失败重试

- 单个资源下载失败时，记录错误并重试 3 次
- 重试仍失败则跳过，最后汇总失败列表

### 网络错误

```bash
# 使用 --retry 参数自动重试
curl -L --retry 3 --retry-delay 2 "$url" -o "$output"
```

### 返回结果

成功：返回下载的资源清单和路径
失败：返回错误原因和失败的资源列表
