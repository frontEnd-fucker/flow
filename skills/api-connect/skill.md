---
name: api-connect
description: 将本地 mock API 转换为真实 API 接口，通过 OpenAPI 规范自动生成 TypeScript SDK 并替换项目中的 API 调用
---

# API Connect - Mock 到真实 API 转换

将项目中的 mock API 调用自动转换为基于 OpenAPI 规范的真实 API SDK 调用。

## 执行流程

### 步骤 1: 获取 OpenAPI 文档

**提示用户提供接口文档地址：**

> 请提供 OpenAPI 接口文档的地址：
> - JSON/YAML 文件的本地路径（如 `./swagger.json` 或 `docs/api.yaml`）
> - 远程 URL（如 `https://api.example.com/swagger.json`）

**获取文档后：**
1. 使用 `Read` 或 `WebFetch` 读取 OpenAPI 文档内容
2. 验证文档格式正确性（检查 `openapi` 版本字段）
3. 提取关键信息：
   - API 基础路径 (basePath/server url)
   - 所有端点路径和 HTTP 方法
   - 请求/响应 Schema 定义

### 步骤 2: 生成 TypeScript SDK

**执行 SDK 生成：**

```bash
# 使用 swagger-typescript-api 生成 SDK
npx swagger-typescript-api \
  -p <openapi文档路径> \
  -o ./api \
  --name api.ts \
  --axios
```

**常用配置选项：**

| 选项 | 说明 |
|------|------|
| `-p, --path` | OpenAPI 文档路径（JSON/YAML） |
| `-o, --output` | 输出目录（默认 `./api`） |
| `--name` | 生成的文件名（默认 `Api.ts`） |
| `--axios` | 使用 axios 作为 HTTP 客户端 |
| `--fetch` | 使用 fetch 作为 HTTP 客户端 |
| `--modular` | 生成模块化代码（分离类型和 API） |
| `--single-http-client` | 生成单一 HTTP 客户端实例 |
| `--default-response` | 指定默认响应类型 |

**生成后验证：**
1. 检查 `./api/` 目录是否成功创建
2. 确认包含以下关键文件：
   - `api.ts` - 包含所有 API 方法和类型定义
   - 如果使用 `--modular` 模式，还会有分离的 types 和 http-client 文件
3. 检查 TypeScript 编译是否通过

### 步骤 3: 配置真实 API 连接

此步骤通过多轮对话逐一确认 API 连接信息。每次只询问一个问题，得到用户回答后再进行下一个。

#### 步骤 3.1: 询问 API 基础地址

> 请提供真实 API 的基础地址（如 `https://api.example.com` 或 `http://localhost:8080`）

#### 步骤 3.2: 询问认证方式

使用 `AskUserQuestion` 工具，以选项方式询问：

```json
{
  "questions": [
    {
      "question": "请选择 API 认证方式",
      "header": "认证方式",
      "options": [
        {"label": "无需认证", "description": "API 无需任何认证"},
        {"label": "Bearer Token (JWT)", "description": "使用 JWT Token 进行认证"},
        {"label": "API Key", "description": "使用 API Key 进行认证"},
        {"label": "其他", "description": "其他认证方式，请说明"}
      ]
    }
  ]
}
```

如果选择 "其他"，追问具体认证方式。

#### 步骤 3.3: 询问是否需要开发环境代理

使用 `AskUserQuestion` 工具：

```json
{
  "questions": [
    {
      "question": "是否需要配置开发环境代理（Proxy）来解决跨域问题？",
      "header": "Proxy 配置",
      "options": [
        {"label": "需要配置", "description": "配置 Vite/Webpack 等代理"},
        {"label": "不需要", "description": "直接调用真实 API"}
      ]
    }
  ]
}
```

**如用户选择需要 Proxy：**

进一步询问代理配置：

```json
{
  "questions": [
    {
      "question": "请配置代理路径前缀",
      "header": "Proxy 路径",
      "options": [
        {"label": "/api", "description": "代理 /api 开头的请求"},
        {"label": "/api/v1", "description": "代理 /api/v1 开头的请求"},
        {"label": "其他", "description": "自定义路径前缀"}
      ]
    }
  ]
}
```

