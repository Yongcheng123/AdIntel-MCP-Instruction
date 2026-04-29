# AdIntel MCP 发布说明

FeedMob 内部存档文档

---

## 一、基本发布信息

| 字段 | 详细信息 |
|------|----------|
| MCP 名称 | AdIntel MCP (AdIntel-HF) |
| 版本 | v1.0（初始发布） |
| 发布日期 | 2026 年 4 月 24 日 |
| 发布平台 | Coolify（通过 npx mcp-remote 部署） |
| 发布账号 | HuggingFace Space (yongchengmu) |
| 发布人 | Yongcheng |
| 所属公司 | FeedMob |
| 适用部门 | 数据分析团队 |
| 文档创建日期 | 2026 年 4 月 29 日 |

---

## 二、项目背景与目的

### 2.1 背景

AdIntel MCP 是 FeedMob 内部自研的 Model Context Protocol 服务，基于 HuggingFace Space 部署，旨在将广告主竞争情报数据库通过标准化 MCP 接口暴露给 AI 助手（如 Claude），实现自然语言驱动的数据查询与竞品分析。

该服务整合了 SensorTower、SocialPeta、AppFollow、Otterly.AI 等多个数据平台，提供全链路的竞品情报分析能力。

### 2.2 核心目的

- 通过 MCP 协议将 SensorTower、SocialPeta、AppFollow、Otterly.AI 等多平台数据统一接入 AI 工作流
- 使分析师能够以对话方式完成广告主竞品比较、GEO 能见度分析、用户评论情感分析等复杂任务
- 降低数据查询门槛，无需手写 SQL 即可完成大多数分析场景

---

## 三、技术架构

### 3.1 部署环境

- **部署平台**: HuggingFace Space (Serverless) + Coolify（npx mcp-remote 代理）
- **账号类型**: HuggingFace 托管服务（yongchengmu）
- **访问方式**: HTTP-only MCP 端点
- **MCP 端点**: `https://yongchengmu-adintel-mcp.hf.space`
- **连接方式**: `npx mcp-remote --transport http-only https://yongchengmu-adintel-mcp.hf.space`

### 3.2 数据源集成

| 数据平台 | 数据类型 | 主要用途 |
|----------|----------|----------|
| SensorTower | 应用下载量、营收、市场排名 | 广告主规模评估与趋势分析 |
| SocialPeta | 展示广告创意、渠道分布 | 广告策略与投放渠道竞品分析 |
| AppFollow | App Store 用户评论与评分 | 用户情感分析与口碑监控 |
| Otterly.AI (GEO) | AI 搜索引擎能见度与引用率 | 生成引擎优化（GEO）竞品分析 |

### 3.3 数据库

底层使用关系型数据库（AdIntel DB），包含 14 张核心数据表，涵盖广告主信息、各平台采集数据、任务状态及告警记录。当前跟踪广告主数量：24 个，涵盖金融科技、手机游戏、生活方式三大垂直领域。

---

## 四、MCP 工具清单

本 MCP 共提供 **23 个**标准工具，按功能分为以下六大类：

### 4.1 广告主数据查询（4 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `list_advertisers` | 列出 AdIntel 中当前存储的广告主及数据新鲜度元数据 |
| `get_advertiser_summary` | 获取指定广告主的 SensorTower 综合摘要，包括下载量、营收等核心指标 |
| `get_metric_timeseries` | 获取指定广告主某指标的每日时间序列数据 |
| `read_schema_text` | 以文本形式返回规范 SQL Schema，供客户端使用 |

### 4.2 竞品比较分析（3 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `get_full_comparison` | 对两个或以上广告主进行完整的竞争对手数据比较（跨 SensorTower、SocialPeta、AppFollow） |
| `compare_geo_visibility` | 并排比较两个或以上品牌的 GEO 能见度 |
| `compare_appfollow_reviews` | 比较多个广告主的 AppFollow 评论情感与评分 |

### 4.3 GEO 生成引擎优化分析（3 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `get_geo_summary` | 基于 Otterly.AI 的 GEO 单品牌综合分析，包括 ChatGPT、Perplexity、Gemini 等引擎的能见度 |
| `get_geo_data_availability` | 检查数据库中 GEO 数据的可用性情况 |
| `compare_geo_visibility` | 并排比较两个或以上品牌的 GEO 能见度 |

### 4.4 用户评论情感分析（4 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `get_appfollow_reviews` | 列出广告主的 AppFollow 用户评论，支持按国家、OS、情感、评分筛选 |
| `get_appfollow_sentiment_trend` | 返回每日情感分布趋势（正面/负面/中性）及平均评分 |
| `get_appfollow_keyword_analysis` | 从 AppFollow 评论标签中提取热门关键词与主题 |
| `compare_appfollow_reviews` | 比较多个广告主的 AppFollow 评论情感与评分 |

