# Baserow 产品需求文档（PRD）

> **版本：** 1.0  
> **状态：** 正式  
> **最后更新：** 2026-03-16  
> **适用版本：** Baserow v2.x  

---

## 目录

1. [产品概述与愿景](#1-产品概述与愿景)
2. [背景与问题定义](#2-背景与问题定义)
3. [目标用户画像](#3-目标用户画像)
4. [核心功能需求](#4-核心功能需求)
   - 4.1 [工作区与协作管理](#41-工作区与协作管理)
   - 4.2 [数据库与表格](#42-数据库与表格)
   - 4.3 [视图系统](#43-视图系统)
   - 4.4 [应用构建器（Builder）](#44-应用构建器builder)
   - 4.5 [自动化工作流（Automation）](#45-自动化工作流automation)
   - 4.6 [仪表板（Dashboard）](#46-仪表板dashboard)
   - 4.7 [AI 能力集成](#47-ai-能力集成)
5. [技术架构设计与设计决策](#5-技术架构设计与设计决策)
   - 5.1 [整体架构](#51-整体架构)
   - 5.2 [后端设计决策](#52-后端设计决策)
   - 5.3 [前端设计决策](#53-前端设计决策)
   - 5.4 [数据库设计决策](#54-数据库设计决策)
   - 5.5 [实时通信设计](#55-实时通信设计)
   - 5.6 [权限系统设计](#56-权限系统设计)
   - 5.7 [插件与扩展性设计](#57-插件与扩展性设计)
6. [非功能性需求](#6-非功能性需求)
7. [开发规范与约定](#7-开发规范与约定)
8. [分层功能边界（Open Source / Premium / Enterprise）](#8-分层功能边界open-source--premium--enterprise)
9. [成功度量指标](#9-成功度量指标)
10. [附录：术语表](#10-附录术语表)

---

## 1. 产品概述与愿景

### 1.1 产品定位

Baserow 是一款**开源、无代码的数据库与应用构建平台**。它让非技术人员可以像操作电子表格一样管理结构化数据，同时提供强大的 API、Automation、Builder 等能力，满足技术团队构建内部工具和业务应用的需要。

### 1.2 核心价值主张

| 维度 | 价值 |
|------|------|
| **数据自主权** | 支持完全自托管（Self-hosted），数据不离开用户自己的基础设施 |
| **开放可扩展** | 完整开源核心，插件体系允许第三方扩展功能 |
| **易用与强大兼得** | 无代码界面覆盖80%场景，REST API + 自动化覆盖剩余20%进阶需求 |
| **企业级安全合规** | 支持 GDPR、HIPAA、SOC 2 Type II；SAML2、OAuth SSO；细粒度权限 |
| **AI 增强** | 自然语言操作数据库、AI 字段自动处理、LLM 工作流集成 |

### 1.3 产品愿景

> **"让每个组织都拥有属于自己的、可完全掌控的数据平台，而无需写一行代码。"**

---

## 2. 背景与问题定义

### 2.1 市场痛点

现有市场上存在两类工具：

- **传统电子表格（Excel / Google Sheets）**：易用但扩展性差，多人协作有冲突，数据量大时性能下降，与其他系统集成困难。
- **传统关系数据库（PostgreSQL / MySQL）**：强大但门槛高，需要专业 DBA，非技术人员几乎无法独立使用。

夹在两者之间的工具（如 Airtable、Notion Database）虽然弥补了部分痛点，但存在以下问题：
1. **数据主权缺失**：数据存储在第三方云端，不适合对数据安全有严格要求的行业（医疗、政务、金融）。
2. **高昂的 SaaS 订阅费用**：按座位收费模式对中大型团队不友好。
3. **扩展性受限**：封闭生态，无法自定义字段类型或集成私有系统。

### 2.2 Baserow 的解法

Baserow 通过以下设计解决上述问题：

1. **MIT 许可的开源核心** → 消除供应商锁定，允许自托管。
2. **PostgreSQL 作为底层存储** → 数据仍在标准关系数据库中，可随时迁出或直连。
3. **插件注册表系统** → 开发者可在不 Fork 主仓库的情况下扩展字段、视图、集成类型。
4. **API-First 架构** → 所有界面操作均可通过 REST API 复现，支持自动化和第三方集成。

---

## 3. 目标用户画像

### 3.1 主要用户群

| 用户类型 | 描述 | 核心诉求 |
|----------|------|----------|
| **业务操作员** | 运营、市场、HR 人员，日常用表格管理数据 | 简单直观的界面，无需培训即可上手 |
| **业务分析师** | 需要对数据做过滤、排序、聚合、可视化 | 强大的视图、公式、仪表板功能 |
| **运维/IT 管理员** | 负责部署、用户管理、权限配置 | 可靠的自托管方案、LDAP/SSO 集成、细粒度权限 |
| **开发者/集成工程师** | 需要将 Baserow 数据接入其他系统 | 完善的 REST API、Webhook、自动化引擎 |
| **平台构建者** | 使用 Builder 为内部用户创建定制化应用 | 可视化 UI 构建、数据绑定、发布管理 |

### 3.2 使用场景举例

- 初创公司用 Baserow 替代 Excel 管理 CRM 数据
- 医疗机构自托管 Baserow 管理病患档案，满足 HIPAA 合规
- SaaS 公司用 Builder 为客户构建数据录入门户
- 开发团队用 API + Webhook 将 Baserow 作为低代码后端

---

## 4. 核心功能需求

### 4.1 工作区与协作管理

#### 需求描述

工作区（Workspace）是 Baserow 中最顶层的隔离边界，所有应用（数据库、Builder、仪表板）都归属于某一工作区。

#### 功能要求

| # | 功能 | 优先级 | 说明 |
|---|------|--------|------|
| W-01 | 创建、重命名、删除工作区 | P0 | 基础 CRUD |
| W-02 | 邀请用户加入工作区（Email 邀请链接） | P0 | 支持邮件邀请和链接邀请 |
| W-03 | 工作区成员角色（ADMIN / MEMBER） | P0 | ADMIN 可管理成员，MEMBER 只能操作被授权的资源 |
| W-04 | 工作区级别的应用列表 | P0 | 展示工作区下所有应用，支持排序和分组 |
| W-05 | 工作区成员管理（移除成员、修改角色） | P1 | |
| W-06 | 多工作区切换 | P1 | 用户可属于多个工作区 |
| W-07 | 工作区级别权限控制（Premium+） | P2 | 细粒度 ACL，见第 5.6 节 |

#### 设计决策：为什么工作区是最顶层隔离边界？

在设计多租户架构时，有两种主流方案：
- **方案 A（Schema 级隔离）**：每个租户独立 PostgreSQL Schema
- **方案 B（Row 级隔离）**：所有租户共享 Schema，通过 `workspace_id` 外键区分

Baserow 选择方案 B（Row 级隔离），原因如下：
1. 用户数据表在 Baserow 中是**动态创建的**（每张 Baserow 表对应一张 PostgreSQL 表），Schema 级隔离意味着在动态创建表时必须切换 Schema 上下文，实现复杂且容易出现跨 Schema 查询的性能问题。
2. PostgreSQL Row-Level Security (RLS) 可以在 DB 层面提供额外隔离，方案 B 更易与 RLS 结合。
3. 方案 B 下，跨工作区的模板复制、数据迁移逻辑更简单。

---

### 4.2 数据库与表格

#### 需求描述

数据库（Database）是应用类型之一，包含若干"表（Table）"。每张表有若干"字段（Field）"和若干"行（Row）"，类似关系型数据库中的 Table。

#### 字段类型要求

Baserow 支持 40+ 种字段类型，以下为核心类型：

| 分类 | 字段类型 | 说明 |
|------|----------|------|
| **文本** | Text, Long Text, URL, Email, Phone Number | |
| **数值** | Number, Rating, Duration | |
| **日期** | Date, Created/Last Modified Date | |
| **选择** | Single Select, Multiple Select | 枚举值管理 |
| **关系** | Link Row, Lookup, Rollup | 跨表引用与聚合 |
| **文件** | File | 支持多文件上传 |
| **布尔** | Boolean | |
| **计算** | Formula, Count | 表达式引擎 |
| **用户** | Collaborator, Created/Last Modified By | |
| **AI** | AI Field | LLM 驱动的字段，见 4.7 |
| **密码** | Password | 敏感数据存储 |
| **UUID** | Auto-generated UUID | |

#### 行操作要求

| # | 功能 | 优先级 |
|---|------|--------|
| R-01 | 创建、更新、删除行（单行 / 批量） | P0 |
| R-02 | 行级评论与协作（Premium+） | P1 |
| R-03 | 行历史记录（Premium+） | P1 |
| R-04 | 跨表行关联（Link Row） | P0 |
| R-05 | 行导入（CSV、XML、JSON） | P0 |
| R-06 | 行导出 | P0 |
| R-07 | Undo / Redo 操作 | P1 |

#### 设计决策：为什么每张 Baserow 表对应一张独立 PostgreSQL 表？

备选方案是使用 EAV（Entity-Attribute-Value）模式，把所有行数据存储在一张大表中，通过 `field_id` 区分列。

Baserow 选择**每张逻辑表对应一张物理 PostgreSQL 表**，原因：
1. **查询性能**：EAV 模式下，读取一行数据需要多行 JOIN，对大数据量表极其低效。独立物理表下，常规 `SELECT * WHERE id=X` 即可。
2. **索引灵活性**：可以对每个字段单独建索引，而 EAV 下无法对"某个字段的值"建高效索引。
3. **PostgreSQL 原生能力**：利用 PostgreSQL 的类型系统、JSONB、数组、全文索引等特性，无需在应用层模拟。
4. **数据可移植性**：用户可以直接用 psql 连接并查询自己的数据，不依赖 Baserow 特定的数据格式。

代价是表的元数据管理（字段 DDL 变更）需要额外的迁移管理逻辑，Baserow 通过 `FieldHandler` 统一处理所有 DDL 操作来解决这一问题。

---

### 4.3 视图系统

#### 需求描述

视图（View）是对同一张表数据的不同"呈现方式"，包含独立的过滤（Filter）、排序（Sort）、分组（Group）、显示字段配置。

#### 视图类型

| 视图类型 | 说明 | 适用场景 |
|----------|------|----------|
| **Grid（表格）** | 类 Excel 的行列视图，默认视图 | 数据录入、浏览 |
| **Form（表单）** | 可发布的数据录入表单 | 外部数据收集 |
| **Gallery（画廊）** | 以卡片形式展示，适合图片/媒体数据 | 素材管理 |
| **Kanban（看板）** | 按 Single Select 字段分列的看板 | 项目管理 |
| **Calendar（日历）** | 按日期字段展示的日历视图（Premium+） | 时间规划 |
| **Timeline（时间线）** | 甘特图风格的时间视图（Premium+） | 项目排期 |

#### 过滤与排序要求

- 支持多条件 AND / OR 过滤
- 过滤操作符覆盖：等于、包含、为空、大于/小于、日期范围等
- 支持多列排序，排序优先级可调
- 支持按字段分组（Group By），组内可折叠

#### 设计决策：视图配置存储在数据库而非客户端

有些产品将视图过滤/排序配置保存在浏览器 localStorage，Baserow 选择将所有视图配置持久化在 PostgreSQL。

原因：
1. **多设备一致性**：用户换设备或浏览器后，配置不丢失。
2. **协作共享**：团队成员打开同一视图看到相同的配置，便于协作。
3. **API 支持**：视图配置可通过 API 读写，支持自动化脚本管理视图。

---

### 4.4 应用构建器（Builder）

#### 需求描述

Builder 允许用户通过可视化拖拽的方式，将 Baserow 数据库中的数据展现为自定义应用页面，对外发布为门户（Portal）或内部工具。

#### 核心要求

| # | 功能 | 优先级 |
|---|------|--------|
| B-01 | 可视化页面编辑器（拖拽组件） | P0 |
| B-02 | 组件库（文本、按钮、表单、表格、图表等） | P0 |
| B-03 | 数据源绑定（绑定到 Baserow 表格或外部 API） | P0 |
| B-04 | 页面导航与路由管理 | P0 |
| B-05 | 应用发布（生成可公开访问的 URL） | P0 |
| B-06 | 应用内用户认证（访客/登录用户） | P1 |
| B-07 | 条件显示（按数据或用户角色控制组件可见性） | P1 |
| B-08 | 事件处理（点击按钮触发行操作、导航、通知） | P1 |
| B-09 | 自定义域名（Enterprise+） | P2 |
| B-10 | 多语言支持 | P2 |

#### 设计决策：Builder 作为独立应用类型而非数据库的附属功能

早期设计中曾考虑将 Builder 嵌入到 Database 模块中作为一种"视图"。最终选择作为独立应用类型的原因：

1. **职责分离**：Database 负责数据存储和结构管理，Builder 负责用户界面构建，两者关注点不同。
2. **多数据源**：Builder 页面可以绑定多个 Database 表，甚至外部 REST API，无法被单一 Database 所包含。
3. **独立发布生命周期**：Builder 应用有自己的发布（publish）状态和版本概念，与 Database 的编辑操作解耦。
4. **扩展性**：Builder 设计为可扩展的组件注册表，第三方开发者可以注册自定义组件，这个扩展点独立于 Database 的字段类型注册表。

---

### 4.5 自动化工作流（Automation）

#### 需求描述

Automation 允许用户配置"触发器 → 条件 → 动作"的工作流，在特定事件发生时自动执行任务。

#### 触发器类型

| 触发器 | 说明 |
|--------|------|
| **行创建** | Baserow 表中新增行时触发 |
| **行更新** | 行字段值变更时触发，可指定字段 |
| **行删除** | 行被删除时触发 |
| **定时触发** | Cron 表达式定义的定时任务 |
| **Webhook 入站** | 外部系统发送 HTTP 请求触发 |

#### 动作类型

| 动作 | 说明 |
|------|------|
| **创建行** | 在指定表中创建新行 |
| **更新行** | 更新指定行的字段值 |
| **删除行** | 删除指定行 |
| **发送邮件** | 发送邮件通知 |
| **HTTP 请求** | 调用外部 REST API |
| **AI 动作** | 调用 LLM 处理数据 |

#### 设计决策：使用 Celery 异步执行自动化任务

自动化任务（尤其是 Webhook 调用、邮件发送、AI 处理）不能同步执行，原因：

1. **防止请求超时**：HTTP 请求处理时间有限（通常 30s），同步执行复杂工作流会超时。
2. **可靠性**：Celery 任务队列支持重试、失败记录、任务状态跟踪，确保自动化任务最终执行成功。
3. **解耦**：触发事件（数据库写操作）与执行（外部调用）解耦，数据库操作的成功不依赖于外部服务的可用性。
4. **扩展性**：可以通过增加 Celery Worker 水平扩展自动化处理能力，无需扩展 API 服务器。

---

### 4.6 仪表板（Dashboard）

#### 需求描述

仪表板允许用户将多个数据可视化组件（Widget）组合在一个页面，用于数据监控和决策支持。

#### 核心 Widget 类型

| Widget | 说明 |
|--------|------|
| **Summary Widget** | 显示单个聚合数值（总数、求和、平均值等） |
| **Chart Widget** | 柱状图、折线图、饼图等 |
| **Table Widget（规划中）** | 内嵌表格数据 |

#### 数据源

- Baserow 数据库表（通过 Service 层抽象）
- 支持过滤条件，使 Widget 只展示相关数据子集

#### 设计决策：通过 Service 层抽象数据源

仪表板 Widget 的数据来源被抽象为"Service"（数据服务），而不是直接绑定到 Baserow 表。

原因：
1. **未来扩展**：Service 抽象可以支持 CSV 导入、外部 API、GraphQL 等数据源，而不仅限于 Baserow 表。
2. **复用**：同一个 Service 配置可以被多个 Widget 甚至多个 Builder 页面组件引用。
3. **与 Builder 统一**：Builder 组件也使用相同的 Service 层绑定数据，代码和概念统一。

---

### 4.7 AI 能力集成

#### 需求描述

Baserow 将 AI 能力深度集成到产品中，覆盖字段级 AI 处理、自然语言操作、AI 驱动的自动化。

#### AI 字段（AI Field）

- 用户可配置提示词（Prompt）模板，引用同行其他字段的值
- 每次行更新时，AI 字段可自动或手动重新计算
- 支持 OpenAI、Anthropic、Mistral 等多家 LLM 提供商

#### AI 助手（Kuma）

- 通过自然语言创建数据库结构（"帮我创建一个项目管理数据库"）
- 通过自然语言查询数据（"找出所有逾期超过 7 天的任务"）
- 生成 Formula 表达式

#### 设计决策：LLM 提供商抽象层

Baserow 通过 LangChain 统一封装多家 LLM 提供商，而不是直接集成 OpenAI SDK。

原因：
1. **避免供应商锁定**：用户可以根据成本、隐私、合规需求选择不同提供商。
2. **统一接口**：LangChain 提供统一的 Chain/Agent 接口，Baserow 代码无需为每个提供商单独处理 API 差异。
3. **私有化部署支持**：支持 Ollama 等本地运行的开源模型，满足数据不出境的需求。

---

## 5. 技术架构设计与设计决策

### 5.1 整体架构

```
┌─────────────────────────────────────────────────────────┐
│                     浏览器客户端                          │
│              Nuxt 3 (Vue 3 + Vuex + TypeScript)          │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTP REST / WebSocket
┌──────────────────────▼──────────────────────────────────┐
│                   API 网关层                              │
│   Django ASGI (Daphne/Uvicorn) + Django Channels         │
│   ┌──────────────────────────────────────────────────┐  │
│   │           Django REST Framework                   │  │
│   │   /api/workspaces/  /api/database/  /api/builder/ │  │
│   └──────────────────────────────────────────────────┘  │
│   ┌──────────────────────────────────────────────────┐  │
│   │       WebSocket Consumer (Django Channels)        │  │
│   └──────────────────────────────────────────────────┘  │
└────────────┬────────────────────────┬───────────────────┘
             │                        │
┌────────────▼──────────┐   ┌─────────▼────────────────────┐
│    业务逻辑层（Handler）│   │      异步任务层（Celery）       │
│  CoreHandler           │   │  自动化工作流执行               │
│  DatabaseHandler       │   │  文件处理、导入导出              │
│  BuilderHandler        │   │  Webhook 触发                  │
│  AutomationHandler     │   │  AI 字段重计算                  │
└────────────┬──────────┘   └─────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────┐
│                    数据层                                 │
│   PostgreSQL（核心业务数据 + 用户表格数据）                  │
│   Redis（Session 缓存 + Celery Broker + WebSocket 通道）  │
│   Object Storage（S3 / Azure / GCS - 文件存储）           │
└─────────────────────────────────────────────────────────┘
```

### 5.2 后端设计决策

#### 5.2.1 为什么选择 Django + Django REST Framework？

1. **成熟稳定**：Django 是 Python 生态中最成熟的 Web 框架，有完善的 ORM、迁移管理、认证系统。
2. **DRF 的序列化与验证**：DRF Serializer 提供了声明式的请求验证和响应序列化，减少样板代码。
3. **Django Channels 支持**：同一框架可以同时处理 HTTP 和 WebSocket，无需引入独立的 WebSocket 服务。
4. **强大的迁移系统**：Baserow 的表结构管理涉及大量 DDL 操作，Django Migrations 提供了可靠的版本管理。
5. **插件生态**：django-rest-framework、djangorestframework-simplejwt、drf-spectacular 等插件覆盖了常见需求。

#### 5.2.2 Handler 模式（为什么业务逻辑不写在 View 中？）

Baserow 使用 `XxxHandler` 类（如 `CoreHandler`、`DatabaseHandler`）封装所有业务逻辑，API View 只负责请求解析和响应序列化。

原因：
1. **可测试性**：Handler 方法可以在不依赖 HTTP 上下文的情况下被单元测试直接调用。
2. **复用性**：Celery 任务、WebSocket 处理、Management Command 都可以直接调用 Handler，不需要模拟 HTTP 请求。
3. **关注点分离**：View 层负责 HTTP 协议细节，Handler 层负责业务规则，两者变化原因不同，分离符合 SRP（单一职责原则）。
4. **Action / Undo-Redo**：Handler 方法是 Action 系统（撤销/重做）的基础操作单元，将逻辑集中在 Handler 使得 Action 的封装更简单。

#### 5.2.3 Action / Undo-Redo 系统设计

每个用户操作（创建行、修改字段等）被封装为一个 `Action` 类，Action 包含：
- `do()` - 执行操作
- `undo()` - 撤销操作，恢复到执行前状态
- `redo()` - 重新执行

Action 执行记录持久化到 `Action` 模型中，支持跨会话的撤销历史。

设计决策原因：
- **协作场景**：多人同时编辑时，需要知道每个操作的完整上下文才能安全撤销，简单的栈结构不够。
- **可审计性**：Action 记录同时作为操作日志，满足合规审计需求。
- **可测试性**：每个 Action 的 do/undo 可以独立测试。

#### 5.2.4 多态模型（Polymorphic Models）

Application、Field、View 等实体使用 Django 的多态模型继承（`django-model-utils` 的 `PolymorphicModel`）：
- `Application` → `Database`、`Builder`、`Dashboard`、`Automation`
- `Field` → `TextField`、`NumberField`、`DateField`、`FormulaField` 等 40+ 子类
- `View` → `GridView`、`GalleryView`、`KanbanView` 等

原因：
1. **类型安全**：每种类型有自己专属字段，不需要在通用表中用 JSONB 存储差异化配置（避免"万能JSONB"反模式）。
2. **可扩展性**：添加新的 Field 类型只需新增一个 Python 类和对应的 Django Model，无需修改核心表结构。
3. **ORM 查询**：可以通过基类（如 `Field`）进行跨类型查询，也可以通过子类精确查询特定类型。

代价是多态查询会引入额外 JOIN，Baserow 通过 `select_related`、`prefetch_related` 和 `specific()` 等 ORM 优化手段缓解性能问题。

### 5.3 前端设计决策

#### 5.3.1 为什么选择 Nuxt 3 / Vue 3？

1. **SSR + SSG 支持**：Nuxt 3 支持服务端渲染，有利于 SEO 和首屏性能。
2. **Composition API**：Vue 3 的 Composition API 使逻辑复用更清晰，相比 Options API 更易维护复杂组件。
3. **模块化架构**：Nuxt 3 的 Module 系统与 Baserow 的插件架构契合，核心功能、高级功能（Premium）、企业功能可以作为独立模块加载。
4. **TypeScript 支持**：Vue 3 + Nuxt 3 有完善的 TypeScript 类型支持，提升代码质量。
5. **生态成熟**：Vuex 4、vue-router 4 与 Nuxt 3 良好集成。

#### 5.3.2 Vuex 模块化状态管理

前端状态按模块组织（`workspace`、`database`、`builder` 等），每个模块管理自己的状态切片。

原因：
1. **实时协作**：WebSocket 接收到服务端推送时，需要精确更新 Store 中的特定状态，模块化使得更新目标明确。
2. **可预测的状态变更**：所有状态变更通过 Mutation 进行，便于调试和时间旅行调试（Vue DevTools）。
3. **服务端状态同步**：Store 是服务端数据在前端的缓存，模块化使缓存失效逻辑清晰。

#### 5.3.3 为什么 Grid View 不使用虚拟 DOM 框架的标准渲染？

Grid View（类 Excel 表格）是性能要求最高的组件，对于有数万行数据的表格，标准的 Vue v-for 渲染会导致严重的性能问题。

Baserow 为 Grid View 实现了**虚拟滚动（Virtual Scrolling）**：只渲染视口内可见的行，其余行用占位符填充。

原因：
- 渲染 DOM 节点的成本远高于 JavaScript 计算，减少 DOM 节点数量是性能优化的关键。
- 用户感知的"行数"可以达到数百万行，但视口内实际可见不超过 50-100 行。

### 5.4 数据库设计决策

#### 5.4.1 用户表格数据的存储方式

如前所述，每张 Baserow 逻辑表对应一张独立的 PostgreSQL 物理表，命名规则为 `database_table_{table_id}`。

字段命名规则：`field_{field_id}`（如 `field_123`）。

为什么不用字段名（如 `name`、`email`）作为列名？
- 用户可以随时重命名字段，若以字段名作为列名则每次重命名都需要 `ALTER TABLE RENAME COLUMN`，风险高。
- 使用不变的 `field_id` 作为列名，字段重命名只是元数据变更，无需 DDL。

#### 5.4.2 为什么选择 PostgreSQL 而非 MySQL？

1. **JSON/JSONB**：PostgreSQL 的 JSONB 类型用于存储字段元数据和视图配置，性能和查询能力优于 MySQL 的 JSON。
2. **Array 类型**：Multiple Select 等字段使用 PostgreSQL Array，MySQL 没有原生数组类型。
3. **全文搜索**：PostgreSQL 内置全文搜索，无需额外引入 Elasticsearch。
4. **pgvector 扩展**：AI 功能需要向量相似度搜索，pgvector 是 PostgreSQL 的官方扩展，MySQL 无对应方案。
5. **Window Functions & CTE**：复杂的查询（如排名、层级结构）使用 PostgreSQL 的窗口函数更高效。
6. **LISTEN/NOTIFY**：PostgreSQL 的 pub/sub 机制可以用于实时变更通知。

#### 5.4.3 软删除（Trash / Soft Delete）

删除工作区、数据库、表、行等操作不立即物理删除，而是移入"垃圾桶（Trash）"并保留 30 天，期间可以恢复。

实现方式：`TrashEntry` 模型记录被删除对象的类型和 ID，被删除对象在数据库中标记 `trashed=True`。

原因：
- **误操作保护**：用户意外删除重要数据后可以恢复，大幅降低用户投诉和数据丢失风险。
- **协作安全**：多人协作时，一个用户的误操作不会立即影响其他人，有缓冲恢复窗口。
- **数据恢复成本**：比起从备份恢复，软删除恢复几乎零成本。

### 5.5 实时通信设计

#### 5.5.1 WebSocket + Django Channels 架构

实时推送（多用户协作时看到彼此的变更）通过 Django Channels + Redis 实现：

1. 用户 A 对行进行修改 → 发送 REST API 请求
2. Handler 执行业务逻辑，保存到数据库
3. Handler 向 Redis Channel Layer 发布事件
4. Django Channels Consumer 接收事件，推送 WebSocket 消息到相关客户端（订阅了相同 Table 的用户）

为什么选择 WebSocket 而非 Server-Sent Events (SSE) 或轮询（Polling）？
- **双向通信**：WebSocket 支持客户端主动发送消息（未来协作功能可能需要），SSE 只支持单向推送。
- **低延迟**：WebSocket 的延迟远低于长轮询。
- **连接效率**：一个 WebSocket 连接可以推送多种类型的事件，而 SSE 通常针对单一数据流。

#### 5.5.2 Redis 的角色

Redis 在 Baserow 中承担三个角色：
1. **Django 会话缓存**：减少数据库会话查询压力
2. **Celery Broker**：任务队列消息传递
3. **Django Channels Layer**：WebSocket 消息广播（多实例部署时同步）

将三个用途共享同一 Redis 实例（默认）vs. 分开配置，取决于部署规模：
- 单机部署：共享一个 Redis，简化运维
- 大规模部署：建议分开，避免相互影响

### 5.6 权限系统设计

#### 5.6.1 权限层级

Baserow 的权限系统按以下层级设计：

```
System Level（系统管理员）
  └── Workspace Level（工作区管理员 / 成员）
        └── Application Level（应用级权限）
              └── Table Level（表级权限）
                    └── Field Level（字段级权限）
                          └── Row Level（行级权限）
```

开源版本支持：工作区级别（ADMIN / MEMBER）和简单的应用权限。
Premium 版本支持：完整的多层级 ACL，包括字段级和行级权限。

#### 5.6.2 Permission Manager 抽象

权限判断通过 `PermissionManager` 接口实现，不同版本注册不同的实现：
- 开源版：`BasicPermissionManager`（基于工作区角色的简单判断）
- Premium 版：`StaffOnlyPermissionManager` + 细粒度 ACL

为什么使用接口抽象而不是硬编码权限逻辑？
- 允许企业客户注册自定义权限管理器，与其 LDAP / AD 系统集成
- 测试时可以注入 `AllowAllPermissionManager` 来跳过权限检查
- 开源版和 Premium 版使用相同的代码路径，差异在于注册的 Manager 实现

### 5.7 插件与扩展性设计

#### 5.7.1 注册表系统（Registry Pattern）

Baserow 的扩展点通过"注册表"实现：

```python
# 字段类型注册
field_type_registry.register(TextFieldType())
field_type_registry.register(NumberFieldType())

# 视图类型注册
view_type_registry.register(GridViewType())
view_type_registry.register(GalleryViewType())

# 认证提供商注册
auth_provider_type_registry.register(GoogleOAuthProviderType())
```

第三方插件可以通过注册自定义类型来扩展 Baserow，无需 Fork 主仓库。

#### 5.7.2 为什么选择注册表模式而非继承或配置文件？

1. **运行时扩展**：注册表在 Django `AppConfig.ready()` 中初始化，支持通过安装 Python Package 来添加新类型。
2. **解耦**：核心代码不需要知道所有具体类型的存在，只需要通过注册表查找。
3. **类型安全**：每个注册类型必须实现特定的接口（Protocol/ABC），IDE 可以提供完整的类型检查。
4. **可测试性**：测试时可以向注册表注册 Mock 类型，模拟各种边界情况。

#### 5.7.3 Premium / Enterprise 功能隔离

Premium 和 Enterprise 功能代码位于独立的 `/premium` 和 `/enterprise` 目录，通过 Django App 和注册表机制注入到核心中。

核心代码永远不直接 `import` Premium 代码，而是通过注册表查找。这样：
- 开源版可以不安装 Premium Package 正常运行
- Premium 功能通过许可证 Key 激活，而不是通过代码修改
- Premium/Enterprise 可以有独立的发布周期

---

## 6. 非功能性需求

### 6.1 性能需求

| 指标 | 目标值 | 说明 |
|------|--------|------|
| API 响应时间（P50） | < 100ms | 常规 CRUD 操作 |
| API 响应时间（P95） | < 500ms | 包含复杂查询 |
| Grid View 首屏加载 | < 2s | 100 行以内的表格 |
| Grid View 滚动帧率 | 60fps | 虚拟滚动场景 |
| 并发用户支持 | 1000 concurrent | 单机部署参考值 |
| 单表最大行数 | 无硬限制（百万级验证） | 受 PostgreSQL 能力限制 |

### 6.2 可靠性需求

| 指标 | 目标值 |
|------|--------|
| 服务可用性（SaaS） | 99.9% |
| 数据持久性 | 99.9999% |
| 备份频率 | 每日自动备份 |
| 灾难恢复 RTO | < 4 小时 |
| 软删除保留期 | 30 天 |

### 6.3 安全需求

| 需求 | 说明 |
|------|------|
| 传输加密 | 所有 HTTPS，WebSocket 使用 WSS |
| 静态加密 | 数据库、文件存储支持静态加密 |
| 认证方式 | JWT（短期 Access Token + Refresh Token）+ Session |
| 双因素认证 | TOTP（Authenticator App）+ Email OTP |
| SSO 集成 | SAML 2.0、OpenID Connect（Google、GitHub 等） |
| API Token | 数据库级别的访问 Token，支持读写权限分离 |
| 密码策略 | 最小长度、复杂度要求可配置（Enterprise+） |
| 会话管理 | JWT 黑名单机制，支持强制踢出会话 |
| CSRF 防护 | Django 内置 CSRF 防护 |
| XSS 防护 | 前端输出 HTML 转义，Content Security Policy |
| SQL 注入防护 | ORM 参数化查询，禁止原始 SQL 拼接 |

### 6.4 可扩展性需求

| 维度 | 方案 |
|------|------|
| **水平扩展 API** | Django 无状态设计，可以多实例部署在负载均衡后 |
| **水平扩展 Celery** | 增加 Worker 节点 |
| **数据库扩展** | PostgreSQL Read Replica 分离读写 |
| **文件存储扩展** | 对接云存储（S3/Azure/GCS），无本地文件系统依赖 |
| **缓存扩展** | Redis Cluster 支持 |

### 6.5 可观测性需求

| 能力 | 实现 |
|------|------|
| **分布式追踪** | OpenTelemetry，支持 Jaeger/Zipkin |
| **错误监控** | Sentry SDK 集成（可配置 DSN） |
| **指标采集** | Prometheus 兼容的 Metrics Endpoint |
| **日志** | 结构化日志（JSON 格式），支持 ELK 接入 |
| **健康检查** | `/api/health/` 端点，用于 Kubernetes Liveness/Readiness |
| **用户行为分析** | PostHog（可选，支持自托管 PostHog） |

### 6.6 部署需求

| 部署方式 | 支持状态 |
|----------|---------|
| Docker Compose（单机） | ✅ 完整支持，提供生产级 compose 文件 |
| Kubernetes（Helm Chart） | ✅ 完整支持，提供官方 Helm Chart |
| Heroku | ✅ 一键部署按钮 |
| Render | ✅ 一键部署按钮 |
| AWS / DigitalOcean / Railway | ✅ 文档指引 |
| Cloudron | ✅ 支持 |

### 6.7 国际化需求

- 界面支持多语言（通过 `@nuxtjs/i18n`）
- 日期/时间字段支持时区配置
- 数字格式支持本地化（千分符、小数点）
- 优先支持语言：英语（默认）、中文、德语、法语、荷兰语、西班牙语

---

## 7. 开发规范与约定

### 7.1 后端开发规范

#### 7.1.1 代码组织规范

```
backend/src/baserow/
├── api/                    # REST API 层（Views, Serializers, URLs）
│   └── {feature}/
│       ├── views.py        # API 视图（仅负责请求/响应处理）
│       ├── serializers.py  # 请求/响应序列化
│       └── urls.py         # URL 路由
├── contrib/{feature}/      # 功能模块
│   ├── handler.py          # 业务逻辑（所有业务写这里）
│   ├── models.py           # 数据模型
│   ├── exceptions.py       # 自定义异常
│   ├── types.py            # Python 类型定义
│   ├── registries.py       # 注册表定义
│   └── field_types.py      # 具体类型实现（字段/视图/等）
└── core/                   # 核心通用模块
```

#### 7.1.2 命名规范

| 元素 | 规范 | 示例 |
|------|------|------|
| 类名 | PascalCase | `CoreHandler`, `TextFieldType` |
| 函数/方法 | snake_case | `create_table()`, `get_workspace()` |
| 变量 | snake_case | `workspace_id`, `field_type` |
| 常量 | UPPER_SNAKE_CASE | `MAX_FIELDS_PER_TABLE = 1500` |
| 数据库表（Baserow 用户表） | `database_table_{id}` | `database_table_42` |
| 数据库列（Baserow 用户字段） | `field_{id}` | `field_123` |
| URL pattern | kebab-case | `/api/database/rows/table/{id}/` |

#### 7.1.3 API 设计规范

- 遵循 RESTful 风格，资源名词复数，HTTP 动词语义化
- 分页：统一使用 `page` + `size` 参数，或 `cursor` 游标分页（大数据集）
- 错误响应格式：
  ```json
  {
    "error": "ERROR_CODE_IN_UPPER_SNAKE",
    "detail": "Human readable message"
  }
  ```
- 所有 API 必须在 drf-spectacular 中添加文档注解（`@extend_schema`）
- 新增 API 须在 `changelog.md` 中标注

#### 7.1.4 测试规范

- 每个 Handler 方法需有对应的单元测试（`tests/baserow/contrib/{feature}/test_{feature}_handler.py`）
- 每个 API View 需有对应的集成测试（`tests/baserow/api/{feature}/test_{feature}_views.py`）
- 测试使用 `pytest` + `pytest-django`，禁止使用 `unittest.TestCase`
- 测试覆盖率目标：核心功能 > 80%
- 测试中使用 `pytest.fixture` 管理测试数据，禁止在测试函数中直接创建业务对象（应通过 Fixture）
- 测试不得依赖测试执行顺序

#### 7.1.5 数据库迁移规范

- 每次 Model 修改必须生成对应 Migration 文件
- Migration 文件必须是可逆的（提供 `reverse` 操作）
- 大表字段添加必须使用 `db_default` 而非 Python 层 default，避免锁表
- 重命名字段必须分两步完成：先添加新字段，数据迁移后再删除旧字段

### 7.2 前端开发规范

#### 7.2.1 代码组织规范

```
web-frontend/modules/{feature}/
├── components/         # Vue 组件
│   ├── {Feature}.vue   # 业务组件
│   └── {Feature}/      # 子组件目录
├── pages/              # Nuxt 路由页面
├── store/              # Vuex Store 模块
├── services/           # API 调用服务
├── mixins/             # Vue Mixins（遗留，新代码优先用 Composables）
├── composables/        # Vue Composition API
└── assets/             # 静态资源（SCSS、图片）
```

#### 7.2.2 组件规范

- 优先使用 Composition API（`<script setup>`）
- 组件 Props 必须有 TypeScript 类型定义
- 组件 Emits 必须有明确的事件类型
- 禁止在组件中直接调用 `axios`，必须通过 `services/` 层
- 禁止在组件中直接修改 Vuex State，必须通过 `commit` Mutation 或 `dispatch` Action

#### 7.2.3 状态管理规范

- Vuex Store 模块按功能模块组织
- 服务端数据必须经过 Store，禁止组件本地缓存服务端数据（避免与 WebSocket 实时更新不一致）
- WebSocket 消息处理在 Store Action 中完成

#### 7.2.4 测试规范

- 组件测试使用 Vitest + `@vue/test-utils`
- 每个关键 Store Action / Mutation 需有单元测试
- 测试文件与被测文件同级（`components/Foo.vue` 对应 `test/unit/components/Foo.spec.js`）

---

## 8. 分层功能边界（Open Source / Premium / Enterprise）

| 功能 | 开源版（MIT） | Premium | Enterprise |
|------|:---:|:---:|:---:|
| 核心数据库与表格 | ✅ | ✅ | ✅ |
| Grid / Form / Gallery / Kanban 视图 | ✅ | ✅ | ✅ |
| REST API | ✅ | ✅ | ✅ |
| 无限工作区、数据库、表 | ✅ | ✅ | ✅ |
| Webhook | ✅ | ✅ | ✅ |
| 数据导入/导出 | ✅ | ✅ | ✅ |
| Automation（工作流自动化） | ✅（限制） | ✅ | ✅ |
| Builder（应用构建器） | ✅（限制） | ✅ | ✅ |
| Dashboard（仪表板） | ✅（限制） | ✅ | ✅ |
| Calendar / Timeline 视图 | ❌ | ✅ | ✅ |
| 行历史记录 | ❌ | ✅ | ✅ |
| 行级评论 | ❌ | ✅ | ✅ |
| 字段级权限 | ❌ | ✅ | ✅ |
| 行级权限 | ❌ | ❌ | ✅ |
| 高级权限（团队、角色） | ❌ | ✅ | ✅ |
| SAML / SSO | ❌ | ❌ | ✅ |
| 审计日志 | ❌ | ❌ | ✅ |
| 白标（Custom Branding） | ❌ | ❌ | ✅ |
| 自定义域名（Builder） | ❌ | ❌ | ✅ |
| AI 字段 | ❌ | ✅ | ✅ |
| 技术支持 SLA | 社区 | 邮件支持 | 专属支持 |

---

## 9. 成功度量指标

### 9.1 产品增长指标

| 指标 | 描述 | 目标（12个月） |
|------|------|----------------|
| **MAU（月活用户）** | 每月至少登录一次的用户数 | 持续增长 30% YoY |
| **工作区创建数** | 新创建的工作区数量（代表新团队注册） | 持续增长 |
| **行写入量** | 每月写入的行数（代表实际数据使用深度） | 持续增长 |
| **自托管部署数** | 通过 Docker Hub 拉取镜像的部署估算 | 持续增长 |
| **GitHub Stars** | 开源社区认可度 | 持续增长 |

### 9.2 产品质量指标

| 指标 | 描述 | 目标 |
|------|------|------|
| **API 错误率** | 5xx 错误 / 总请求数 | < 0.1% |
| **P95 API 延迟** | 第95百分位 API 响应时间 | < 500ms |
| **测试覆盖率** | 后端核心模块代码覆盖率 | > 80% |
| **关键 Bug TTR** | P0 Bug 从报告到修复的时间 | < 24 小时 |
| **开放 Issue 数** | GitHub 未关闭 Bug Issue 数量 | 稳定可控 |

### 9.3 开发者生态指标

| 指标 | 描述 | 目标 |
|------|------|------|
| **社区插件数** | 社区发布的第三方插件数量 | 增长趋势 |
| **API 文档完整性** | 所有端点有文档注解 | 100% |
| **外部贡献者数** | 非核心团队提交 PR 的开发者数 | 增长趋势 |

---

## 10. 附录：术语表

| 术语 | 定义 |
|------|------|
| **Workspace** | 工作区，Baserow 中最顶层的多租户隔离单元，类比 Notion 的 Workspace 或 GitHub 的 Organization |
| **Application** | 应用，工作区下的资源单元，包含 Database、Builder、Dashboard、Automation 四种类型 |
| **Database** | 数据库应用类型，包含若干 Table |
| **Table** | 表，对应 PostgreSQL 中一张物理表，包含 Field（列）和 Row（行） |
| **Field** | 字段（列），每个字段有特定类型（文本、数字、日期等），元数据存储在 `database_field` 表中，实际数据列命名为 `field_{id}` |
| **Row** | 行，用户数据的基本单元，存储在对应的物理表中 |
| **View** | 视图，对同一 Table 数据的不同呈现方式，包含独立的过滤/排序配置 |
| **Builder** | 应用构建器，可视化构建自定义 Web 应用的工具 |
| **Automation** | 自动化，配置触发器 + 动作的工作流引擎 |
| **Dashboard** | 仪表板，数据可视化看板 |
| **Handler** | 业务逻辑处理器，如 `CoreHandler`、`DatabaseHandler`，是 Baserow 后端的核心业务层 |
| **Registry** | 注册表，用于管理可扩展类型（字段类型、视图类型等）的中心化注册机制 |
| **Action** | 操作，可撤销/重做的用户操作单元，持久化在数据库中 |
| **Trash** | 垃圾桶，软删除机制，被删除的资源在 30 天内可恢复 |
| **Service** | 数据服务，Builder 和 Dashboard 组件绑定数据的抽象层 |
| **Premium** | 高级版，付费功能层，在开源核心之上提供额外功能 |
| **Enterprise** | 企业版，面向大型组织的高级版，提供 SSO、审计日志、高级权限等企业级特性 |
| **MCP** | Model Context Protocol，AI 代理与 Baserow 交互的协议接口 |
| **pgvector** | PostgreSQL 向量扩展，用于 AI Embedding 存储和相似度搜索 |

---

*本文档由 Baserow 核心团队维护，如有疑问请在 GitHub Issues 提交，标签使用 `documentation`。*