**根据项目类型自动配置 Proxy：**

**Vite 项目 (`vite.config.ts`)：**
```typescript
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
});
```

**Create React App (`src/setupProxy.js`)：**
```javascript
const { createProxyMiddleware } = require('http-proxy-middleware');

module.exports = function(app) {
  app.use(
    '/api',
    createProxyMiddleware({
      target: 'https://api.example.com',
      changeOrigin: true,
      pathRewrite: { '^/api': '' },
    })
  );
};
```

**Next.js (`next.config.js`)：**
```javascript
module.exports = {
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'https://api.example.com/:path*',
      },
    ];
  },
};
```

**Vue CLI (`vue.config.js`)：**
```javascript
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        pathRewrite: { '^/api': '' },
      },
    },
  },
};
```

配置完成后告知用户：

> ✅ 已配置开发环境代理：
> - 开发时请求 `/api/users` 将代理到 `https://api.example.com/users`

#### 步骤 3.4: 询问数据获取方案

使用 `AskUserQuestion` 工具：

```json
{
  "questions": [
    {
      "question": "请选择项目中使用的数据获取方案",
      "header": "数据获取库",
      "options": [
        {"label": "直接使用 SDK/Axios", "description": "直接调用 SDK 方法"},
        {"label": "Tanstack Query (React Query)", "description": "使用 useQuery/useMutation"},
        {"label": "SWR", "description": "使用 useSWR"},
        {"label": "RTK Query", "description": "Redux Toolkit Query"},
        {"label": "Apollo Client", "description": "GraphQL 客户端"},
        {"label": "其他", "description": "其他方案"}
      ]
    }
  ]
}
```

#### 步骤 3.5: 配置 SDK baseURL

根据前面收集的信息，更新 SDK 配置：

```typescript
// 方式 1: 创建 Api 实例时传入 baseURL
import { Api } from './api';

const api = new Api({
  baseURL: 'https://api.example.com', // 用户提供的地址
});

// 方式 2: 如果使用 axios，配置 axios defaults
import axios from 'axios';
axios.defaults.baseURL = 'https://api.example.com';

// 使用
const users = await api.users.usersGet();
```

**配置认证：**

```typescript
// Bearer Token 认证
const api = new Api({
  baseURL: 'https://api.example.com',
  headers: {
    Authorization: `Bearer ${token}`,
  },
});

// 或添加请求拦截器
api.instance.interceptors.request.use((config) => {
  config.headers.Authorization = `Bearer ${localStorage.getItem('token')}`;
  return config;
});
```

### 步骤 4: 分析 Mock API 并展示映射关系

**扫描项目中的 Mock API 调用：**

搜索以下模式的 API 调用：
- `fetch('/api/...')` 或 `fetch("/api/...")`
- `axios.get('/api/...')` 等 axios 调用
- `$.ajax({ url: '/api/...' })` 等 jQuery 调用
- 项目中定义的 mock 服务函数

**提取信息：**
- Mock 端点路径（如 `/api/users`, `/api/orders/:id`）
- HTTP 方法（GET/POST/PUT/DELETE）
- 当前使用的请求/响应类型定义

**生成映射表格并展示给用户：**

```markdown
| # | Mock API 端点 | 方法 | 当前调用位置 | SDK 对应方法 | 状态 |
|---|--------------|------|-------------|-------------|------|
| 1 | /api/users | GET | src/services/user.ts:45 | UsersApi.usersGet() | 待确认 |
| 2 | /api/users | POST | src/services/user.ts:78 | UsersApi.usersPost() | 待确认 |
| 3 | /api/orders/{id} | GET | src/services/order.ts:23 | OrdersApi.ordersIdGet() | 待确认 |
```

**展示映射表格：**

展示检测到的映射关系表格：

```markdown
| # | Mock API 端点 | 方法 | 当前调用位置 | SDK 对应方法 | 状态 |
|---|--------------|------|-------------|-------------|------|
| 1 | /api/users | GET | src/services/user.ts:45 | UsersApi.usersGet() | 待确认 |
| 2 | /api/users | POST | src/services/user.ts:78 | UsersApi.usersPost() | 待确认 |
| 3 | /api/orders/{id} | GET | src/services/order.ts:23 | OrdersApi.ordersIdGet() | 待确认 |
```