### 4.5 广告创意分析（2 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `get_socialpeta_summary` | 获取广告主的 SocialPeta 展示广告摘要，含创意组合、渠道分布与标签 |
| `get_socialpeta_comparison` | 使用 SocialPeta 展示广告数据比较多个广告主 |

### 4.6 数据采集管理与运维（7 个工具）

| 工具名称 | 功能描述 |
|----------|----------|
| `get_collection_status` | 检查数据采集健康状态、活跃告警、近期运行记录及各平台数据可用性 |
| `get_job_status` | 查看单个任务状态，包含关联抓取运行的指标进度 |
| `list_jobs` | 列出近期任务，可按广告主或状态过滤 |
| `request_advertiser` | 记录缺失广告主的纳入申请 |
| `request_advertiser_refresh` | 触发指定广告主的按需抓取 |
| `list_requested_advertisers` | 列出已申请但尚未纳入的广告主 |
| `run_query` | 对 AdIntel 数据库执行只读 SQL 查询 |

---

## 五、功能模块说明

### 5.1 广告主数据查询

通过 `list_advertisers`、`get_advertiser_summary` 等工具，支持列举所有已追踪广告主、查询指定广告主的 SensorTower 综合指标摘要（下载量、营收、市场排名等），并可通过 `get_metric_timeseries` 获取任意指标的每日时序数据。

### 5.2 竞品比较分析

`get_full_comparison` 支持两个或多个广告主的全维度竞品比较（跨 SensorTower、SocialPeta、AppFollow 三个平台）。支持同类产品横向比较，如 MonopolyGo vs. ScrabbleGo，或金融类产品 Chime vs. Current 等场景。

### 5.3 GEO 生成引擎优化分析

基于 Otterly.AI 数据，提供品牌在 ChatGPT、Perplexity、Gemini 等 AI 搜索引擎中的能见度与引用率分析。支持单品牌深度分析（`get_geo_summary`）与多品牌对比（`compare_geo_visibility`），可用于评估品牌在 AI 时代的搜索曝光策略。

### 5.4 用户评论情感分析

通过 AppFollow 数据，支持查询指定广告主的用户评论列表、每日情感趋势（正面/负面/中性占比与平均评分），以及从评论标签中提取热门关键词。支持跨品牌情感对比，辅助产品口碑监控。

### 5.5 广告创意分析

通过 SocialPeta 数据，支持查询广告主的展示广告创意组合、投放渠道分布及标签分析。支持跨广告主比较，用于竞品广告策略研究。

### 5.6 数据采集管理

提供数据采集健康监控（`get_collection_status`）、任务状态查询（`get_job_status`、`list_jobs`）、新广告主申请（`request_advertiser`）及按需刷新（`request_advertiser_refresh`）等运维管理功能。

---

## 六、访问权限与安全说明

| 项目 | 说明 |
|------|------|
| 鉴权方式 | 无鉴权（公开 MCP 端点） |
| 服务归属 | HuggingFace Space 托管（yongchengmu） |
| Token 类型 | 无需 Token（公开测试版本） |
| 访问范围 | 公开（仅供内部开发测试使用） |
| 数据权限 | 只读（run_query 工具仅支持 SELECT 查询，禁止写操作） |
| 敏感数据 | 不含用户 PII，所有数据为广告主公开竞品情报 |

**注意**: 本 MCP 为公开测试版本，无鉴权保护。生产环境建议添加 Bearer Token 鉴权。

---

## 七、当前运行状态

截至文档创建日期（2026 年 4 月 29 日），数据采集状态如下：

| 数据平台 | 覆盖广告主数 | 数据新鲜度状态 | 备注 |
|----------|-------------|---------------|------|
| SensorTower | 24/24 | 需检查 | 采集管道可能存在停滞 |
| AppFollow | 24/24 | 需检查 | 近期有活跃抓取记录 |
| SocialPeta | 24/24 | 需检查 | 数据存量完整 |
| OtterlyAI (GEO) | 23/24 | 需检查 | Koho 缺少 GEO 数据 |

当前共有 156 个活跃告警（均为 critical 级别的 stale_data），建议相关负责人核查 SensorTower 采集管道状态。

---

## 八、后续计划

- [ ] 添加 Bearer Token 鉴权，支持生产环境部署
- [ ] 修复 SensorTower 及各平台的数据采集停滞问题
- [ ] 扩充跟踪广告主范围（目前 24 个，计划持续增加）
- [ ] 为 Koho 补充 GEO（OtterlyAI）数据采集配置
- [ ] 探索与 Femini MCP 的协同工作流，支持客户 Campaign 数据与竞品数据的联合分析

---

本文档为 FeedMob 内部存档，机密，请勿对外分享。

机密 · 仅供内部存档使用