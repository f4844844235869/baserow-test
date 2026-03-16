# Baserow 后端开发文档（FastAPI 版）

> **版本：** 1.0  
> **状态：** 正式  
> **最后更新：** 2026-03-16  
> **面向读者：** 后端工程师  
> **对应前端文档：** [frontend-prd.md](./frontend-prd.md)

---

## 目录

1. [技术栈与依赖](#1-技术栈与依赖)
2. [项目结构](#2-项目结构)
3. [数据库模型设计](#3-数据库模型设计)
4. [认证系统](#4-认证系统)
5. [API 接口定义（全量）](#5-api-接口定义全量)
   - 5.1 [认证接口](#51-认证接口)
   - 5.2 [工作区接口](#52-工作区接口)
   - 5.3 [应用接口](#53-应用接口)
   - 5.4 [表接口](#54-表接口)
   - 5.5 [字段接口](#55-字段接口)
   - 5.6 [行接口](#56-行接口)
   - 5.7 [视图接口](#57-视图接口)
   - 5.8 [Builder 接口](#58-builder-接口)
   - 5.9 [仪表板接口](#59-仪表板接口)
   - 5.10 [自动化接口](#510-自动化接口)
   - 5.11 [文件上传接口](#511-文件上传接口)
   - 5.12 [健康检查接口](#512-健康检查接口)
6. [WebSocket 协议](#6-websocket-协议)
7. [错误码定义](#7-错误码定义)
8. [Pydantic Schema 完整定义](#8-pydantic-schema-完整定义)
9. [Celery 异步任务](#9-celery-异步任务)
10. [配置管理](#10-配置管理)
11. [部署指南](#11-部署指南)

---

## 1. 技术栈与依赖

### 核心依赖

```toml
[tool.poetry.dependencies]
python = "^3.11"
fastapi = "^0.115"
uvicorn = { version = "^0.30", extras = ["standard"] }
pydantic = "^2.7"
pydantic-settings = "^2.3"
sqlalchemy = "^2.0"
alembic = "^1.13"
asyncpg = "^0.29"          # PostgreSQL async driver
redis = { version = "^5.0", extras = ["hiredis"] }
celery = "^5.4"
python-jose = { version = "^3.3", extras = ["cryptography"] }
passlib = { version = "^1.7", extras = ["bcrypt"] }
python-multipart = "^0.0.9"  # 文件上传
httpx = "^0.27"            # 内部 HTTP 客户端（Webhook、AI）
pillow = "^10.3"           # 图片处理
boto3 = "^1.34"            # S3 文件存储
langchain = "^0.2"         # AI 能力集成
opentelemetry-sdk = "^1.24"
sentry-sdk = { version = "^2.6", extras = ["fastapi"] }
```

### 开发依赖

```toml
[tool.poetry.group.dev.dependencies]
pytest = "^8.2"
pytest-asyncio = "^0.23"
pytest-cov = "^5.0"
httpx = "^0.27"            # 测试客户端
factory-boy = "^3.3"       # 测试工厂
faker = "^25.0"
ruff = "^0.4"              # Lint & Format
mypy = "^1.10"
```

---

## 2. 项目结构

```
backend/
├── app/
│   ├── main.py                     # FastAPI 应用入口，注册路由、中间件
│   ├── config.py                   # pydantic-settings 配置
│   ├── database.py                 # SQLAlchemy async engine & session
│   ├── dependencies.py             # 公共依赖注入（get_db, get_current_user）
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── router.py           # 汇总所有子路由
│   │       ├── auth.py
│   │       ├── workspaces.py
│   │       ├── applications.py
│   │       ├── tables.py
│   │       ├── fields.py
│   │       ├── rows.py
│   │       ├── views.py
│   │       ├── builder.py
│   │       ├── dashboard.py
│   │       ├── automation.py
│   │       ├── files.py
│   │       └── health.py
│   │
│   ├── models/
│   │   ├── base.py                 # Base declarative + TimestampMixin
│   │   ├── user.py                 # User, UserProfile
│   │   ├── workspace.py            # Workspace, WorkspaceMember, WorkspaceInvitation
│   │   ├── application.py          # Application（多态基类）
│   │   ├── table.py                # Table
│   │   ├── field.py                # Field（多态基类）+ 各子类型
│   │   ├── view.py                 # View（多态基类）+ GridView/GalleryView 等
│   │   ├── builder.py              # BuilderPage, BuilderElement
│   │   ├── dashboard.py            # DashboardWidget
│   │   ├── automation.py           # Automation, AutomationTrigger, AutomationAction
│   │   └── trash.py                # TrashEntry
│   │
│   ├── schemas/
│   │   ├── common.py               # 公共 Schema（分页、错误等）
│   │   ├── auth.py
│   │   ├── workspace.py
│   │   ├── application.py
│   │   ├── table.py
│   │   ├── field.py
│   │   ├── row.py
│   │   ├── view.py
│   │   ├── builder.py
│   │   ├── dashboard.py
│   │   └── automation.py
│   │
│   ├── services/
│   │   ├── auth_service.py
│   │   ├── workspace_service.py
│   │   ├── application_service.py
│   │   ├── table_service.py
│   │   ├── field_service.py
│   │   ├── row_service.py
│   │   ├── view_service.py
│   │   ├── builder_service.py
│   │   ├── dashboard_service.py
│   │   └── automation_service.py
│   │
│   ├── websocket/
│   │   ├── manager.py              # WebSocket 连接管理器
│   │   └── events.py               # 事件类型定义
│   │
│   ├── tasks/
│   │   ├── celery_app.py           # Celery 实例
│   │   ├── automation_tasks.py
│   │   ├── import_export_tasks.py
│   │   └── ai_tasks.py
│   │
│   └── exceptions/
│       ├── base.py                 # BaserowException
│       └── handlers.py             # FastAPI 异常处理器
│
├── alembic/
│   ├── env.py
│   └── versions/
│
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   └── services/
│   └── integration/
│       └── api/
│
├── pyproject.toml
├── Dockerfile
└── docker-compose.yml
```

---

## 3. 数据库模型设计

### 3.1 核心模型概览

```
users                    用户账户
workspaces               工作区
workspace_members        工作区成员关系（多对多）
workspace_invitations    工作区邀请
applications             应用（多态：database/builder/dashboard/automation）
tables                   表（属于 database 类型 Application）
fields                   字段（多态：text/number/date/...，属于 Table）
views                    视图（多态：grid/gallery/kanban/...，属于 Table）
view_filters             视图过滤条件
view_sorts               视图排序规则
trash_entries            软删除记录
builder_pages            Builder 页面（属于 builder 类型 Application）
builder_elements         Builder 页面元素（多态）
dashboard_widgets        仪表板 Widget（多态，属于 dashboard 类型 Application）
automations              自动化工作流（属于 automation 类型 Application）
automation_triggers      触发器
automation_actions       动作
actions                  用户操作历史（Undo/Redo）
```

### 3.2 关键模型字段

#### users 表

| 列名 | 类型 | 说明 |
|------|------|------|
| id | UUID (PK) | 主键 |
| email | VARCHAR(254) UNIQUE | 登录邮箱 |
| password_hash | VARCHAR(128) | bcrypt 哈希 |
| first_name | VARCHAR(64) | 名 |
| last_name | VARCHAR(64) | 姓 |
| is_active | BOOLEAN | 账户是否激活 |
| is_staff | BOOLEAN | 是否为系统管理员 |
| language | VARCHAR(10) | 界面语言偏好 |
| created_at | TIMESTAMPTZ | 创建时间 |
| updated_at | TIMESTAMPTZ | 更新时间 |

#### workspaces 表

| 列名 | 类型 | 说明 |
|------|------|------|
| id | BIGINT (PK) | 主键 |
| name | VARCHAR(165) | 工作区名称 |
| created_at | TIMESTAMPTZ | 创建时间 |
| updated_at | TIMESTAMPTZ | 更新时间 |

#### workspace_members 表

| 列名 | 类型 | 说明 |
|------|------|------|
| id | BIGINT (PK) | 主键 |
| workspace_id | BIGINT (FK) | 关联工作区 |
| user_id | UUID (FK) | 关联用户 |
| role | VARCHAR(32) | ADMIN / MEMBER |
| order | INTEGER | 排序权重 |
| created_at | TIMESTAMPTZ | 加入时间 |

#### applications 表（多态）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | BIGINT (PK) | 主键 |
| workspace_id | BIGINT (FK) | 所属工作区 |
| type | VARCHAR(32) | database / builder / dashboard / automation |
| name | VARCHAR(160) | 应用名称 |
| order | INTEGER | 排序权重 |
| trashed | BOOLEAN | 是否在垃圾桶 |
| created_at | TIMESTAMPTZ | 创建时间 |
| updated_at | TIMESTAMPTZ | 更新时间 |

#### tables 表

| 列名 | 类型 | 说明 |
|------|------|------|
| id | BIGINT (PK) | 主键 |
| database_id | BIGINT (FK) | 所属 database 应用 |
| name | VARCHAR(255) | 表名 |
| order | INTEGER | 排序权重 |
| trashed | BOOLEAN | 是否在垃圾桶 |
| created_at | TIMESTAMPTZ | 创建时间 |
| updated_at | TIMESTAMPTZ | 更新时间 |

#### fields 表（多态）

| 列名 | 类型 | 说明 |
|------|------|------|
| id | BIGINT (PK) | 主键 |
| table_id | BIGINT (FK) | 所属表 |
| type | VARCHAR(64) | text / number / date / single_select / ... |
| name | VARCHAR(255) | 字段名 |
| order | INTEGER | 排序权重 |
| primary | BOOLEAN | 是否为主字段 |
| trashed | BOOLEAN | 是否在垃圾桶 |
| created_at | TIMESTAMPTZ | 创建时间 |
| updated_at | TIMESTAMPTZ | 更新时间 |
| extra_config | JSONB | 各类型专有配置（如 number precision, date_format 等） |

> 用户实际行数据存储在单独的动态表 `database_table_{table_id}` 中。

---

## 4. 认证系统

### JWT 认证流程

```
1. POST /api/v1/auth/login → 返回 access_token（15分钟）+ refresh_token（30天）
2. 客户端在每个请求 Header 携带: Authorization: Bearer <access_token>
3. access_token 过期后，使用 POST /api/v1/auth/token/refresh 换新 access_token
4. refresh_token 也过期或登出时，需重新登录
```

### Token 结构（JWT Payload）

```json
{
  "sub": "user-uuid-here",
  "type": "access",
  "exp": 1710000000,
  "iat": 1709999100
}
```

### FastAPI 依赖注入

```python
# app/dependencies.py
async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
) -> User:
    ...

async def get_current_active_user(
    user: User = Depends(get_current_user)
) -> User:
    if not user.is_active:
        raise HTTPException(status_code=400, detail="Inactive user")
    return user

async def require_workspace_member(
    workspace_id: int,
    current_user: User = Depends(get_current_active_user),
    db: AsyncSession = Depends(get_db)
) -> WorkspaceMember:
    ...

async def require_workspace_admin(
    member: WorkspaceMember = Depends(require_workspace_member)
) -> WorkspaceMember:
    if member.role != "ADMIN":
        raise HTTPException(status_code=403, detail="Admin role required")
    return member
```

---

## 5. API 接口定义（全量）

### 全局约定

- **Base URL**: `/api/v1`
- **Content-Type**: `application/json`（文件上传除外）
- **认证**: `Authorization: Bearer <access_token>`（标注"需认证"的接口必须携带）
- **分页参数**: `?page=1&size=20`（默认 page=1，size=20，最大 size=200）
- **错误格式**:
  ```json
  { "error": "ERROR_CODE", "detail": "可读说明" }
  ```

---

### 5.1 认证接口

#### POST `/api/v1/auth/register` — 注册

**请求体**

```json
{
  "email": "user@example.com",
  "password": "SecureP@ssw0rd",
  "first_name": "张",
  "last_name": "三",
  "language": "zh-CN"
}
```

**响应** `201 Created`

```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "first_name": "张",
    "last_name": "三",
    "language": "zh-CN",
    "is_staff": false,
    "created_at": "2026-03-16T00:00:00Z"
  },
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer"
}
```

**错误码**: `ERROR_EMAIL_ALREADY_EXISTS`, `ERROR_PASSWORD_TOO_SHORT`

---

#### POST `/api/v1/auth/login` — 登录

**请求体**

```json
{
  "email": "user@example.com",
  "password": "SecureP@ssw0rd"
}
```

**响应** `200 OK`

```json
{
  "user": { "id": "uuid", "email": "...", "first_name": "...", "last_name": "...", "language": "...", "is_staff": false, "created_at": "..." },
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer"
}
```

**错误码**: `ERROR_INVALID_CREDENTIALS`, `ERROR_DEACTIVATED_USER`

---

#### POST `/api/v1/auth/token/refresh` — 刷新 Token

**请求体**

```json
{ "refresh_token": "eyJ..." }
```

**响应** `200 OK`

```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer"
}
```

**错误码**: `ERROR_REFRESH_TOKEN_EXPIRED`, `ERROR_REFRESH_TOKEN_INVALID`

---

#### POST `/api/v1/auth/logout` — 登出（需认证）

**请求体**

```json
{ "refresh_token": "eyJ..." }
```

**响应** `204 No Content`

---

#### GET `/api/v1/auth/me` — 获取当前用户信息（需认证）

**响应** `200 OK`

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "first_name": "张",
  "last_name": "三",
  "language": "zh-CN",
  "is_staff": false,
  "created_at": "2026-03-16T00:00:00Z"
}
```

---

#### PATCH `/api/v1/auth/me` — 更新用户信息（需认证）

**请求体**（所有字段可选）

```json
{
  "first_name": "张",
  "last_name": "四",
  "language": "en"
}
```

**响应** `200 OK` — 同 GET `/auth/me`

---

#### POST `/api/v1/auth/change-password` — 修改密码（需认证）

**请求体**

```json
{
  "old_password": "OldPass",
  "new_password": "NewPass123!"
}
```

**响应** `204 No Content`

**错误码**: `ERROR_INVALID_OLD_PASSWORD`, `ERROR_PASSWORD_TOO_SHORT`

---

### 5.2 工作区接口

#### GET `/api/v1/workspaces` — 列出我的工作区（需认证）

**响应** `200 OK`

```json
{
  "workspaces": [
    {
      "id": 1,
      "name": "我的团队",
      "my_role": "ADMIN",
      "members_count": 5,
      "created_at": "2026-03-16T00:00:00Z",
      "updated_at": "2026-03-16T00:00:00Z"
    }
  ]
}
```

---

#### POST `/api/v1/workspaces` — 创建工作区（需认证）

**请求体**

```json
{ "name": "新工作区" }
```

**响应** `201 Created`

```json
{
  "id": 2,
  "name": "新工作区",
  "my_role": "ADMIN",
  "members_count": 1,
  "created_at": "...",
  "updated_at": "..."
}
```

**错误码**: `ERROR_WORKSPACE_NAME_TOO_LONG`

---

#### GET `/api/v1/workspaces/{workspace_id}` — 获取工作区详情（需认证，需成员）

**响应** `200 OK` — 同上结构

**错误码**: `ERROR_WORKSPACE_DOES_NOT_EXIST`, `ERROR_NOT_A_MEMBER`

---

#### PATCH `/api/v1/workspaces/{workspace_id}` — 更新工作区（需认证，需 ADMIN）

**请求体**

```json
{ "name": "新名称" }
```

**响应** `200 OK`

---

#### DELETE `/api/v1/workspaces/{workspace_id}` — 删除工作区（需认证，需 ADMIN）

**响应** `204 No Content`

**效果**: 工作区及其下所有应用移入垃圾桶

---

#### GET `/api/v1/workspaces/{workspace_id}/members` — 工作区成员列表（需认证，需成员）

**响应** `200 OK`

```json
{
  "members": [
    {
      "id": 10,
      "user_id": "uuid",
      "email": "user@example.com",
      "first_name": "张",
      "last_name": "三",
      "role": "ADMIN",
      "created_at": "..."
    }
  ]
}
```

---

#### POST `/api/v1/workspaces/{workspace_id}/invitations` — 邀请成员（需认证，需 ADMIN）

**请求体**

```json
{
  "email": "newmember@example.com",
  "role": "MEMBER",
  "message": "欢迎加入我们的工作区！"
}
```

**响应** `201 Created`

```json
{
  "id": 5,
  "email": "newmember@example.com",
  "role": "MEMBER",
  "workspace_id": 1,
  "invited_by": "uuid",
  "expires_at": "2026-03-23T00:00:00Z",
  "created_at": "..."
}
```

**效果**: 发送邀请邮件（Celery 任务）

**错误码**: `ERROR_USER_ALREADY_MEMBER`, `ERROR_INVITATION_ALREADY_SENT`

---

#### POST `/api/v1/workspaces/invitations/{invitation_token}/accept` — 接受邀请

**无需认证（通过 Token 识别邀请）**

**请求体**（若未登录则需提供注册信息，已登录则留空）

```json
{
  "first_name": "李",
  "last_name": "四",
  "password": "MyPass123!"
}
```

**响应** `200 OK` — 返回用户信息 + JWT Token

---

#### PATCH `/api/v1/workspaces/{workspace_id}/members/{user_id}` — 修改成员角色（需认证，需 ADMIN）

**请求体**

```json
{ "role": "ADMIN" }
```

**响应** `200 OK`

**错误码**: `ERROR_CANNOT_CHANGE_OWN_ROLE_IF_LAST_ADMIN`

---

#### DELETE `/api/v1/workspaces/{workspace_id}/members/{user_id}` — 移除成员（需认证，需 ADMIN）

**响应** `204 No Content`

**错误码**: `ERROR_CANNOT_REMOVE_LAST_ADMIN`

---

#### DELETE `/api/v1/workspaces/{workspace_id}/members/me` — 离开工作区（需认证）

**响应** `204 No Content`

---

### 5.3 应用接口

#### GET `/api/v1/workspaces/{workspace_id}/applications` — 列出工作区应用（需认证，需成员）

**响应** `200 OK`

```json
{
  "applications": [
    {
      "id": 1,
      "type": "database",
      "name": "CRM 数据库",
      "order": 1,
      "workspace_id": 1,
      "created_at": "...",
      "updated_at": "..."
    }
  ]
}
```

---

#### POST `/api/v1/workspaces/{workspace_id}/applications` — 创建应用（需认证，需 ADMIN）

**请求体**

```json
{
  "type": "database",
  "name": "项目管理"
}
```

`type` 取值: `database` | `builder` | `dashboard` | `automation`

**响应** `201 Created` — 同列表中单个条目结构

---

#### GET `/api/v1/applications/{application_id}` — 获取应用详情（需认证，需成员）

**响应** `200 OK` — 同上结构

---

#### PATCH `/api/v1/applications/{application_id}` — 更新应用（需认证，需 ADMIN）

**请求体**

```json
{ "name": "新名称" }
```

**响应** `200 OK`

---

#### DELETE `/api/v1/applications/{application_id}` — 删除应用（需认证，需 ADMIN）

**响应** `204 No Content`（移入垃圾桶）

---

#### POST `/api/v1/applications/{application_id}/order` — 调整应用顺序（需认证，需 ADMIN）

**请求体**

```json
{ "applications": [3, 1, 2] }
```

**响应** `204 No Content`

---

### 5.4 表接口

#### GET `/api/v1/applications/{application_id}/tables` — 列出表（需认证，需成员）

**响应** `200 OK`

```json
{
  "tables": [
    {
      "id": 1,
      "name": "联系人",
      "order": 1,
      "database_id": 1,
      "created_at": "...",
      "updated_at": "..."
    }
  ]
}
```

---

#### POST `/api/v1/applications/{application_id}/tables` — 创建表（需认证，需 ADMIN）

**请求体**

```json
{
  "name": "任务",
  "default_fields": true
}
```

`default_fields`: 是否创建默认的 Name 字段，默认 `true`

**响应** `201 Created`

```json
{
  "id": 2,
  "name": "任务",
  "order": 2,
  "database_id": 1,
  "created_at": "...",
  "updated_at": "...",
  "fields": [
    { "id": 1, "type": "text", "name": "Name", "primary": true, "order": 1 }
  ],
  "views": [
    { "id": 1, "type": "grid", "name": "Grid View", "order": 1 }
  ]
}
```

---

#### GET `/api/v1/tables/{table_id}` — 获取表详情（需认证，需成员）

**响应** `200 OK`

```json
{
  "id": 1,
  "name": "联系人",
  "order": 1,
  "database_id": 1,
  "fields": [...],
  "views": [...],
  "created_at": "...",
  "updated_at": "..."
}
```

---

#### PATCH `/api/v1/tables/{table_id}` — 更新表（需认证，需 ADMIN）

**请求体**

```json
{ "name": "新表名" }
```

**响应** `200 OK`

---

#### DELETE `/api/v1/tables/{table_id}` — 删除表（需认证，需 ADMIN）

**响应** `204 No Content`（移入垃圾桶）

---

#### POST `/api/v1/applications/{application_id}/tables/order` — 调整表顺序（需认证，需 ADMIN）

**请求体**

```json
{ "tables": [2, 1, 3] }
```

**响应** `204 No Content`

---

### 5.5 字段接口

#### GET `/api/v1/tables/{table_id}/fields` — 列出字段（需认证，需成员）

**响应** `200 OK`

```json
{
  "fields": [
    {
      "id": 1,
      "table_id": 1,
      "type": "text",
      "name": "Name",
      "primary": true,
      "order": 1,
      "config": {}
    },
    {
      "id": 2,
      "table_id": 1,
      "type": "number",
      "name": "金额",
      "primary": false,
      "order": 2,
      "config": {
        "number_decimal_places": 2,
        "number_negative": true
      }
    }
  ]
}
```

**字段类型 `config` 结构说明**见第 8 节。

---

#### POST `/api/v1/tables/{table_id}/fields` — 创建字段（需认证，需 ADMIN）

**请求体**（以 `single_select` 为例）

```json
{
  "type": "single_select",
  "name": "状态",
  "config": {
    "select_options": [
      { "value": "待处理", "color": "blue" },
      { "value": "进行中", "color": "yellow" },
      { "value": "已完成", "color": "green" }
    ]
  }
}
```

**响应** `201 Created` — 同列表中单个字段结构

**错误码**: `ERROR_MAX_FIELD_COUNT_EXCEEDED`, `ERROR_INVALID_FIELD_TYPE`, `ERROR_FIELD_NAME_TOO_LONG`

---

#### GET `/api/v1/fields/{field_id}` — 获取字段详情（需认证，需成员）

**响应** `200 OK` — 同列表单个字段结构

---

#### PATCH `/api/v1/fields/{field_id}` — 更新字段（需认证，需 ADMIN）

**请求体**（可更新字段名、config，类型变更需慎重）

```json
{
  "name": "新字段名",
  "config": { "number_decimal_places": 0 }
}
```

**响应** `200 OK`

---

#### DELETE `/api/v1/fields/{field_id}` — 删除字段（需认证，需 ADMIN）

**响应** `204 No Content`（移入垃圾桶）

**错误码**: `ERROR_CANNOT_DELETE_PRIMARY_FIELD`

---

#### POST `/api/v1/tables/{table_id}/fields/order` — 调整字段顺序（需认证，需 ADMIN）

**请求体**

```json
{ "fields": [3, 1, 2] }
```

**响应** `204 No Content`

---

### 5.6 行接口

> 行数据直接读写用户的动态 PostgreSQL 表。

#### GET `/api/v1/tables/{table_id}/rows` — 列出行（需认证，需成员）

**Query Parameters**

| 参数 | 类型 | 说明 |
|------|------|------|
| `page` | int | 页码，默认 1 |
| `size` | int | 每页数量，默认 20，最大 200 |
| `search` | string | 全文搜索 |
| `order_by` | string | 排序字段，如 `field_1`（升序）或 `-field_1`（降序） |
| `filter_{field_id}_{operator}` | string | 过滤，如 `filter_1_contains=张三` |
| `view_id` | int | 应用视图的过滤/排序配置（可选） |
| `include_fields` | string | 仅返回指定字段，逗号分隔，如 `field_1,field_2` |

**响应** `200 OK`

```json
{
  "count": 100,
  "next": "/api/v1/tables/1/rows?page=2&size=20",
  "previous": null,
  "results": [
    {
      "id": 1,
      "order": "1.00000000000000000000",
      "field_1": "张三",
      "field_2": 5000.00,
      "field_3": "2026-03-16",
      "created_on": "2026-03-16T10:00:00Z",
      "updated_on": "2026-03-16T11:00:00Z"
    }
  ]
}
```

---

#### POST `/api/v1/tables/{table_id}/rows` — 创建行（需认证，需成员）

**请求体**

```json
{
  "field_1": "李四",
  "field_2": 8000.00,
  "field_3": "2026-04-01"
}
```

**Query Parameters**

| 参数 | 说明 |
|------|------|
| `before_row_id` | 在指定行前插入 |
| `user_field_names` | 若为 `true`，请求体 key 使用字段名而非 `field_{id}` |

**响应** `201 Created` — 同列表单行结构

---

#### GET `/api/v1/tables/{table_id}/rows/{row_id}` — 获取行详情（需认证，需成员）

**响应** `200 OK` — 同列表单行结构

---

#### PATCH `/api/v1/tables/{table_id}/rows/{row_id}` — 更新行（需认证，需成员）

**请求体**（仅需传要更新的字段）

```json
{ "field_2": 9000.00 }
```

**响应** `200 OK`

---

#### DELETE `/api/v1/tables/{table_id}/rows/{row_id}` — 删除行（需认证，需成员）

**响应** `204 No Content`

---

#### POST `/api/v1/tables/{table_id}/rows/batch` — 批量创建行（需认证，需成员）

**请求体**

```json
{
  "rows": [
    { "field_1": "王五", "field_2": 6000 },
    { "field_1": "赵六", "field_2": 7000 }
  ]
}
```

**响应** `201 Created`

```json
{
  "items": [
    { "id": 10, "field_1": "王五", "field_2": 6000, ... },
    { "id": 11, "field_1": "赵六", "field_2": 7000, ... }
  ]
}
```

---

#### PATCH `/api/v1/tables/{table_id}/rows/batch` — 批量更新行（需认证，需成员）

**请求体**

```json
{
  "rows": [
    { "id": 10, "field_2": 6500 },
    { "id": 11, "field_2": 7500 }
  ]
}
```

**响应** `200 OK` — 同批量创建响应结构

---

#### DELETE `/api/v1/tables/{table_id}/rows/batch` — 批量删除行（需认证，需成员）

**请求体**

```json
{ "row_ids": [10, 11, 12] }
```

**响应** `204 No Content`

---

#### POST `/api/v1/tables/{table_id}/rows/import` — 导入行（需认证，需 ADMIN）

**请求**: `multipart/form-data`

| 字段 | 类型 | 说明 |
|------|------|------|
| `file` | File | CSV、JSON 或 XML 文件 |
| `format` | string | `csv` \| `json` \| `xml` |
| `charset` | string | 编码，默认 `utf-8` |
| `first_row_header` | boolean | CSV 首行是否为标题（默认 true） |

**响应** `202 Accepted`（异步导入，返回任务 ID）

```json
{
  "task_id": "celery-task-uuid",
  "status": "pending"
}
```

---

#### GET `/api/v1/tables/{table_id}/rows/export` — 导出行（需认证，需成员）

**Query Parameters**: `format=csv|json|xml`, `view_id=<int>`（可选，应用视图过滤）

**响应** `200 OK` — 文件流（Content-Disposition: attachment）

---

### 5.7 视图接口

#### GET `/api/v1/tables/{table_id}/views` — 列出视图（需认证，需成员）

**响应** `200 OK`

```json
{
  "views": [
    {
      "id": 1,
      "table_id": 1,
      "type": "grid",
      "name": "Grid View",
      "order": 1,
      "filter_type": "AND",
      "filters_disabled": false,
      "config": {},
      "created_at": "...",
      "updated_at": "..."
    }
  ]
}
```

---

#### POST `/api/v1/tables/{table_id}/views` — 创建视图（需认证，需 ADMIN）

**请求体**

```json
{
  "type": "kanban",
  "name": "看板视图",
  "config": {
    "kanban_field_id": 5
  }
}
```

`type` 取值: `grid` | `form` | `gallery` | `kanban` | `calendar` | `timeline`

**响应** `201 Created` — 同列表单个视图结构

---

#### GET `/api/v1/views/{view_id}` — 获取视图（需认证，需成员）

**响应** `200 OK` — 包含 filters 和 sorts 完整配置

```json
{
  "id": 1,
  "type": "grid",
  "name": "Grid View",
  "order": 1,
  "filter_type": "AND",
  "filters_disabled": false,
  "config": {},
  "filters": [
    {
      "id": 1,
      "field_id": 2,
      "type": "contains",
      "value": "张"
    }
  ],
  "sorts": [
    {
      "id": 1,
      "field_id": 1,
      "order": "ASC"
    }
  ],
  "field_options": [
    {
      "field_id": 1,
      "hidden": false,
      "order": 1,
      "width": 200
    }
  ]
}
```

---

#### PATCH `/api/v1/views/{view_id}` — 更新视图（需认证，需 ADMIN）

**请求体**

```json
{
  "name": "新视图名",
  "filter_type": "OR",
  "filters_disabled": false
}
```

**响应** `200 OK`

---

#### DELETE `/api/v1/views/{view_id}` — 删除视图（需认证，需 ADMIN）

**响应** `204 No Content`

---

#### POST `/api/v1/views/{view_id}/filters` — 添加过滤条件（需认证，需 ADMIN）

**请求体**

```json
{
  "field_id": 2,
  "type": "contains",
  "value": "张"
}
```

**过滤类型 (`type`) 取值**:

| 类型 | 说明 | 适用字段 |
|------|------|----------|
| `equal` | 等于 | 所有类型 |
| `not_equal` | 不等于 | 所有类型 |
| `contains` | 包含 | text |
| `contains_not` | 不包含 | text |
| `empty` | 为空 | 所有类型 |
| `not_empty` | 不为空 | 所有类型 |
| `higher_than` | 大于 | number, date |
| `lower_than` | 小于 | number, date |
| `date_equals` | 日期等于 | date |
| `date_before` | 日期在之前 | date |
| `date_after` | 日期在之后 | date |
| `link_row_has` | 关联行包含 | link_row |
| `link_row_has_not` | 关联行不包含 | link_row |
| `multiple_select_has` | 多选包含 | multiple_select |

**响应** `201 Created`

```json
{ "id": 3, "field_id": 2, "type": "contains", "value": "张" }
```

---

#### PATCH `/api/v1/views/{view_id}/filters/{filter_id}` — 更新过滤条件（需认证，需 ADMIN）

**请求体** — 同创建，字段可选

**响应** `200 OK`

---

#### DELETE `/api/v1/views/{view_id}/filters/{filter_id}` — 删除过滤条件（需认证，需 ADMIN）

**响应** `204 No Content`

---

#### POST `/api/v1/views/{view_id}/sorts` — 添加排序（需认证，需 ADMIN）

**请求体**

```json
{ "field_id": 1, "order": "DESC" }
```

**响应** `201 Created`

```json
{ "id": 2, "field_id": 1, "order": "DESC" }
```

---

#### PATCH `/api/v1/views/{view_id}/sorts/{sort_id}` — 更新排序（需认证，需 ADMIN）

**响应** `200 OK`

---

#### DELETE `/api/v1/views/{view_id}/sorts/{sort_id}` — 删除排序（需认证，需 ADMIN）

**响应** `204 No Content`

---

#### PATCH `/api/v1/views/{view_id}/field-options` — 更新视图字段配置（需认证，需 ADMIN）

**请求体**（更新指定字段的显示/宽度配置）

```json
{
  "field_options": {
    "1": { "hidden": false, "width": 250 },
    "2": { "hidden": true }
  }
}
```

**响应** `200 OK`

---

### 5.8 Builder 接口

#### GET `/api/v1/applications/{application_id}/builder/pages` — 列出页面（需认证，需成员）

**响应** `200 OK`

```json
{
  "pages": [
    {
      "id": 1,
      "application_id": 1,
      "name": "首页",
      "path": "/",
      "order": 1,
      "created_at": "...",
      "updated_at": "..."
    }
  ]
}
```

---

#### POST `/api/v1/applications/{application_id}/builder/pages` — 创建页面（需认证，需 ADMIN）

**请求体**

```json
{
  "name": "联系人列表",
  "path": "/contacts"
}
```

**响应** `201 Created`

---

#### GET `/api/v1/builder/pages/{page_id}` — 获取页面详情（需认证，需成员）

**响应** `200 OK`

```json
{
  "id": 1,
  "name": "首页",
  "path": "/",
  "order": 1,
  "elements": [
    {
      "id": 1,
      "page_id": 1,
      "type": "heading",
      "order": "1.00",
      "parent_element_id": null,
      "config": {
        "value": "欢迎使用",
        "level": 1
      }
    }
  ]
}
```

---

#### PATCH `/api/v1/builder/pages/{page_id}` — 更新页面（需认证，需 ADMIN）

**请求体**

```json
{ "name": "新页面名", "path": "/new-path" }
```

**响应** `200 OK`

---

#### DELETE `/api/v1/builder/pages/{page_id}` — 删除页面（需认证，需 ADMIN）

**响应** `204 No Content`

---

#### POST `/api/v1/builder/pages/{page_id}/elements` — 创建元素（需认证，需 ADMIN）

**请求体**

```json
{
  "type": "table",
  "order": "2.00",
  "parent_element_id": null,
  "config": {
    "data_source_id": 1,
    "fields": ["field_1", "field_2"]
  }
}
```

**元素类型 (`type`) 取值**: `heading` | `text` | `image` | `button` | `table` | `form` | `chart` | `container` | `column`

**响应** `201 Created`

---

#### PATCH `/api/v1/builder/elements/{element_id}` — 更新元素（需认证，需 ADMIN）

**请求体** — 与创建相同，字段可选

**响应** `200 OK`

---

#### DELETE `/api/v1/builder/elements/{element_id}` — 删除元素（需认证，需 ADMIN）

**响应** `204 No Content`

---

#### POST `/api/v1/builder/pages/{page_id}/elements/order` — 调整元素顺序（需认证，需 ADMIN）

**请求体**

```json
{ "elements": [3, 1, 2] }
```

**响应** `204 No Content`

---

#### POST `/api/v1/applications/{application_id}/builder/publish` — 发布 Builder 应用（需认证，需 ADMIN）

**响应** `200 OK`

```json
{
  "public_url": "https://app.baserow.io/public/applications/abc123",
  "published_at": "2026-03-16T12:00:00Z"
}
```

---

#### GET `/api/v1/public/builder/{application_id}/pages/{path}` — 公开访问 Builder 页面（无需认证）

**响应** `200 OK` — 页面结构及对应数据

---

### 5.9 仪表板接口

#### GET `/api/v1/applications/{application_id}/dashboard/widgets` — 列出 Widget（需认证，需成员）

**响应** `200 OK`

```json
{
  "widgets": [
    {
      "id": 1,
      "application_id": 1,
      "type": "summary",
      "name": "总联系人数",
      "order": 1,
      "config": {
        "table_id": 1,
        "aggregation": "count",
        "field_id": null
      }
    }
  ]
}
```

---

#### POST `/api/v1/applications/{application_id}/dashboard/widgets` — 创建 Widget（需认证，需 ADMIN）

**请求体**

```json
{
  "type": "chart",
  "name": "月度销售额",
  "config": {
    "table_id": 1,
    "chart_type": "bar",
    "x_field_id": 3,
    "y_field_id": 4,
    "aggregation": "sum"
  }
}
```

**Widget 类型 (`type`) 取值**: `summary` | `chart`

**响应** `201 Created`

---

#### PATCH `/api/v1/dashboard/widgets/{widget_id}` — 更新 Widget（需认证，需 ADMIN）

**响应** `200 OK`

---

#### DELETE `/api/v1/dashboard/widgets/{widget_id}` — 删除 Widget（需认证，需 ADMIN）

**响应** `204 No Content`

---

#### GET `/api/v1/dashboard/widgets/{widget_id}/data` — 获取 Widget 数据（需认证，需成员）

**响应** `200 OK`

```json
{
  "type": "summary",
  "value": 1234,
  "label": "总联系人数"
}
```

---

### 5.10 自动化接口

#### GET `/api/v1/applications/{application_id}/automations` — 列出自动化（需认证，需成员）

**响应** `200 OK`

```json
{
  "automations": [
    {
      "id": 1,
      "application_id": 1,
      "name": "新行通知",
      "active": true,
      "trigger": {
        "type": "row_created",
        "table_id": 1
      },
      "created_at": "...",
      "updated_at": "..."
    }
  ]
}
```

---

#### POST `/api/v1/applications/{application_id}/automations` — 创建自动化（需认证，需 ADMIN）

**请求体**

```json
{
  "name": "发送欢迎邮件",
  "trigger": {
    "type": "row_created",
    "table_id": 1
  },
  "actions": [
    {
      "type": "send_email",
      "order": 1,
      "config": {
        "to": "{{field_1}}",
        "subject": "欢迎加入",
        "body": "您好 {{field_2}}，欢迎加入我们！"
      }
    }
  ]
}
```

**触发器类型 (`trigger.type`)**: `row_created` | `row_updated` | `row_deleted` | `schedule` | `webhook_received`

**动作类型 (`action.type`)**: `create_row` | `update_row` | `delete_row` | `send_email` | `http_request` | `ai_action`

**响应** `201 Created`

---

#### GET `/api/v1/automations/{automation_id}` — 获取自动化详情（需认证，需成员）

**响应** `200 OK` — 同列表单个结构，附带完整 actions 列表

---

#### PATCH `/api/v1/automations/{automation_id}` — 更新自动化（需认证，需 ADMIN）

**请求体** — 字段可选

**响应** `200 OK`

---

#### DELETE `/api/v1/automations/{automation_id}` — 删除自动化（需认证，需 ADMIN）

**响应** `204 No Content`

---

#### POST `/api/v1/automations/{automation_id}/trigger` — 手动触发（需认证，需 ADMIN）

**请求体**（可选的测试输入数据）

```json
{ "test_data": { "field_1": "测试" } }
```

**响应** `202 Accepted`

```json
{ "task_id": "celery-task-uuid" }
```

---

#### GET `/api/v1/automations/{automation_id}/history` — 执行历史（需认证，需成员）

**Query Parameters**: `page`, `size`

**响应** `200 OK`

```json
{
  "count": 50,
  "results": [
    {
      "id": 1,
      "automation_id": 1,
      "triggered_at": "2026-03-16T10:00:00Z",
      "status": "success",
      "duration_ms": 450,
      "error": null
    }
  ]
}
```

---

### 5.11 文件上传接口

#### POST `/api/v1/files/upload` — 上传文件（需认证）

**请求**: `multipart/form-data`

| 字段 | 类型 | 说明 |
|------|------|------|
| `file` | File | 上传文件（最大 100MB） |

**响应** `200 OK`

```json
{
  "url": "https://storage.example.com/files/uuid/filename.jpg",
  "name": "filename.jpg",
  "size": 102400,
  "mime_type": "image/jpeg",
  "is_image": true,
  "image_width": 1920,
  "image_height": 1080,
  "uploaded_at": "2026-03-16T12:00:00Z"
}
```

---

#### POST `/api/v1/files/upload-via-url` — 通过 URL 上传文件（需认证）

**请求体**

```json
{ "url": "https://example.com/image.jpg" }
```

**响应** `200 OK` — 同上传文件响应

---

### 5.12 健康检查接口

#### GET `/api/v1/health` — 健康检查（无需认证）

**响应** `200 OK`

```json
{
  "status": "healthy",
  "version": "2.0.0",
  "database": "ok",
  "redis": "ok",
  "celery": "ok"
}
```

#### GET `/api/v1/health/full` — 详细健康检查（需认证，需 Staff）

**响应** `200 OK`

```json
{
  "status": "healthy",
  "version": "2.0.0",
  "checks": {
    "database": { "status": "ok", "latency_ms": 2 },
    "redis": { "status": "ok", "latency_ms": 1 },
    "celery": { "status": "ok", "pending_tasks": 0 },
    "storage": { "status": "ok" }
  }
}
```

---

## 6. WebSocket 协议

### 连接端点

```
WS /ws/workspaces/{workspace_id}
Authorization: Bearer <access_token>   (在 Query String 中传递: ?token=<access_token>)
```

### 客户端 → 服务端消息

#### 订阅表更新

```json
{
  "type": "subscribe_table",
  "table_id": 1
}
```

#### 取消订阅

```json
{
  "type": "unsubscribe_table",
  "table_id": 1
}
```

#### Ping

```json
{ "type": "ping" }
```

---

### 服务端 → 客户端消息

所有推送消息均遵循以下格式：

```json
{
  "type": "<event_type>",
  "workspace_id": 1,
  "table_id": 1,
  "user_id": "uuid",        // 触发操作的用户（自己的操作也会收到）
  "data": { ... }           // 事件数据
}
```

#### 事件类型列表

| 事件类型 | 触发场景 | data 内容 |
|----------|----------|-----------|
| `row_created` | 行被创建 | `{ "row": {...} }` |
| `row_updated` | 行被更新 | `{ "row_id": 1, "values": {"field_1": "新值"} }` |
| `row_deleted` | 行被删除 | `{ "row_id": 1 }` |
| `rows_created` | 批量创建行 | `{ "rows": [{...}] }` |
| `rows_updated` | 批量更新行 | `{ "rows": [{...}] }` |
| `rows_deleted` | 批量删除行 | `{ "row_ids": [1, 2, 3] }` |
| `field_created` | 字段被创建 | `{ "field": {...} }` |
| `field_updated` | 字段被更新 | `{ "field_id": 1, "field": {...} }` |
| `field_deleted` | 字段被删除 | `{ "field_id": 1 }` |
| `view_created` | 视图被创建 | `{ "view": {...} }` |
| `view_updated` | 视图被更新 | `{ "view_id": 1, "view": {...} }` |
| `view_deleted` | 视图被删除 | `{ "view_id": 1 }` |
| `table_created` | 表被创建 | `{ "table": {...} }` |
| `table_updated` | 表被更新 | `{ "table_id": 1, "table": {...} }` |
| `table_deleted` | 表被删除 | `{ "table_id": 1 }` |
| `application_created` | 应用被创建 | `{ "application": {...} }` |
| `application_updated` | 应用被更新 | `{ "application_id": 1, "application": {...} }` |
| `application_deleted` | 应用被删除 | `{ "application_id": 1 }` |
| `pong` | 响应 ping | `{}` |

---

## 7. 错误码定义

### HTTP 状态码约定

| 状态码 | 场景 |
|--------|------|
| 200 | 成功查询/更新 |
| 201 | 成功创建 |
| 202 | 已接受（异步任务） |
| 204 | 成功删除/无内容 |
| 400 | 请求参数错误 |
| 401 | 未认证（Token 无效或缺失） |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 409 | 业务冲突（如重名、已存在） |
| 422 | Pydantic 验证失败 |
| 429 | 请求频率超限 |
| 500 | 服务端内部错误 |

### 业务错误码

| 错误码 | HTTP | 说明 |
|--------|------|------|
| `ERROR_EMAIL_ALREADY_EXISTS` | 409 | 邮箱已注册 |
| `ERROR_PASSWORD_TOO_SHORT` | 400 | 密码太短（<8位） |
| `ERROR_INVALID_CREDENTIALS` | 401 | 邮箱或密码错误 |
| `ERROR_DEACTIVATED_USER` | 403 | 账户已禁用 |
| `ERROR_REFRESH_TOKEN_EXPIRED` | 401 | Refresh Token 已过期 |
| `ERROR_REFRESH_TOKEN_INVALID` | 401 | Refresh Token 无效 |
| `ERROR_INVALID_OLD_PASSWORD` | 400 | 旧密码错误 |
| `ERROR_WORKSPACE_DOES_NOT_EXIST` | 404 | 工作区不存在 |
| `ERROR_NOT_A_MEMBER` | 403 | 非工作区成员 |
| `ERROR_WORKSPACE_NAME_TOO_LONG` | 400 | 工作区名称过长 |
| `ERROR_CANNOT_CHANGE_OWN_ROLE_IF_LAST_ADMIN` | 400 | 最后一个管理员无法降级 |
| `ERROR_CANNOT_REMOVE_LAST_ADMIN` | 400 | 无法移除最后一个管理员 |
| `ERROR_USER_ALREADY_MEMBER` | 409 | 用户已是成员 |
| `ERROR_APPLICATION_DOES_NOT_EXIST` | 404 | 应用不存在 |
| `ERROR_TABLE_DOES_NOT_EXIST` | 404 | 表不存在 |
| `ERROR_FIELD_DOES_NOT_EXIST` | 404 | 字段不存在 |
| `ERROR_CANNOT_DELETE_PRIMARY_FIELD` | 400 | 不能删除主字段 |
| `ERROR_MAX_FIELD_COUNT_EXCEEDED` | 400 | 超过最大字段数（1500） |
| `ERROR_INVALID_FIELD_TYPE` | 400 | 无效字段类型 |
| `ERROR_FIELD_NAME_TOO_LONG` | 400 | 字段名过长 |
| `ERROR_ROW_DOES_NOT_EXIST` | 404 | 行不存在 |
| `ERROR_VIEW_DOES_NOT_EXIST` | 404 | 视图不存在 |
| `ERROR_FILTER_DOES_NOT_EXIST` | 404 | 过滤条件不存在 |
| `ERROR_SORT_DOES_NOT_EXIST` | 404 | 排序规则不存在 |
| `ERROR_PERMISSION_DENIED` | 403 | 权限拒绝 |
| `ERROR_FILE_SIZE_TOO_LARGE` | 400 | 文件过大 |
| `ERROR_INVALID_FILE_TYPE` | 400 | 不支持的文件类型 |

---

## 8. Pydantic Schema 完整定义

### 8.1 公共 Schema

```python
# app/schemas/common.py

from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar("T")

class PaginatedResponse(BaseModel, Generic[T]):
    count: int
    next: Optional[str] = None
    previous: Optional[str] = None
    results: list[T]

class ErrorResponse(BaseModel):
    error: str
    detail: str
```

### 8.2 认证 Schema

```python
# app/schemas/auth.py

from pydantic import BaseModel, EmailStr, Field
from uuid import UUID
from datetime import datetime

class UserRegisterRequest(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8, max_length=256)
    first_name: str = Field(max_length=64)
    last_name: str = Field(max_length=64)
    language: str = Field(default="en", max_length=10)

class UserLoginRequest(BaseModel):
    email: EmailStr
    password: str

class TokenRefreshRequest(BaseModel):
    refresh_token: str

class LogoutRequest(BaseModel):
    refresh_token: str

class UserResponse(BaseModel):
    id: UUID
    email: EmailStr
    first_name: str
    last_name: str
    language: str
    is_staff: bool
    created_at: datetime

    model_config = {"from_attributes": True}

class AuthResponse(BaseModel):
    user: UserResponse
    access_token: str
    refresh_token: str
    token_type: str = "bearer"

class TokenResponse(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"

class ChangePasswordRequest(BaseModel):
    old_password: str
    new_password: str = Field(min_length=8, max_length=256)

class UpdateUserRequest(BaseModel):
    first_name: Optional[str] = Field(None, max_length=64)
    last_name: Optional[str] = Field(None, max_length=64)
    language: Optional[str] = Field(None, max_length=10)
```

### 8.3 工作区 Schema

```python
# app/schemas/workspace.py

from pydantic import BaseModel, EmailStr, Field
from datetime import datetime
from typing import Literal, Optional

class WorkspaceCreateRequest(BaseModel):
    name: str = Field(max_length=165)

class WorkspaceUpdateRequest(BaseModel):
    name: str = Field(max_length=165)

class WorkspaceResponse(BaseModel):
    id: int
    name: str
    my_role: Literal["ADMIN", "MEMBER"]
    members_count: int
    created_at: datetime
    updated_at: datetime

    model_config = {"from_attributes": True}

class WorkspaceMemberResponse(BaseModel):
    id: int
    user_id: str
    email: EmailStr
    first_name: str
    last_name: str
    role: Literal["ADMIN", "MEMBER"]
    created_at: datetime

class InviteMemberRequest(BaseModel):
    email: EmailStr
    role: Literal["ADMIN", "MEMBER"] = "MEMBER"
    message: Optional[str] = Field(None, max_length=500)

class InvitationResponse(BaseModel):
    id: int
    email: EmailStr
    role: str
    workspace_id: int
    invited_by: str
    expires_at: datetime
    created_at: datetime

class UpdateMemberRoleRequest(BaseModel):
    role: Literal["ADMIN", "MEMBER"]
```

### 8.4 字段 Schema（含各类型 config）

```python
# app/schemas/field.py

from pydantic import BaseModel, Field
from typing import Literal, Optional, Union, Annotated
from datetime import datetime

# ---- 各字段类型 config ----

class TextFieldConfig(BaseModel):
    text_default: Optional[str] = None

class NumberFieldConfig(BaseModel):
    number_decimal_places: int = Field(default=0, ge=0, le=10)
    number_negative: bool = True
    number_prefix: str = ""
    number_suffix: str = ""

class DateFieldConfig(BaseModel):
    date_format: Literal["ISO", "US", "EU"] = "ISO"
    date_include_time: bool = False
    date_time_format: Literal["24", "12"] = "24"
    date_show_tzinfo: bool = False
    date_force_timezone: Optional[str] = None

class SingleSelectOption(BaseModel):
    id: Optional[int] = None
    value: str = Field(max_length=255)
    color: str = Field(default="blue", max_length=64)

class SingleSelectFieldConfig(BaseModel):
    select_options: list[SingleSelectOption] = []

class MultipleSelectFieldConfig(BaseModel):
    select_options: list[SingleSelectOption] = []

class LinkRowFieldConfig(BaseModel):
    link_row_table_id: int
    link_row_related_field_id: Optional[int] = None

class FormulaFieldConfig(BaseModel):
    formula: str
    formula_type: Optional[str] = None

class RatingFieldConfig(BaseModel):
    max_value: int = Field(default=5, ge=1, le=10)
    color: str = "yellow"
    style: Literal["star", "heart", "thumbs-up", "flag"] = "star"

class FileFieldConfig(BaseModel):
    pass  # 无额外配置

class BooleanFieldConfig(BaseModel):
    pass  # 无额外配置

# ---- 统一字段类型 ----

FieldConfigType = Union[
    TextFieldConfig, NumberFieldConfig, DateFieldConfig,
    SingleSelectFieldConfig, MultipleSelectFieldConfig,
    LinkRowFieldConfig, FormulaFieldConfig, RatingFieldConfig,
    FileFieldConfig, BooleanFieldConfig
]

class FieldCreateRequest(BaseModel):
    type: str
    name: str = Field(max_length=255)
    config: Optional[dict] = None

class FieldUpdateRequest(BaseModel):
    name: Optional[str] = Field(None, max_length=255)
    config: Optional[dict] = None

class FieldResponse(BaseModel):
    id: int
    table_id: int
    type: str
    name: str
    primary: bool
    order: int
    config: dict

    model_config = {"from_attributes": True}

class FieldOrderRequest(BaseModel):
    fields: list[int]
```

### 8.5 行 Schema

```python
# app/schemas/row.py

from pydantic import BaseModel
from typing import Any, Optional
from datetime import datetime

class RowResponse(BaseModel):
    id: int
    order: str
    created_on: datetime
    updated_on: datetime
    # 动态字段（field_1, field_2...）通过 model_extra 传递

    model_config = {"extra": "allow", "from_attributes": True}

class RowCreateRequest(BaseModel):
    model_config = {"extra": "allow"}
    # 动态字段（field_1, field_2...）通过 extra 传递

class RowUpdateRequest(BaseModel):
    model_config = {"extra": "allow"}

class BatchCreateRequest(BaseModel):
    rows: list[dict[str, Any]]

class BatchUpdateRequest(BaseModel):
    rows: list[dict[str, Any]]  # 每项必须包含 "id"

class BatchDeleteRequest(BaseModel):
    row_ids: list[int]

class BatchRowsResponse(BaseModel):
    items: list[RowResponse]
```

---

## 9. Celery 异步任务

### 9.1 任务列表

| 任务名 | 触发场景 | 参数 |
|--------|----------|------|
| `tasks.send_invitation_email` | 邀请成员 | `invitation_id: int` |
| `tasks.run_automation` | 自动化触发 | `automation_id: int, trigger_data: dict` |
| `tasks.import_rows` | 行数据导入 | `table_id: int, file_path: str, format: str` |
| `tasks.send_email_action` | 自动化发邮件动作 | `to: str, subject: str, body: str` |
| `tasks.http_request_action` | 自动化 HTTP 请求动作 | `method: str, url: str, headers: dict, body: dict` |
| `tasks.recalculate_formula_fields` | 公式字段重计算 | `table_id: int, field_id: int, row_ids: list` |
| `tasks.recalculate_ai_field` | AI 字段重计算 | `table_id: int, field_id: int, row_id: int` |
| `tasks.cleanup_trash` | 定时清理垃圾桶（>30天） | 无 |

### 9.2 定时任务（Celery Beat）

```python
# app/tasks/celery_app.py

from celery.schedules import crontab

beat_schedule = {
    "cleanup-trash-daily": {
        "task": "tasks.cleanup_trash",
        "schedule": crontab(hour=3, minute=0),  # 每天凌晨 3 点
    },
}
```

---

## 10. 配置管理

```python
# app/config.py

from pydantic_settings import BaseSettings, SettingsConfigDict
from typing import Optional

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

    # 基础
    APP_NAME: str = "Baserow"
    DEBUG: bool = False
    SECRET_KEY: str
    ALLOWED_HOSTS: list[str] = ["*"]

    # 数据库
    DATABASE_URL: str  # postgresql+asyncpg://user:pass@host/db

    # Redis
    REDIS_URL: str = "redis://localhost:6379/0"

    # JWT
    JWT_SECRET_KEY: str
    JWT_ACCESS_TOKEN_EXPIRE_MINUTES: int = 15
    JWT_REFRESH_TOKEN_EXPIRE_DAYS: int = 30
    JWT_ALGORITHM: str = "HS256"

    # 邮件
    EMAIL_HOST: str = "localhost"
    EMAIL_PORT: int = 25
    EMAIL_USE_TLS: bool = False
    EMAIL_HOST_USER: Optional[str] = None
    EMAIL_HOST_PASSWORD: Optional[str] = None
    EMAIL_FROM: str = "no-reply@baserow.io"

    # 文件存储
    STORAGE_BACKEND: str = "local"  # local | s3 | azure | gcs
    STORAGE_LOCAL_PATH: str = "/baserow/media"
    AWS_S3_BUCKET: Optional[str] = None
    AWS_S3_REGION: Optional[str] = None
    AWS_ACCESS_KEY_ID: Optional[str] = None
    AWS_SECRET_ACCESS_KEY: Optional[str] = None

    # AI
    OPENAI_API_KEY: Optional[str] = None
    ANTHROPIC_API_KEY: Optional[str] = None

    # Sentry
    SENTRY_DSN: Optional[str] = None

    # 限速
    RATE_LIMIT_PER_MINUTE: int = 200

settings = Settings()
```

---

## 11. 部署指南

### Docker Compose（开发）

```yaml
# docker-compose.yml
version: "3.9"

services:
  db:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_USER: baserow
      POSTGRES_PASSWORD: baserow
      POSTGRES_DB: baserow
    volumes:
      - db_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  api:
    build: .
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    environment:
      DATABASE_URL: postgresql+asyncpg://baserow:baserow@db/baserow
      REDIS_URL: redis://redis:6379/0
      SECRET_KEY: dev-secret-key
      JWT_SECRET_KEY: dev-jwt-secret
    volumes:
      - .:/app
      - media_data:/baserow/media
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis

  worker:
    build: .
    command: celery -A app.tasks.celery_app worker --loglevel=info
    environment:
      DATABASE_URL: postgresql+asyncpg://baserow:baserow@db/baserow
      REDIS_URL: redis://redis:6379/0
      SECRET_KEY: dev-secret-key
      JWT_SECRET_KEY: dev-jwt-secret
    volumes:
      - .:/app
    depends_on:
      - db
      - redis

  beat:
    build: .
    command: celery -A app.tasks.celery_app beat --loglevel=info
    environment:
      REDIS_URL: redis://redis:6379/0
    depends_on:
      - redis

volumes:
  db_data:
  media_data:
```

### 初始化数据库

```bash
# 运行 Alembic 迁移
alembic upgrade head

# 创建初始超级管理员
python -m app.cli create-admin --email admin@example.com --password admin123
```

### 启动服务

```bash
# 开发模式
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 生产模式（Gunicorn + Uvicorn Workers）
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

---

*本文档由后端架构团队维护。如有变更请同步更新 [frontend-prd.md](./frontend-prd.md) 中对应接口定义。*