**确认映射关系：**

使用 `AskUserQuestion` 工具：

```json
{
  "questions": [
    {
      "question": "映射关系是否正确？",
      "header": "映射确认",
      "options": [
        {"label": "完全正确", "description": "所有映射都正确，开始转换"},
        {"label": "需要修改", "description": "有映射需要调整"}
      ]
    }
  ]
}
```

**如用户选择"需要修改"：**

逐一询问需要修改的项：

> 第 1 项 `/api/users` 的映射：
> - 当前 SDK 方法：`UsersApi.usersGet()`
> - 请提供正确的 SDK 方法名（如确认正确请回复"正确"）

> 是否有遗漏的 API 调用需要添加？如果有，请提供端点路径和方法。

> 是否有不需要替换的调用？如果有，请提供编号。

待用户确认所有映射无误后，告知用户：

> ✅ 映射关系已确认。现在开始执行代码转换。

### 步骤 5: 执行代码转换

**用户确认后，根据数据获取方案执行替换：**

---

#### 方案 A: 直接使用 SDK/Axios

1. **替换 API 调用：**
   - 将 `fetch('/api/users')` 替换为 SDK 方法调用
   - 添加 SDK 导入语句：`import { Api } from '../api'`

2. **替换类型定义：**
   - 删除原有的 mock 类型定义文件
   - 更新导入语句，从 SDK 导入类型

**转换示例：**

转换前：
```typescript
// src/services/user.ts
import { User } from '../types/user';

export async function getUsers(): Promise<User[]> {
  const res = await fetch('/api/users');
  return res.json();
}
```

转换后：
```typescript
// src/services/user.ts
import { Api, User } from '../../api';

const api = new Api({
  baseURL: 'https://api.example.com', // 或使用环境变量
});

export async function getUsers(): Promise<User[]> {
  const res = await api.users.usersGet();
  return res.data;
}
```

---

#### 方案 B: Tanstack Query (React Query)

**保持 hook 结构，替换 queryFn 中的 fetch：**

转换前：
```typescript
// src/hooks/useUsers.ts
import { useQuery } from '@tanstack/react-query';

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const res = await fetch('/api/users');
      return res.json();
    }
  });
}
```

转换后：
```typescript
// src/hooks/useUsers.ts
import { useQuery } from '@tanstack/react-query';
import { Api } from '../api';

const api = new Api({
  baseURL: 'https://api.example.com', // 或使用环境变量
});

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const res = await api.users.usersGet();
      return res.data;
    }
  });
}
```

**mutation 示例：**

转换前：
```typescript
export function useCreateUser() {
  return useMutation({
    mutationFn: async (user: CreateUserRequest) => {
      const res = await fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(user)
      });
      return res.json();
    }
  });
}
```

转换后：
```typescript
export function useCreateUser() {
  return useMutation({
    mutationFn: async (user: CreateUserRequest) => {
      const res = await api.users.usersPost(user);
      return res.data;
    }
  });
}
```

**Tanstack Query 集成要点：**
- 保持 `queryKey` 不变，确保缓存策略不受影响
- 将 `queryFn` / `mutationFn` 中的 fetch 替换为 SDK 调用
- 如需处理错误，使用 SDK 返回的错误信息

---

#### 方案 C: SWR

**保持 useSWR 调用，替换 fetcher：**

转换前：
```typescript
// src/hooks/useUsers.ts
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

export function useUsers() {
  return useSWR('/api/users', fetcher);
}
```

转换后：
```typescript
// src/hooks/useUsers.ts
import useSWR from 'swr';
import { Api } from '../api';

const api = new Api({
  baseURL: 'https://api.example.com', // 或使用环境变量
});

export function useUsers() {
  return useSWR(
    '/api/users',
    () => api.users.usersGet().then(r => r.data)
  );
}
```

**SWR 集成要点：**
- 保持 SWR key 不变
- 将 fetcher 函数替换为 SDK 调用
- 保持 SWR 配置选项（revalidateOnFocus、refreshInterval 等）不变

