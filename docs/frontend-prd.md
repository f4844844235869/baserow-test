# Baserow 前端开发文档（Nuxt 3 + TypeScript 版）

> **版本：** 1.0  
> **状态：** 正式  
> **最后更新：** 2026-03-16  
> **面向读者：** 前端工程师  
> **对应后端文档：** [backend-prd.md](./backend-prd.md)

---

## 目录

1. [技术栈与依赖](#1-技术栈与依赖)
2. [项目结构](#2-项目结构)
3. [TypeScript 类型定义](#3-typescript-类型定义)
4. [API Service 层](#4-api-service-层)
5. [Pinia Store 设计](#5-pinia-store-设计)
6. [路由设计](#6-路由设计)
7. [组件设计规范](#7-组件设计规范)
8. [核心页面与组件](#8-核心页面与组件)
9. [WebSocket 实时通信](#9-websocket-实时通信)
10. [认证流程](#10-认证流程)
11. [错误处理规范](#11-错误处理规范)
12. [测试规范](#12-测试规范)
13. [环境配置](#13-环境配置)

---

## 1. 技术栈与依赖

### 核心依赖

```json
{
  "dependencies": {
    "nuxt": "^3.12",
    "vue": "^3.4",
    "@pinia/nuxt": "^0.5",
    "pinia": "^2.1",
    "@nuxtjs/i18n": "^8.3",
    "axios": "^1.7",
    "@vueuse/core": "^10.11",
    "chart.js": "^4.4",
    "vue-chartjs": "^5.3",
    "date-fns": "^3.6",
    "@headlessui/vue": "^1.7",
    "lucide-vue-next": "^0.395"
  },
  "devDependencies": {
    "typescript": "^5.4",
    "@vue/test-utils": "^2.4",
    "vitest": "^1.6",
    "@testing-library/vue": "^8.1",
    "@nuxt/test-utils": "^3.13",
    "msw": "^2.3",
    "eslint": "^9.5",
    "@nuxt/eslint": "^0.3"
  }
}
```

### 技术选型说明

| 技术 | 选型理由 |
|------|----------|
| **Nuxt 3** | SSR/SSG 支持，模块化架构，TypeScript 原生支持 |
| **Pinia** | Vue 3 官方推荐状态管理，比 Vuex 更简洁，TypeScript 类型推断更好 |
| **Axios** | 成熟的 HTTP 客户端，支持拦截器（统一处理 Token 刷新） |
| **@vueuse/core** | 常用 Composition API 工具集，减少重复代码 |
| **Chart.js + vue-chartjs** | 仪表板图表渲染 |
| **@headlessui/vue** | 无样式 UI 组件（Modal、Dropdown 等），完全可定制 |

---

## 2. 项目结构

```
web-frontend/
├── pages/                          # Nuxt 路由页面（文件即路由）
│   ├── index.vue                   # 重定向到 /login 或 /workspaces
│   ├── login.vue                   # 登录页
│   ├── register.vue                # 注册页
│   ├── workspaces/
│   │   ├── index.vue               # 工作区列表
│   │   └── [workspaceId]/
│   │       ├── index.vue           # 工作区首页（应用列表）
│   │       └── settings.vue        # 工作区设置
│   └── database/
│       └── [tableId]/
│           └── [viewId].vue        # 表格/视图页面
│
├── components/
│   ├── layout/
│   │   ├── AppSidebar.vue          # 侧边栏（工作区/应用列表）
│   │   ├── AppHeader.vue           # 顶部导航
│   │   └── AppLayout.vue          # 主布局容器
│   ├── workspace/
│   │   ├── WorkspaceCard.vue       # 工作区卡片
│   │   ├── WorkspaceCreateModal.vue
│   │   ├── WorkspaceSettings.vue
│   │   ├── MemberList.vue
│   │   ├── InviteMemberModal.vue
│   │   └── MemberRoleSelect.vue
│   ├── database/
│   │   ├── grid/
│   │   │   ├── GridView.vue        # Grid 视图（虚拟滚动）
│   │   │   ├── GridRow.vue         # 行组件
│   │   │   ├── GridCell.vue        # 单元格组件
│   │   │   └── GridHeader.vue      # 列头组件
│   │   ├── views/
│   │   │   ├── ViewSelector.vue    # 视图切换器
│   │   │   ├── KanbanView.vue
│   │   │   ├── GalleryView.vue
│   │   │   └── FormView.vue
│   │   ├── fields/
│   │   │   ├── FieldTypeIcon.vue
│   │   │   ├── FieldCreateModal.vue
│   │   │   ├── FieldEditModal.vue
│   │   │   └── cells/              # 各类型单元格渲染组件
│   │   │       ├── TextCell.vue
│   │   │       ├── NumberCell.vue
│   │   │       ├── DateCell.vue
│   │   │       ├── SelectCell.vue
│   │   │       ├── FileCell.vue
│   │   │       └── BooleanCell.vue
│   │   ├── filters/
│   │   │   ├── FilterPanel.vue
│   │   │   └── FilterRow.vue
│   │   ├── sorts/
│   │   │   └── SortPanel.vue
│   │   └── RowDetailModal.vue      # 行详情弹窗
│   ├── builder/
│   │   ├── BuilderCanvas.vue       # 拖拽画布
│   │   ├── ElementPanel.vue        # 元素面板
│   │   ├── PropertyPanel.vue       # 属性配置面板
│   │   └── elements/
│   │       ├── HeadingElement.vue
│   │       ├── TextElement.vue
│   │       ├── ButtonElement.vue
│   │       ├── TableElement.vue
│   │       └── FormElement.vue
│   ├── dashboard/
│   │   ├── DashboardGrid.vue       # Widget 布局容器
│   │   ├── WidgetAddModal.vue
│   │   └── widgets/
│   │       ├── SummaryWidget.vue
│   │       └── ChartWidget.vue
│   ├── automation/
│   │   ├── AutomationEditor.vue
│   │   ├── TriggerConfig.vue
│   │   └── ActionConfig.vue
│   └── ui/                         # 通用 UI 组件
│       ├── BaseButton.vue
│       ├── BaseInput.vue
│       ├── BaseSelect.vue
│       ├── BaseModal.vue
│       ├── BaseDropdown.vue
│       ├── BaseToast.vue
│       ├── BaseTable.vue
│       ├── BasePagination.vue
│       ├── BaseEmptyState.vue
│       └── BaseSpinner.vue
│
├── stores/                         # Pinia Store
│   ├── auth.ts
│   ├── workspace.ts
│   ├── application.ts
│   ├── table.ts
│   ├── field.ts
│   ├── row.ts
│   ├── view.ts
│   ├── builder.ts
│   ├── dashboard.ts
│   ├── automation.ts
│   └── notification.ts             # Toast 通知状态
│
├── services/                       # API 调用服务（对应后端接口）
│   ├── http.ts                     # Axios 实例（含拦截器）
│   ├── authService.ts
│   ├── workspaceService.ts
│   ├── applicationService.ts
│   ├── tableService.ts
│   ├── fieldService.ts
│   ├── rowService.ts
│   ├── viewService.ts
│   ├── builderService.ts
│   ├── dashboardService.ts
│   └── automationService.ts
│
├── types/                          # TypeScript 类型定义（与后端 Schema 严格对应）
│   ├── auth.ts
│   ├── workspace.ts
│   ├── application.ts
│   ├── table.ts
│   ├── field.ts
│   ├── row.ts
│   ├── view.ts
│   ├── builder.ts
│   ├── dashboard.ts
│   ├── automation.ts
│   └── websocket.ts
│
├── composables/                    # 复用逻辑
│   ├── useAuth.ts
│   ├── useWorkspace.ts
│   ├── useTableRows.ts             # 行数据管理（含虚拟滚动）
│   ├── useWebSocket.ts             # WebSocket 连接管理
│   ├── useNotification.ts          # Toast 通知
│   └── useFieldRenderer.ts        # 字段渲染策略
│
├── middleware/
│   ├── auth.ts                     # 路由守卫（未登录重定向）
│   └── workspace.ts                # 工作区权限检查
│
├── plugins/
│   └── websocket.client.ts         # WebSocket 客户端插件（仅浏览器）
│
├── utils/
│   ├── fieldUtils.ts
│   ├── dateUtils.ts
│   └── colorUtils.ts
│
├── nuxt.config.ts
├── tsconfig.json
└── package.json
```

---

## 3. TypeScript 类型定义

所有类型严格对应后端 `docs/backend-prd.md` 中的 Pydantic Schema。

### 3.1 认证类型

```typescript
// types/auth.ts

export interface User {
  id: string
  email: string
  first_name: string
  last_name: string
  language: string
  is_staff: boolean
  created_at: string
}

export interface AuthResponse {
  user: User
  access_token: string
  refresh_token: string
  token_type: string
}

export interface TokenResponse {
  access_token: string
  refresh_token: string
  token_type: string
}

export interface LoginRequest {
  email: string
  password: string
}

export interface RegisterRequest {
  email: string
  password: string
  first_name: string
  last_name: string
  language?: string
}

export interface ChangePasswordRequest {
  old_password: string
  new_password: string
}

export interface UpdateUserRequest {
  first_name?: string
  last_name?: string
  language?: string
}
```

### 3.2 工作区类型

```typescript
// types/workspace.ts

export type WorkspaceRole = 'ADMIN' | 'MEMBER'

export interface Workspace {
  id: number
  name: string
  my_role: WorkspaceRole
  members_count: number
  created_at: string
  updated_at: string
}

export interface WorkspaceMember {
  id: number
  user_id: string
  email: string
  first_name: string
  last_name: string
  role: WorkspaceRole
  created_at: string
}

export interface WorkspaceInvitation {
  id: number
  email: string
  role: WorkspaceRole
  workspace_id: number
  invited_by: string
  expires_at: string
  created_at: string
}

export interface CreateWorkspaceRequest {
  name: string
}

export interface UpdateWorkspaceRequest {
  name: string
}

export interface InviteMemberRequest {
  email: string
  role: WorkspaceRole
  message?: string
}

export interface UpdateMemberRoleRequest {
  role: WorkspaceRole
}
```

### 3.3 应用类型

```typescript
// types/application.ts

export type ApplicationType = 'database' | 'builder' | 'dashboard' | 'automation'

export interface Application {
  id: number
  type: ApplicationType
  name: string
  order: number
  workspace_id: number
  created_at: string
  updated_at: string
}

export interface CreateApplicationRequest {
  type: ApplicationType
  name: string
}

export interface UpdateApplicationRequest {
  name?: string
}

export interface OrderApplicationsRequest {
  applications: number[]
}
```

### 3.4 表类型

```typescript
// types/table.ts

import type { Field } from './field'
import type { View } from './view'

export interface Table {
  id: number
  name: string
  order: number
  database_id: number
  created_at: string
  updated_at: string
}

export interface TableDetail extends Table {
  fields: Field[]
  views: View[]
}

export interface CreateTableRequest {
  name: string
  default_fields?: boolean
}

export interface UpdateTableRequest {
  name?: string
}

export interface OrderTablesRequest {
  tables: number[]
}
```

### 3.5 字段类型

```typescript
// types/field.ts

export type FieldType =
  | 'text'
  | 'long_text'
  | 'number'
  | 'rating'
  | 'boolean'
  | 'date'
  | 'last_modified_date'
  | 'created_on'
  | 'url'
  | 'email'
  | 'phone_number'
  | 'single_select'
  | 'multiple_select'
  | 'link_row'
  | 'file'
  | 'formula'
  | 'lookup'
  | 'count'
  | 'rollup'
  | 'collaborator'
  | 'uuid'
  | 'password'
  | 'ai'

export interface SelectOption {
  id?: number
  value: string
  color: string
}

// 各类型 config
export interface TextFieldConfig {
  text_default?: string
}

export interface NumberFieldConfig {
  number_decimal_places?: number
  number_negative?: boolean
  number_prefix?: string
  number_suffix?: string
}

export interface DateFieldConfig {
  date_format?: 'ISO' | 'US' | 'EU'
  date_include_time?: boolean
  date_time_format?: '24' | '12'
  date_show_tzinfo?: boolean
  date_force_timezone?: string | null
}

export interface SelectFieldConfig {
  select_options: SelectOption[]
}

export interface LinkRowFieldConfig {
  link_row_table_id: number
  link_row_related_field_id?: number | null
}

export interface RatingFieldConfig {
  max_value?: number
  color?: string
  style?: 'star' | 'heart' | 'thumbs-up' | 'flag'
}

export type FieldConfig =
  | TextFieldConfig
  | NumberFieldConfig
  | DateFieldConfig
  | SelectFieldConfig
  | LinkRowFieldConfig
  | RatingFieldConfig
  | Record<string, unknown>

export interface Field {
  id: number
  table_id: number
  type: FieldType
  name: string
  primary: boolean
  order: number
  config: FieldConfig
}

export interface CreateFieldRequest {
  type: FieldType
  name: string
  config?: FieldConfig
}

export interface UpdateFieldRequest {
  name?: string
  config?: FieldConfig
}

export interface OrderFieldsRequest {
  fields: number[]
}
```

### 3.6 行类型

```typescript
// types/row.ts

export interface Row {
  id: number
  order: string
  created_on: string
  updated_on: string
  [key: `field_${number}`]: unknown  // 动态字段
}

export interface PaginatedRows {
  count: number
  next: string | null
  previous: string | null
  results: Row[]
}

export interface RowQueryParams {
  page?: number
  size?: number
  search?: string
  order_by?: string
  view_id?: number
  include_fields?: string
  [key: string]: string | number | undefined  // 动态 filter 参数
}

export type RowCreateRequest = Record<string, unknown>
export type RowUpdateRequest = Record<string, unknown>

export interface BatchCreateRequest {
  rows: Record<string, unknown>[]
}

export interface BatchUpdateRequest {
  rows: Array<{ id: number } & Record<string, unknown>>
}

export interface BatchDeleteRequest {
  row_ids: number[]
}

export interface BatchRowsResponse {
  items: Row[]
}
```

### 3.7 视图类型

```typescript
// types/view.ts

export type ViewType = 'grid' | 'form' | 'gallery' | 'kanban' | 'calendar' | 'timeline'

export type FilterType =
  | 'equal' | 'not_equal'
  | 'contains' | 'contains_not'
  | 'empty' | 'not_empty'
  | 'higher_than' | 'lower_than'
  | 'date_equals' | 'date_before' | 'date_after'
  | 'link_row_has' | 'link_row_has_not'
  | 'multiple_select_has'

export type SortOrder = 'ASC' | 'DESC'

export interface ViewFilter {
  id: number
  field_id: number
  type: FilterType
  value: string
}

export interface ViewSort {
  id: number
  field_id: number
  order: SortOrder
}

export interface ViewFieldOption {
  field_id: number
  hidden: boolean
  order: number
  width?: number
}

export interface View {
  id: number
  table_id: number
  type: ViewType
  name: string
  order: number
  filter_type: 'AND' | 'OR'
  filters_disabled: boolean
  config: Record<string, unknown>
  created_at: string
  updated_at: string
}

export interface ViewDetail extends View {
  filters: ViewFilter[]
  sorts: ViewSort[]
  field_options: ViewFieldOption[]
}

export interface CreateViewRequest {
  type: ViewType
  name: string
  config?: Record<string, unknown>
}

export interface UpdateViewRequest {
  name?: string
  filter_type?: 'AND' | 'OR'
  filters_disabled?: boolean
}

export interface CreateFilterRequest {
  field_id: number
  type: FilterType
  value: string
}

export interface UpdateFilterRequest {
  field_id?: number
  type?: FilterType
  value?: string
}

export interface CreateSortRequest {
  field_id: number
  order: SortOrder
}

export interface UpdateFieldOptionsRequest {
  field_options: Record<string, Partial<ViewFieldOption>>
}
```

### 3.8 WebSocket 类型

```typescript
// types/websocket.ts

export type WebSocketEventType =
  | 'row_created' | 'row_updated' | 'row_deleted'
  | 'rows_created' | 'rows_updated' | 'rows_deleted'
  | 'field_created' | 'field_updated' | 'field_deleted'
  | 'view_created' | 'view_updated' | 'view_deleted'
  | 'table_created' | 'table_updated' | 'table_deleted'
  | 'application_created' | 'application_updated' | 'application_deleted'
  | 'pong'

export interface WebSocketMessage {
  type: WebSocketEventType
  workspace_id: number
  table_id?: number
  user_id: string
  data: Record<string, unknown>
}

export interface SubscribeTableMessage {
  type: 'subscribe_table'
  table_id: number
}

export interface UnsubscribeTableMessage {
  type: 'unsubscribe_table'
  table_id: number
}

export interface PingMessage {
  type: 'ping'
}
```

---

## 4. API Service 层

### 4.1 HTTP 客户端（含 Token 刷新拦截器）

```typescript
// services/http.ts

import axios, { type AxiosInstance, type InternalAxiosRequestConfig } from 'axios'
import type { TokenResponse } from '~/types/auth'

const BASE_URL = useRuntimeConfig().public.apiBase || 'http://localhost:8000/api/v1'

export const http: AxiosInstance = axios.create({
  baseURL: BASE_URL,
  headers: { 'Content-Type': 'application/json' },
  timeout: 30000,
})

// 请求拦截器：注入 Access Token
http.interceptors.request.use((config: InternalAxiosRequestConfig) => {
  const token = localStorage.getItem('access_token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 响应拦截器：Token 过期自动刷新
let isRefreshing = false
let refreshSubscribers: Array<(token: string) => void> = []

function subscribeTokenRefresh(callback: (token: string) => void) {
  refreshSubscribers.push(callback)
}

function onTokenRefreshed(newToken: string) {
  refreshSubscribers.forEach((cb) => cb(newToken))
  refreshSubscribers = []
}

http.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config
    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        return new Promise((resolve) => {
          subscribeTokenRefresh((token) => {
            originalRequest.headers.Authorization = `Bearer ${token}`
            resolve(http(originalRequest))
          })
        })
      }
      originalRequest._retry = true
      isRefreshing = true
      try {
        const refreshToken = localStorage.getItem('refresh_token')
        if (!refreshToken) throw new Error('No refresh token')
        const { data } = await axios.post<TokenResponse>(
          `${BASE_URL}/auth/token/refresh`,
          { refresh_token: refreshToken }
        )
        localStorage.setItem('access_token', data.access_token)
        localStorage.setItem('refresh_token', data.refresh_token)
        onTokenRefreshed(data.access_token)
        originalRequest.headers.Authorization = `Bearer ${data.access_token}`
        return http(originalRequest)
      } catch (refreshError) {
        localStorage.removeItem('access_token')
        localStorage.removeItem('refresh_token')
        navigateTo('/login')
        return Promise.reject(refreshError)
      } finally {
        isRefreshing = false
      }
    }
    return Promise.reject(error)
  }
)
```

### 4.2 认证 Service

```typescript
// services/authService.ts

import { http } from './http'
import type {
  LoginRequest, RegisterRequest, AuthResponse,
  TokenResponse, User, ChangePasswordRequest, UpdateUserRequest
} from '~/types/auth'

export const authService = {
  async register(data: RegisterRequest): Promise<AuthResponse> {
    const res = await http.post<AuthResponse>('/auth/register', data)
    return res.data
  },

  async login(data: LoginRequest): Promise<AuthResponse> {
    const res = await http.post<AuthResponse>('/auth/login', data)
    return res.data
  },

  async refreshToken(refreshToken: string): Promise<TokenResponse> {
    const res = await http.post<TokenResponse>('/auth/token/refresh', { refresh_token: refreshToken })
    return res.data
  },

  async logout(refreshToken: string): Promise<void> {
    await http.post('/auth/logout', { refresh_token: refreshToken })
  },

  async getMe(): Promise<User> {
    const res = await http.get<User>('/auth/me')
    return res.data
  },

  async updateMe(data: UpdateUserRequest): Promise<User> {
    const res = await http.patch<User>('/auth/me', data)
    return res.data
  },

  async changePassword(data: ChangePasswordRequest): Promise<void> {
    await http.post('/auth/change-password', data)
  },
}
```

### 4.3 工作区 Service

```typescript
// services/workspaceService.ts

import { http } from './http'
import type {
  Workspace, WorkspaceMember, WorkspaceInvitation,
  CreateWorkspaceRequest, UpdateWorkspaceRequest,
  InviteMemberRequest, UpdateMemberRoleRequest
} from '~/types/workspace'

export const workspaceService = {
  async list(): Promise<{ workspaces: Workspace[] }> {
    const res = await http.get('/workspaces')
    return res.data
  },

  async create(data: CreateWorkspaceRequest): Promise<Workspace> {
    const res = await http.post<Workspace>('/workspaces', data)
    return res.data
  },

  async get(id: number): Promise<Workspace> {
    const res = await http.get<Workspace>(`/workspaces/${id}`)
    return res.data
  },

  async update(id: number, data: UpdateWorkspaceRequest): Promise<Workspace> {
    const res = await http.patch<Workspace>(`/workspaces/${id}`, data)
    return res.data
  },

  async delete(id: number): Promise<void> {
    await http.delete(`/workspaces/${id}`)
  },

  async listMembers(id: number): Promise<{ members: WorkspaceMember[] }> {
    const res = await http.get(`/workspaces/${id}/members`)
    return res.data
  },

  async invite(id: number, data: InviteMemberRequest): Promise<WorkspaceInvitation> {
    const res = await http.post<WorkspaceInvitation>(`/workspaces/${id}/invitations`, data)
    return res.data
  },

  async updateMemberRole(workspaceId: number, userId: string, data: UpdateMemberRoleRequest): Promise<WorkspaceMember> {
    const res = await http.patch<WorkspaceMember>(`/workspaces/${workspaceId}/members/${userId}`, data)
    return res.data
  },

  async removeMember(workspaceId: number, userId: string): Promise<void> {
    await http.delete(`/workspaces/${workspaceId}/members/${userId}`)
  },

  async leaveWorkspace(workspaceId: number): Promise<void> {
    await http.delete(`/workspaces/${workspaceId}/members/me`)
  },
}
```

### 4.4 行 Service

```typescript
// services/rowService.ts

import { http } from './http'
import type {
  Row, PaginatedRows, RowQueryParams,
  RowCreateRequest, RowUpdateRequest,
  BatchCreateRequest, BatchUpdateRequest, BatchDeleteRequest, BatchRowsResponse
} from '~/types/row'

export const rowService = {
  async list(tableId: number, params: RowQueryParams = {}): Promise<PaginatedRows> {
    const res = await http.get<PaginatedRows>(`/tables/${tableId}/rows`, { params })
    return res.data
  },

  async create(tableId: number, data: RowCreateRequest, params?: { before_row_id?: number; user_field_names?: boolean }): Promise<Row> {
    const res = await http.post<Row>(`/tables/${tableId}/rows`, data, { params })
    return res.data
  },

  async get(tableId: number, rowId: number): Promise<Row> {
    const res = await http.get<Row>(`/tables/${tableId}/rows/${rowId}`)
    return res.data
  },

  async update(tableId: number, rowId: number, data: RowUpdateRequest): Promise<Row> {
    const res = await http.patch<Row>(`/tables/${tableId}/rows/${rowId}`, data)
    return res.data
  },

  async delete(tableId: number, rowId: number): Promise<void> {
    await http.delete(`/tables/${tableId}/rows/${rowId}`)
  },

  async batchCreate(tableId: number, data: BatchCreateRequest): Promise<BatchRowsResponse> {
    const res = await http.post<BatchRowsResponse>(`/tables/${tableId}/rows/batch`, data)
    return res.data
  },

  async batchUpdate(tableId: number, data: BatchUpdateRequest): Promise<BatchRowsResponse> {
    const res = await http.patch<BatchRowsResponse>(`/tables/${tableId}/rows/batch`, data)
    return res.data
  },

  async batchDelete(tableId: number, data: BatchDeleteRequest): Promise<void> {
    await http.delete(`/tables/${tableId}/rows/batch`, { data })
  },

  async import(tableId: number, file: File, format: string): Promise<{ task_id: string }> {
    const formData = new FormData()
    formData.append('file', file)
    formData.append('format', format)
    const res = await http.post(`/tables/${tableId}/rows/import`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    })
    return res.data
  },

  getExportUrl(tableId: number, format: string, viewId?: number): string {
    const params = new URLSearchParams({ format })
    if (viewId) params.append('view_id', String(viewId))
    return `${http.defaults.baseURL}/tables/${tableId}/rows/export?${params.toString()}`
  },
}
```

### 4.5 视图 Service

```typescript
// services/viewService.ts

import { http } from './http'
import type {
  View, ViewDetail, CreateViewRequest, UpdateViewRequest,
  ViewFilter, CreateFilterRequest, UpdateFilterRequest,
  ViewSort, CreateSortRequest, UpdateFieldOptionsRequest
} from '~/types/view'

export const viewService = {
  async list(tableId: number): Promise<{ views: View[] }> {
    const res = await http.get(`/tables/${tableId}/views`)
    return res.data
  },

  async create(tableId: number, data: CreateViewRequest): Promise<View> {
    const res = await http.post<View>(`/tables/${tableId}/views`, data)
    return res.data
  },

  async get(viewId: number): Promise<ViewDetail> {
    const res = await http.get<ViewDetail>(`/views/${viewId}`)
    return res.data
  },

  async update(viewId: number, data: UpdateViewRequest): Promise<View> {
    const res = await http.patch<View>(`/views/${viewId}`, data)
    return res.data
  },

  async delete(viewId: number): Promise<void> {
    await http.delete(`/views/${viewId}`)
  },

  async createFilter(viewId: number, data: CreateFilterRequest): Promise<ViewFilter> {
    const res = await http.post<ViewFilter>(`/views/${viewId}/filters`, data)
    return res.data
  },

  async updateFilter(viewId: number, filterId: number, data: UpdateFilterRequest): Promise<ViewFilter> {
    const res = await http.patch<ViewFilter>(`/views/${viewId}/filters/${filterId}`, data)
    return res.data
  },

  async deleteFilter(viewId: number, filterId: number): Promise<void> {
    await http.delete(`/views/${viewId}/filters/${filterId}`)
  },

  async createSort(viewId: number, data: CreateSortRequest): Promise<ViewSort> {
    const res = await http.post<ViewSort>(`/views/${viewId}/sorts`, data)
    return res.data
  },

  async deleteSort(viewId: number, sortId: number): Promise<void> {
    await http.delete(`/views/${viewId}/sorts/${sortId}`)
  },

  async updateFieldOptions(viewId: number, data: UpdateFieldOptionsRequest): Promise<void> {
    await http.patch(`/views/${viewId}/field-options`, data)
  },
}
```

---

## 5. Pinia Store 设计

### 5.1 认证 Store

```typescript
// stores/auth.ts

import { defineStore } from 'pinia'
import { authService } from '~/services/authService'
import type { User, LoginRequest, RegisterRequest } from '~/types/auth'

export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)
  const isAuthenticated = computed(() => !!user.value)

  async function login(credentials: LoginRequest) {
    const data = await authService.login(credentials)
    user.value = data.user
    localStorage.setItem('access_token', data.access_token)
    localStorage.setItem('refresh_token', data.refresh_token)
  }

  async function register(data: RegisterRequest) {
    const res = await authService.register(data)
    user.value = res.user
    localStorage.setItem('access_token', res.access_token)
    localStorage.setItem('refresh_token', res.refresh_token)
  }

  async function logout() {
    const refreshToken = localStorage.getItem('refresh_token')
    if (refreshToken) {
      await authService.logout(refreshToken).catch(() => {})
    }
    user.value = null
    localStorage.removeItem('access_token')
    localStorage.removeItem('refresh_token')
    navigateTo('/login')
  }

  async function fetchMe() {
    const data = await authService.getMe()
    user.value = data
  }

  return { user, isAuthenticated, login, register, logout, fetchMe }
})
```

### 5.2 工作区 Store

```typescript
// stores/workspace.ts

import { defineStore } from 'pinia'
import { workspaceService } from '~/services/workspaceService'
import type { Workspace, WorkspaceMember, CreateWorkspaceRequest } from '~/types/workspace'

export const useWorkspaceStore = defineStore('workspace', () => {
  const workspaces = ref<Workspace[]>([])
  const currentWorkspace = ref<Workspace | null>(null)
  const members = ref<WorkspaceMember[]>([])
  const loading = ref(false)

  async function fetchWorkspaces() {
    loading.value = true
    try {
      const data = await workspaceService.list()
      workspaces.value = data.workspaces
    } finally {
      loading.value = false
    }
  }

  async function createWorkspace(data: CreateWorkspaceRequest) {
    const workspace = await workspaceService.create(data)
    workspaces.value.push(workspace)
    return workspace
  }

  async function deleteWorkspace(id: number) {
    await workspaceService.delete(id)
    workspaces.value = workspaces.value.filter((w) => w.id !== id)
    if (currentWorkspace.value?.id === id) {
      currentWorkspace.value = null
    }
  }

  async function selectWorkspace(id: number) {
    const workspace = await workspaceService.get(id)
    currentWorkspace.value = workspace
  }

  // 由 WebSocket 事件调用
  function handleWorkspaceUpdated(workspaceId: number, data: Partial<Workspace>) {
    const idx = workspaces.value.findIndex((w) => w.id === workspaceId)
    if (idx !== -1) {
      workspaces.value[idx] = { ...workspaces.value[idx], ...data }
    }
  }

  return {
    workspaces, currentWorkspace, members, loading,
    fetchWorkspaces, createWorkspace, deleteWorkspace, selectWorkspace,
    handleWorkspaceUpdated,
  }
})
```

### 5.3 行 Store（含虚拟滚动支持）

```typescript
// stores/row.ts

import { defineStore } from 'pinia'
import { rowService } from '~/services/rowService'
import type { Row, PaginatedRows, RowQueryParams } from '~/types/row'

export const useRowStore = defineStore('row', () => {
  // 按 tableId 缓存行数据
  const rowsByTable = ref<Record<number, Row[]>>({})
  const totalByTable = ref<Record<number, number>>({})
  const loadingByTable = ref<Record<number, boolean>>({})

  function getRows(tableId: number): Row[] {
    return rowsByTable.value[tableId] ?? []
  }

  function getTotal(tableId: number): number {
    return totalByTable.value[tableId] ?? 0
  }

  async function fetchRows(tableId: number, params: RowQueryParams = {}) {
    loadingByTable.value[tableId] = true
    try {
      const data = await rowService.list(tableId, params)
      rowsByTable.value[tableId] = data.results
      totalByTable.value[tableId] = data.count
      return data
    } finally {
      loadingByTable.value[tableId] = false
    }
  }

  async function createRow(tableId: number, values: Record<string, unknown>): Promise<Row> {
    const row = await rowService.create(tableId, values)
    rowsByTable.value[tableId] = [row, ...(rowsByTable.value[tableId] ?? [])]
    return row
  }

  async function updateRow(tableId: number, rowId: number, values: Record<string, unknown>): Promise<Row> {
    const row = await rowService.update(tableId, rowId, values)
    const rows = rowsByTable.value[tableId] ?? []
    const idx = rows.findIndex((r) => r.id === rowId)
    if (idx !== -1) rows[idx] = row
    return row
  }

  async function deleteRow(tableId: number, rowId: number): Promise<void> {
    await rowService.delete(tableId, rowId)
    rowsByTable.value[tableId] = (rowsByTable.value[tableId] ?? []).filter((r) => r.id !== rowId)
  }

  // WebSocket 事件处理
  function handleRowCreated(tableId: number, row: Row) {
    const rows = rowsByTable.value[tableId] ?? []
    if (!rows.find((r) => r.id === row.id)) {
      rowsByTable.value[tableId] = [row, ...rows]
      totalByTable.value[tableId] = (totalByTable.value[tableId] ?? 0) + 1
    }
  }

  function handleRowUpdated(tableId: number, rowId: number, values: Record<string, unknown>) {
    const rows = rowsByTable.value[tableId] ?? []
    const idx = rows.findIndex((r) => r.id === rowId)
    if (idx !== -1) {
      rows[idx] = { ...rows[idx], ...values }
    }
  }

  function handleRowDeleted(tableId: number, rowId: number) {
    rowsByTable.value[tableId] = (rowsByTable.value[tableId] ?? []).filter((r) => r.id !== rowId)
    totalByTable.value[tableId] = Math.max(0, (totalByTable.value[tableId] ?? 0) - 1)
  }

  return {
    rowsByTable, totalByTable, loadingByTable,
    getRows, getTotal, fetchRows,
    createRow, updateRow, deleteRow,
    handleRowCreated, handleRowUpdated, handleRowDeleted,
  }
})
```

---

## 6. 路由设计

### 页面路由映射

| URL 路径 | 页面文件 | 说明 | 需认证 |
|----------|----------|------|--------|
| `/` | `pages/index.vue` | 重定向到 `/workspaces` | ✅ |
| `/login` | `pages/login.vue` | 登录页 | ❌ |
| `/register` | `pages/register.vue` | 注册页 | ❌ |
| `/invite/:token` | `pages/invite/[token].vue` | 接受邀请页 | ❌ |
| `/workspaces` | `pages/workspaces/index.vue` | 工作区列表 | ✅ |
| `/workspaces/:id` | `pages/workspaces/[workspaceId]/index.vue` | 工作区（应用列表） | ✅ |
| `/workspaces/:id/settings` | `pages/workspaces/[workspaceId]/settings.vue` | 工作区设置 | ✅ ADMIN |
| `/database/:tableId/:viewId` | `pages/database/[tableId]/[viewId].vue` | 表格/视图主页面 | ✅ |
| `/builder/:appId/:pageId` | `pages/builder/[appId]/[pageId].vue` | Builder 编辑器 | ✅ |
| `/dashboard/:appId` | `pages/dashboard/[appId].vue` | 仪表板 | ✅ |
| `/automation/:appId` | `pages/automation/[appId].vue` | 自动化编辑器 | ✅ |
| `/public/builder/:appId/:path` | `pages/public/builder/[appId]/[...path].vue` | 公开 Builder 页面 | ❌ |

### 路由中间件

```typescript
// middleware/auth.ts

export default defineNuxtRouteMiddleware((to) => {
  const authStore = useAuthStore()
  const publicRoutes = ['/login', '/register']
  if (!authStore.isAuthenticated && !publicRoutes.some(r => to.path.startsWith(r))) {
    return navigateTo(`/login?redirect=${encodeURIComponent(to.fullPath)}`)
  }
})
```

---

## 7. 组件设计规范

### 7.1 组件模板结构

```vue
<script setup lang="ts">
// 1. 导入
import type { Workspace } from '~/types/workspace'

// 2. Props 定义
interface Props {
  workspace: Workspace
  isActive?: boolean
}
const props = withDefaults(defineProps<Props>(), {
  isActive: false,
})

// 3. Emits 定义
const emit = defineEmits<{
  select: [workspace: Workspace]
  delete: [id: number]
}>()

// 4. Store 和 Composable
const workspaceStore = useWorkspaceStore()

// 5. 响应式状态
const isDeleting = ref(false)

// 6. 计算属性
const displayName = computed(() => props.workspace.name.toUpperCase())

// 7. 方法
async function handleDelete() {
  isDeleting.value = true
  try {
    await workspaceStore.deleteWorkspace(props.workspace.id)
    emit('delete', props.workspace.id)
  } finally {
    isDeleting.value = false
  }
}
</script>

<template>
  <div :class="['workspace-card', { 'is-active': isActive }]">
    <h3>{{ displayName }}</h3>
    <button @click="handleDelete" :disabled="isDeleting">
      {{ isDeleting ? '删除中...' : '删除' }}
    </button>
  </div>
</template>
```

### 7.2 表单验证规范

```typescript
// 使用 @vueuse/core 的 useVModel + 本地验证逻辑
// 错误信息统一从后端错误码映射

const ERROR_MESSAGES: Record<string, string> = {
  ERROR_EMAIL_ALREADY_EXISTS: '该邮箱已注册，请直接登录',
  ERROR_INVALID_CREDENTIALS: '邮箱或密码不正确',
  ERROR_PASSWORD_TOO_SHORT: '密码长度至少 8 位',
  // ...
}

function parseApiError(error: unknown): string {
  if (axios.isAxiosError(error) && error.response?.data) {
    const code = error.response.data.error as string
    return ERROR_MESSAGES[code] ?? error.response.data.detail ?? '请求失败'
  }
  return '网络错误，请稍后重试'
}
```

### 7.3 Grid View 虚拟滚动规范

Grid View 中使用虚拟滚动以支持大数据量（10万+ 行）：

```typescript
// composables/useTableRows.ts

export function useVirtualRows(tableId: number, viewId: number) {
  const ROW_HEIGHT = 36  // px
  const BUFFER = 10      // 视口外额外渲染的行数

  const containerRef = ref<HTMLElement | null>(null)
  const scrollTop = ref(0)
  const containerHeight = ref(600)
  const rowStore = useRowStore()

  const visibleStart = computed(() =>
    Math.max(0, Math.floor(scrollTop.value / ROW_HEIGHT) - BUFFER)
  )
  const visibleEnd = computed(() =>
    Math.min(
      rowStore.getTotal(tableId),
      Math.ceil((scrollTop.value + containerHeight.value) / ROW_HEIGHT) + BUFFER
    )
  )
  const visibleRows = computed(() =>
    rowStore.getRows(tableId).slice(visibleStart.value, visibleEnd.value)
  )
  const topPadding = computed(() => visibleStart.value * ROW_HEIGHT)
  const bottomPadding = computed(() =>
    (rowStore.getTotal(tableId) - visibleEnd.value) * ROW_HEIGHT
  )

  function onScroll(e: Event) {
    scrollTop.value = (e.target as HTMLElement).scrollTop
  }

  return { containerRef, visibleRows, topPadding, bottomPadding, onScroll }
}
```

---

## 8. 核心页面与组件

### 8.1 数据库表格页面（Grid View）

**页面文件**: `pages/database/[tableId]/[viewId].vue`

**职责**:
1. 从 Store 加载表元数据（字段列表、视图配置）
2. 从 Store 加载行数据（第一页）
3. 渲染 Grid View（虚拟滚动）
4. 处理行内联编辑（点击单元格进入编辑态）
5. 响应 WebSocket 实时更新

**关键数据流**:
```
页面加载 → fetchTableDetail(tableId) → fieldStore + viewStore
       → fetchView(viewId) → viewStore.currentView（含 filters/sorts）
       → fetchRows(tableId, { view_id: viewId }) → rowStore

行编辑 → rowStore.updateRow() → PATCH /api/v1/tables/{id}/rows/{rowId}
      → WebSocket 推送 row_updated → rowStore.handleRowUpdated()
```

**所需 Props/Context**:
- `tableId: number` （路由参数）
- `viewId: number` （路由参数）

**布局**:
```
┌─────────────────────────────────────────────┐
│  ViewSelector | FilterPanel | SortPanel | +  │  ← 工具栏
├────┬────────────────────────────────────────┤
│    │  Field 1   │  Field 2   │  Field 3   │  │  ← 列头
│ S  ├────────────────────────────────────────┤
│ i  │  cell      │  cell      │  cell      │  │  ← 虚拟滚动区域
│ d  │  ...       │  ...       │  ...       │  │
│ e  │            │            │            │  │
│ b  ├────────────────────────────────────────┤
│ a  │  + 添加行                               │  ← 新增行
│ r  └────────────────────────────────────────┘
└────┘
```

### 8.2 Builder 编辑器页面

**页面文件**: `pages/builder/[appId]/[pageId].vue`

**布局**:
```
┌──────────────────────────────────────────────────┐
│  [元素面板]  │      [画布区域]       │  [属性面板]  │
│  heading     │                       │             │
│  text        │   ┌─────────────┐    │  选中元素的  │
│  button      │   │  Heading    │    │  属性配置    │
│  table       │   │  Text       │    │             │
│  form        │   │  Button     │    │             │
│  chart       │   └─────────────┘    │             │
│              │                       │             │
└──────────────┴───────────────────────┴─────────────┘
```

### 8.3 页面加载状态管理

所有页面必须处理三种状态：Loading / Error / Success

```vue
<template>
  <div>
    <BaseSpinner v-if="loading" />
    <BaseEmptyState v-else-if="error" :message="error" />
    <template v-else>
      <!-- 正常内容 -->
    </template>
  </div>
</template>
```

---

## 9. WebSocket 实时通信

### 9.1 连接管理

```typescript
// composables/useWebSocket.ts

export function useWebSocket(workspaceId: number) {
  const authStore = useAuthStore()
  const rowStore = useRowStore()
  const fieldStore = useFieldStore()

  let ws: WebSocket | null = null
  let pingInterval: ReturnType<typeof setInterval> | null = null
  let reconnectTimeout: ReturnType<typeof setTimeout> | null = null
  let reconnectAttempts = 0
  const MAX_RECONNECT = 5

  function connect() {
    const token = localStorage.getItem('access_token')
    const wsBase = useRuntimeConfig().public.wsBase || 'ws://localhost:8000'
    ws = new WebSocket(`${wsBase}/ws/workspaces/${workspaceId}?token=${token}`)

    ws.onopen = () => {
      reconnectAttempts = 0
      pingInterval = setInterval(() => {
        ws?.send(JSON.stringify({ type: 'ping' }))
      }, 30000)
    }

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data) as WebSocketMessage
      handleMessage(message)
    }

    ws.onclose = () => {
      if (pingInterval) clearInterval(pingInterval)
      if (reconnectAttempts < MAX_RECONNECT) {
        reconnectAttempts++
        reconnectTimeout = setTimeout(connect, 2000 * reconnectAttempts)
      }
    }

    ws.onerror = () => {
      ws?.close()
    }
  }

  function handleMessage(message: WebSocketMessage) {
    switch (message.type) {
      case 'row_created':
        rowStore.handleRowCreated(message.table_id!, message.data.row as Row)
        break
      case 'row_updated':
        rowStore.handleRowUpdated(message.table_id!, message.data.row_id as number, message.data.values as Record<string, unknown>)
        break
      case 'row_deleted':
        rowStore.handleRowDeleted(message.table_id!, message.data.row_id as number)
        break
      case 'field_created':
        fieldStore.handleFieldCreated(message.table_id!, message.data.field as Field)
        break
      case 'field_updated':
        fieldStore.handleFieldUpdated(message.table_id!, message.data.field_id as number, message.data.field as Partial<Field>)
        break
      case 'field_deleted':
        fieldStore.handleFieldDeleted(message.table_id!, message.data.field_id as number)
        break
      // ... 其他事件类型
    }
  }

  function subscribeTable(tableId: number) {
    ws?.send(JSON.stringify({ type: 'subscribe_table', table_id: tableId }))
  }

  function unsubscribeTable(tableId: number) {
    ws?.send(JSON.stringify({ type: 'unsubscribe_table', table_id: tableId }))
  }

  function disconnect() {
    if (pingInterval) clearInterval(pingInterval)
    if (reconnectTimeout) clearTimeout(reconnectTimeout)
    ws?.close()
  }

  onMounted(() => connect())
  onUnmounted(() => disconnect())

  return { subscribeTable, unsubscribeTable }
}
```

### 9.2 WebSocket 插件（Nuxt 客户端插件）

```typescript
// plugins/websocket.client.ts

export default defineNuxtPlugin(() => {
  const authStore = useAuthStore()
  const workspaceStore = useWorkspaceStore()

  // 当工作区切换时重连 WebSocket
  watch(
    () => workspaceStore.currentWorkspace?.id,
    (newId, oldId) => {
      if (oldId) {
        // 断开旧连接
      }
      if (newId) {
        useWebSocket(newId)
      }
    }
  )
})
```

---

## 10. 认证流程

### 10.1 登录流程

```
用户填写邮箱/密码
    ↓
authService.login()
    ↓ 成功
保存 access_token + refresh_token 到 localStorage
    ↓
authStore.user = 用户信息
    ↓
跳转到 /workspaces 或 redirect 参数中的页面
```

### 10.2 Token 自动刷新流程

```
API 请求返回 401
    ↓
Axios 拦截器检测到 401 且非 /auth/token/refresh 接口
    ↓
用 refresh_token 调用 POST /api/v1/auth/token/refresh
    ↓ 成功
更新 localStorage 中的 token
重放原始请求
    ↓ 失败（refresh_token 也过期）
清除 localStorage 中的 token
跳转到 /login 页面
```

### 10.3 路由守卫

```typescript
// middleware/auth.ts（已在第 6 节展示）
// 应用于所有需要认证的路由
```

---

## 11. 错误处理规范

### 11.1 API 错误处理统一模式

```typescript
// composables/useNotification.ts

export function useNotification() {
  const notifications = ref<Array<{ id: string; type: 'success' | 'error' | 'info'; message: string }>>([])

  function show(type: 'success' | 'error' | 'info', message: string) {
    const id = crypto.randomUUID()
    notifications.value.push({ id, type, message })
    setTimeout(() => remove(id), 5000)
  }

  function remove(id: string) {
    notifications.value = notifications.value.filter((n) => n.id !== id)
  }

  return { notifications, show, remove }
}
```

### 11.2 组件中的错误处理模式

```typescript
// 在组件或 Store 中统一处理
const notification = useNotification()

async function saveRow() {
  try {
    await rowStore.updateRow(tableId, rowId, values)
    notification.show('success', '已保存')
  } catch (error) {
    notification.show('error', parseApiError(error))
  }
}
```

### 11.3 错误码到用户提示映射

```typescript
// utils/errorMessages.ts

export const API_ERROR_MESSAGES: Record<string, string> = {
  ERROR_EMAIL_ALREADY_EXISTS: '该邮箱已注册，请直接登录',
  ERROR_PASSWORD_TOO_SHORT: '密码长度至少 8 位',
  ERROR_INVALID_CREDENTIALS: '邮箱或密码不正确',
  ERROR_DEACTIVATED_USER: '账户已被禁用，请联系管理员',
  ERROR_WORKSPACE_DOES_NOT_EXIST: '工作区不存在或已被删除',
  ERROR_NOT_A_MEMBER: '您没有访问此工作区的权限',
  ERROR_CANNOT_DELETE_PRIMARY_FIELD: '主字段不能被删除',
  ERROR_MAX_FIELD_COUNT_EXCEEDED: '已达到最大字段数限制（1500个）',
  ERROR_PERMISSION_DENIED: '您没有执行此操作的权限',
  ERROR_FILE_SIZE_TOO_LARGE: '文件大小超过限制（最大 100MB）',
}

export function parseApiError(error: unknown): string {
  if (axios.isAxiosError(error) && error.response?.data) {
    const code = error.response.data.error as string
    return API_ERROR_MESSAGES[code] ?? error.response.data.detail ?? '请求失败，请稍后重试'
  }
  if (error instanceof Error) return error.message
  return '网络错误，请检查网络连接'
}
```

---

## 12. 测试规范

### 12.1 Service 层单元测试（MSW Mock）

```typescript
// tests/unit/services/workspaceService.test.ts

import { setupServer } from 'msw/node'
import { http, HttpResponse } from 'msw'
import { workspaceService } from '~/services/workspaceService'
import { describe, it, expect, beforeAll, afterAll } from 'vitest'

const server = setupServer(
  http.get('/api/v1/workspaces', () => {
    return HttpResponse.json({
      workspaces: [
        { id: 1, name: '测试工作区', my_role: 'ADMIN', members_count: 1, created_at: '', updated_at: '' }
      ]
    })
  })
)

beforeAll(() => server.listen())
afterAll(() => server.close())

describe('workspaceService', () => {
  it('应该能获取工作区列表', async () => {
    const data = await workspaceService.list()
    expect(data.workspaces).toHaveLength(1)
    expect(data.workspaces[0].name).toBe('测试工作区')
  })
})
```

### 12.2 Store 单元测试

```typescript
// tests/unit/stores/workspace.test.ts

import { setActivePinia, createPinia } from 'pinia'
import { useWorkspaceStore } from '~/stores/workspace'
import { describe, it, expect, beforeEach, vi } from 'vitest'

vi.mock('~/services/workspaceService', () => ({
  workspaceService: {
    list: vi.fn().mockResolvedValue({
      workspaces: [{ id: 1, name: '工作区A', my_role: 'ADMIN', members_count: 1, created_at: '', updated_at: '' }]
    })
  }
}))

describe('useWorkspaceStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('fetchWorkspaces 应该填充 workspaces 列表', async () => {
    const store = useWorkspaceStore()
    await store.fetchWorkspaces()
    expect(store.workspaces).toHaveLength(1)
    expect(store.workspaces[0].name).toBe('工作区A')
  })
})
```

### 12.3 组件测试

```typescript
// tests/unit/components/WorkspaceCard.test.ts

import { mount } from '@vue/test-utils'
import WorkspaceCard from '~/components/workspace/WorkspaceCard.vue'
import { describe, it, expect } from 'vitest'

describe('WorkspaceCard', () => {
  const workspace = {
    id: 1, name: '测试工作区', my_role: 'ADMIN' as const,
    members_count: 3, created_at: '', updated_at: ''
  }

  it('应该显示工作区名称', () => {
    const wrapper = mount(WorkspaceCard, { props: { workspace } })
    expect(wrapper.text()).toContain('测试工作区')
  })

  it('点击删除按钮应该 emit delete 事件', async () => {
    const wrapper = mount(WorkspaceCard, { props: { workspace } })
    await wrapper.find('[data-test="delete-btn"]').trigger('click')
    expect(wrapper.emitted('delete')).toBeTruthy()
  })
})
```

---

## 13. 环境配置

### nuxt.config.ts

```typescript
// nuxt.config.ts

export default defineNuxtConfig({
  modules: [
    '@pinia/nuxt',
    '@nuxtjs/i18n',
    '@vueuse/nuxt',
    '@nuxt/test-utils/module',
  ],

  runtimeConfig: {
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_BASE || 'http://localhost:8000/api/v1',
      wsBase: process.env.NUXT_PUBLIC_WS_BASE || 'ws://localhost:8000',
    }
  },

  typescript: {
    strict: true,
    typeCheck: true,
  },

  i18n: {
    locales: [
      { code: 'zh-CN', file: 'zh-CN.json', name: '中文（简体）' },
      { code: 'en', file: 'en.json', name: 'English' },
    ],
    defaultLocale: 'zh-CN',
    langDir: 'locales/',
  },

  css: ['~/assets/styles/main.scss'],

  devtools: { enabled: true },
})
```

### 环境变量（.env）

```bash
# .env
NUXT_PUBLIC_API_BASE=http://localhost:8000/api/v1
NUXT_PUBLIC_WS_BASE=ws://localhost:8000
```

### .env.production

```bash
NUXT_PUBLIC_API_BASE=https://api.yourdomain.com/api/v1
NUXT_PUBLIC_WS_BASE=wss://api.yourdomain.com
```

---

## 附录：前后端接口对照表

| 功能 | 前端调用 | 后端接口 |
|------|----------|----------|
| 登录 | `authService.login()` | `POST /api/v1/auth/login` |
| 获取当前用户 | `authService.getMe()` | `GET /api/v1/auth/me` |
| 获取工作区列表 | `workspaceService.list()` | `GET /api/v1/workspaces` |
| 创建工作区 | `workspaceService.create()` | `POST /api/v1/workspaces` |
| 获取应用列表 | `applicationService.list(wsId)` | `GET /api/v1/workspaces/{id}/applications` |
| 获取表列表 | `tableService.list(appId)` | `GET /api/v1/applications/{id}/tables` |
| 获取字段列表 | `fieldService.list(tableId)` | `GET /api/v1/tables/{id}/fields` |
| 获取行数据 | `rowService.list(tableId, params)` | `GET /api/v1/tables/{id}/rows` |
| 创建行 | `rowService.create(tableId, data)` | `POST /api/v1/tables/{id}/rows` |
| 更新行 | `rowService.update(tableId, rowId, data)` | `PATCH /api/v1/tables/{id}/rows/{rowId}` |
| 删除行 | `rowService.delete(tableId, rowId)` | `DELETE /api/v1/tables/{id}/rows/{rowId}` |
| 获取视图详情 | `viewService.get(viewId)` | `GET /api/v1/views/{id}` |
| 添加过滤条件 | `viewService.createFilter(viewId, data)` | `POST /api/v1/views/{id}/filters` |
| WebSocket 连接 | `useWebSocket(workspaceId)` | `WS /ws/workspaces/{id}?token=...` |

---

*本文档由前端架构团队维护。如接口有变更请同步更新 [backend-prd.md](./backend-prd.md) 中对应的接口定义，并在本文档的附录对照表中更新。*
