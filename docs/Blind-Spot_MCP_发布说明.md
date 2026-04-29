# Blind-Spot MCP 发布说明

FeedMob 内部存档文档

---

## 一、基本发布信息

| 字段 | 详细信息 |
|------|----------|
| MCP 名称 | Blind-Spot MCP |
| 版本 | v1.0（初始发布） |
| 发布日期 | 2026 年 4 月 24 日 |
| 发布平台 | Coolify（通过 npx mcp-remote 部署） |
| 发布账号 | blind-spot-demo（Vercel Serverless） |
| 发布人 | Yongcheng |
| 所属公司 | FeedMob |
| 适用部门 | 数据分析团队 |
| 文档创建日期 | 2026 年 4 月 28 日 |

---

## 二、项目背景与目的

### 2.1 背景

Blind-Spot MCP 是 FeedMob 内部自研的 Model Context Protocol 服务，基于 Vercel Serverless 部署，旨在将广告主竞争情报数据库通过标准化 MCP 接口暴露给 AI 助手（如 Claude），实现自然语言驱动的数据查询与竞品分析。

该服务是 AdIntel MCP 的轻量级版本，托管于 Vercel 平台，适合快速验证和原型开发。

### 2.2 核心目的

- 通过 MCP 协议将广告主竞争情报数据统一接入 AI 工作流
- 使分析师能够以对话方式完成广告主信息查询、市场排名分析等任务
- 提供轻量级 MCP 部署方案，降低部署门槛

---

## 三、技术架构

### 3.1 部署环境

- **部署平台**: Vercel（Serverless Functions）+ Coolify（npx mcp-remote 代理）
- **账号类型**: Vercel 托管服务（blind-spot-demo）
- **访问方式**: HTTP-only MCP 端点
- **MCP 端点**: `https://blindspot-demo-mcp.vercel.app/api/mcp`
- **连接方式**: `npx mcp-remote --transport http-only https://blindspot-demo-mcp.vercel.app/api/mcp`

### 3.2 数据源集成

Blind-Spot MCP 复用 AdIntel 后端数据库，提供以下数据访问：

| 数据平台 | 数据类型 | 主要用途 |
|----------|----------|----------|
| AdIntel DB | 广告主基础信息 | 广告主列表、元数据查询 |
| AdIntel DB | SensorTower 数据 | 应用下载量、营收、市场排名 |
| AdIntel DB | 市场排名数据 | 按网络的应用/广告主排名 |

### 3.3 数据库

底层使用 AdIntel 关系型数据库，核心数据表包括广告主信息表、SensorTower 采集数据表、市场排名数据表等。

---

## 四、MCP 工具清单

本 MCP 共提供 **11 个**标准工具，按功能分为以下四大类：

### 4.1 广告主数据查询（3 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `list_advertisers` | 列出 AdIntel 中当前存储的广告主及数据新鲜度元数据 |
| `get_advertiser_summary` | 获取指定广告主的 SensorTower 综合摘要，包括下载量、营收等核心指标 |
| `run_query` | 对 AdIntel 数据库执行只读 SQL 查询 |

### 4.2 市场排名分析（2 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `get_market_top_apps` | 从 SensorTower 获取市场整体应用排名，支持按类别、国家、排序方式筛选 |
| `get_market_top_advertisers` | 按广告投放获取市场广告主排名，支持按网络、类别、国家筛选 |

### 4.3 广告主申请与管理（2 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `request_advertiser` | 记录缺失广告主的纳入申请 |
| `list_requested_advertisers` | 列出已申请但尚未纳入的广告主 |

### 4.4 资源与 Prompt（4 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `list_resources` | 列出 MCP 服务器上可用的资源 |
| `read_resource` | 按 URI 读取指定资源内容 |
| `list_prompts` | 列出 MCP 服务器上可用的 prompts |
| `get_prompt` | 按名称获取指定 prompt |

---

## 五、功能模块说明

### 5.1 广告主数据查询

通过 `list_advertisers`、`get_advertiser_summary` 等工具，支持列举所有已追踪广告主、查询指定广告主的 SensorTower 综合指标摘要（下载量、营收、市场排名等）。

### 5.2 市场排名分析

通过 `get_market_top_apps` 支持查询市场应用排名，可按下载量、营收、DAU 或印象份额排序。`get_market_top_advertisers` 支持查询广告投放排名，可按网络（Admob、Facebook、TikTok 等）筛选。

### 5.3 SQL 查询

`run_query` 工具支持对 AdIntel 数据库执行只读 SQL 查询，允许高级用户直接编写 SQL 进行复杂分析。

### 5.4 广告主申请与管理

支持通过 `request_advertiser` 申请新增广告主，通过 `list_requested_advertisers` 查看待处理申请。

---

## 六、访问权限与安全说明

| 项目 | 说明 |
|------|------|
| 鉴权方式 | 无鉴权（公开 MCP 端点） |
| 服务归属 | Vercel 托管（blind-spot-demo） |
| Token 类型 | 无需 Token |
| 访问范围 | 公开（仅供内部开发测试使用） |
| 数据权限 | 只读（run_query 工具仅支持 SELECT 查询） |
| 敏感数据 | 不含用户 PII，所有数据为广告主公开竞品情报 |

**注意**: 本 MCP 为公开测试版本，无鉴权保护。生产环境请使用 AdIntel-HF MCP（通过 Coolify 部署，需 Bearer Token 鉴权）。

---

## 七、当前运行状态

截至文档创建日期（2026 年 4 月 28 日），服务状态如下：

| 检查项 | 状态 | 备注 |
|--------|------|------|
| 服务健康 | 正常 | Vercel Serverless 正常运行 |
| MCP 端点响应 | 正常 | 11 个工具可正常调用 |
| 数据源连接 | 正常 | 连接到 AdIntel 数据库 |

---

## 八、后续计划

- [ ] 添加 Bearer Token 鉴权，支持生产环境部署
- [ ] 扩展工具覆盖范围（对齐 AdIntel MCP 的 22 个工具）
- [ ] 迁移至 Coolify 私有部署（替代 Vercel）
- [ ] 增加 GEO 生成引擎优化分析功能
- [ ] 增加用户评论情感分析功能

---

本文档为 FeedMob 内部存档，机密，请勿对外分享。

机密 · 仅供内部存档使用