---

#### 方案 D: RTK Query

**使用 queryFn 替代 fetchBaseQuery：**

转换前：
```typescript
// src/store/userApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const userApi = createApi({
  reducerPath: 'userApi',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: (builder) => ({
    getUsers: builder.query<User[], void>({
      query: () => 'users',
    }),
    createUser: builder.mutation<User, CreateUserRequest>({
      query: (body) => ({
        url: 'users',
        method: 'POST',
        body
      })
    })
  }),
});
```

转换后：
```typescript
// src/store/userApi.ts
import { createApi } from '@reduxjs/toolkit/query/react';
import { Api } from '../api';

const api = new Api({
  baseURL: 'https://api.example.com', // 或使用环境变量
});

export const userApi = createApi({
  reducerPath: 'userApi',
  baseQuery: () => ({ data: null }), // 占位
  endpoints: (builder) => ({
    getUsers: builder.query<User[], void>({
      queryFn: async () => {
        const res = await api.users.usersGet();
        return { data: res.data };
      },
    }),
    createUser: builder.mutation<User, CreateUserRequest>({
      queryFn: async (body) => {
        const res = await api.users.usersPost(body);
        return { data: res.data };
      }
    })
  }),
});
```

**RTK Query 集成要点：**
- 使用 `queryFn` 替代 `query` + `fetchBaseQuery`
- 保持 reducerPath 和 tagTypes 不变
- 保持 providesTags / invalidatesTags 逻辑（如有）

---

#### 方案 E: Apollo Client

**注意：** 如果 OpenAPI 文档生成的是 REST API，而项目使用 Apollo Client（GraphQL），需要评估：

1. 后端是否同时提供 GraphQL API？
   - 是：建议生成 GraphQL 客户端（需要 GraphQL schema）
   - 否：建议切换到 Tanstack Query 或 SWR 来处理 REST API

2. 如果必须使用 Apollo Client 调用 REST API：
   - 使用 `apollo-link-rest` 或保持原有的 REST 调用方式

**从 Apollo Client 切换到 Tanstack Query（推荐）：**

转换前：
```typescript
import { useQuery, gql } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
    }
  }
`;

export function useUsers() {
  return useQuery(GET_USERS);
}
```

转换后：
```typescript
import { useQuery } from '@tanstack/react-query';
import { Api } from '../api';

const api = new Api({
  baseURL: 'https://api.example.com', // 或使用环境变量
});

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const res = await api.users.usersGet();
      return res.data;
    }
  });
}
```

---

#### 通用替换步骤

无论使用哪种数据获取方案，都需要执行：

1. **替换类型定义：**
   - 删除原有的 mock 类型定义文件（如 `src/types/user.ts` 中的接口）
   - 更新导入语句，从 SDK 导入类型：`import { User, UserResponse } from '../api'`

2. **处理 SDK 配置：**
   - 在 SDK 配置中设置正确的 baseURL
   - 如有认证需求，配置拦截器传递 token

### 步骤 6: 验证与清理

**编译检查：**
```bash
# 运行 TypeScript 编译检查
npx tsc --noEmit
```

**验证清单：**
- [ ] SDK 导入路径正确
- [ ] 所有 mock 调用已替换
- [ ] 类型定义无冲突
- [ ] 无编译错误

**向用户报告：**

> API 转换完成！已替换 X 个 API 调用，涉及 Y 个文件。
>
> 配置摘要：
> - SDK 生成路径：`./api/`
> - API Base URL：`https://api.example.com`（根据用户输入）
> - Proxy 配置：已配置 Vite proxy（如需要）
>
> 建议验证步骤：
> 1. 启动开发服务器 (`npm run dev` 或 `yarn dev`)，测试关键 API 调用
> 2. 检查浏览器控制台是否有请求错误
> 3. 在 Network 面板中验证请求是否正确代理到真实 API
> 4. 如有问题，可以查看 `api/` 目录中的 SDK 文档

## 输入

- OpenAPI 规范文档（JSON/YAML）的本地路径或 URL
- 项目中现有的 mock API 调用代码

## 输出

- `./api/` 目录下的 TypeScript SDK
- 修改后的业务代码文件（替换 mock 调用为 SDK 调用）
- 更新的类型导入语句
- 配置的开发环境 Proxy（如 `vite.config.ts` 中的 `server.proxy`）

## 注意事项

1. **SDK 生成路径**：固定为项目根目录的 `api/` 文件夹
2. **类型处理**：完全复用 SDK 生成的类型，删除原有 mock 类型
3. **备份建议**：执行前建议用户提交当前代码，以便必要时回滚
4. **路径别名**：如果项目使用路径别名（如 `@/api`），需要额外配置
5. **请求拦截器**：如需统一处理认证、错误等，通过 `api.instance.interceptors` 配置（参见 FAQ）
6. **Proxy 配置**：开发环境代理仅解决开发时的跨域问题，生产环境需要确保部署后 API 可正常访问
7. **数据获取库**：转换后的代码保持原有数据获取库的使用模式（queryKey、缓存策略等）
8. **Apollo Client**：如果使用 GraphQL，需要单独生成 GraphQL 客户端或使用 Apollo REST Link

## 依赖要求

- Node.js >= 14
- 项目中使用 TypeScript
- 支持 axios（SDK 基于 axios 生成）

## 常见问题

**Q: 生成的 SDK 方法名不符合预期？**
A: OpenAPI 的 `operationId` 字段决定方法名，可在文档中调整。

**Q: SDK 类型与现有类型冲突？**
A: 将优先使用 SDK 类型，需要手动调整代码中的类型引用。

**Q: 需要自定义 axios 配置（如超时、拦截器）？**
A: `swagger-typescript-api` 生成的 SDK 暴露了 axios 实例，可以直接配置：
```typescript
const api = new Api({
  baseURL: 'https://api.example.com',
  timeout: 10000, // 超时配置
});

// 添加请求拦截器
api.instance.interceptors.request.use(
  (config) => {
    // 在发送请求之前做些什么
    config.headers.Authorization = `Bearer ${token}`;
    return config;
  },
  (error) => {
    // 对请求错误做些什么
    return Promise.reject(error);
  }
);

// 添加响应拦截器
api.instance.interceptors.response.use(
  (response) => response,
  (error) => {
    // 统一处理错误
    if (error.response?.status === 401) {
      // 处理未授权
    }
    return Promise.reject(error);
  }
);
```

**Q: 生产环境需要使用 Proxy 吗？**
A: 不需要。Proxy 仅用于开发环境解决跨域问题。生产环境应确保 API 服务器支持 CORS，或使用同源部署策略。

**Q: Proxy 配置后请求仍然失败？**
A: 请检查：
1. 开发服务器是否重启（修改配置后需要重启）
2. 目标 API 地址是否可访问
3. `changeOrigin` 是否设置为 `true`
4. `rewrite` 路径规则是否正确

**Q: 使用 Tanstack Query 时如何保持原有的缓存策略？**
A: SDK 替换不会改变 `queryKey`，因此缓存策略完全保留。只需确保：
1. `queryKey` 保持不变
2. 返回的数据结构保持一致（必要时使用 `select` 转换）

**Q: 使用 SWR 时如何处理全局配置？**
A: 在 `SWRConfig` 中定义的 `fetcher`、`revalidateOnFocus` 等配置仍然有效。SDK 替换只影响单个 hook 的 fetcher 函数。

**Q: RTK Query 的自动刷新（tag-based invalidation）还能用吗？**
A: 可以。保持 `providesTags` 和 `invalidatesTags` 配置不变即可。

**Q: 项目从 GraphQL 迁移到 REST API，如何处理 Apollo Client？**
A: 有以下选择：
1. **推荐**：切换到 Tanstack Query 或 SWR（更好的 REST API 支持）
2. 使用 `apollo-link-rest` 让 Apollo Client 调用 REST API
3. 如果后端同时提供 GraphQL，使用 GraphQL 客户端生成工具

**Q: 如何批量转换多个数据获取 hook？**
A: 按照映射表格，为每个 API 端点：
1. 找到对应的 hook 文件
2. 根据选择的数据获取方案应用转换模板
3. 批量替换所有相关 import 语句
