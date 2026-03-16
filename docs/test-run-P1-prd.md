# PRD: ClawToolkit — AI Agent 工具市场

> Version: 1.2 (经 2 轮自审)
> Date: 2026-03-15
> Status: Pending Approval
> Based on: P0 Brainstorm Report

---

## 1. 背景与目标

### 问题陈述

AI Agent 需要调用外部工具完成任务，但当前工具生态存在四大痛点：
1. **发现碎片化** — 工具散落在 GitHub、npm、MCP.so 等数十个目录，没有统一商业化市场
2. **接入成本高** — 每个工具有不同的认证方式、调用协议、数据格式
3. **无统一计费** — Agent 调用 10 个工具需要 10 个账号和付费渠道
4. **质量不透明** — 无标准化的可靠性评分、延迟基准和安全审计

### 产品愿景

**ClawToolkit 是 AI Agent 的工具市场**。开发者发布工具，Agent 通过统一 API Key 发现、调用、付费——一个入口使用所有工具。

### 成功标准

见 Section 9 量化指标。

---

## 2. 目标用户画像

### Persona 1: Alex — AI Agent 开发者

- **角色**: 全栈开发者，正在用 LangChain 构建自动化 Agent
- **目标**: 快速找到并接入高质量工具，让 Agent 完成复杂任务
- **痛点**: 每个工具都要单独注册、认证、计费；不确定工具质量和可靠性
- **技术水平**: 高级

### Persona 2: Sam — 工具发布者

- **角色**: 后端开发者，有一个好用的 API（如实时汇率查询）想变现
- **目标**: 用最少的工作量将 API 发布到市场并赚取收入
- **痛点**: 自己搭建计费系统太重；没有分发渠道；用户获取困难
- **技术水平**: 中高级

### Persona 3: Jordan — 企业 AI 负责人

- **角色**: 某公司 AI 团队负责人，管理 20+ Agent 部署
- **目标**: 统一管理所有 Agent 使用的工具，可控的预算和用量
- **痛点**: 多个 Agent 使用不同工具，账单分散，无法统一监控用量和成本
- **技术水平**: 中级（偏管理）

---

## 3. 核心功能

### P0 — Must Have (MVP)

| # | 功能 | 描述 | User Story |
|---|------|------|-----------|
| 1 | 工具浏览与搜索 | 按分类/标签/关键词浏览和搜索工具，含详情页 | US-001 |
| 2 | 工具发布 | 开发者通过 Web UI 或 CLI 发布工具（支持 MCP Server / REST API / 自定义） | US-002 |
| 3 | 统一 API Gateway | 一个 API Key 调用所有已订阅的工具，平台代理转发 | US-003 |
| 4 | 计量与计费 | 按次计量工具调用，CP 积分扣费，Stripe 法币充值 | US-004 |
| 5 | 开发者 Dashboard | 调用统计、用量趋势、余额、API Key 管理 | US-005 |
| 6 | 发布者 Dashboard | 工具调用量、收入统计、提现 | US-006 |
| 7 | 用户认证 | 注册/登录（邮箱 + GitHub OAuth），API Key 生成 | US-007 |

### P1 — Should Have (v1.1)

| # | 功能 | 描述 |
|---|------|------|
| 8 | 工具评分与评论 | 用户对工具打分、留评，影响排序 |
| 9 | CLAW Token 支付 | CLAW 支付享 9 折，钱包连接 |
| 10 | 工具版本管理 | 发布者管理工具的多个版本，用户可锁定版本 |
| 11 | 工具组合 (Bundles) | 多个工具打包为一个组合，统一定价 |
| 12 | Webhook 通知 | 调用失败、余额不足、工具更新等事件通知 |

### P2 — Nice to Have (Future)

| # | 功能 | 描述 |
|---|------|------|
| 13 | 企业团队版 | 团队成员管理、用量配额、审计日志 |
| 14 | 工具自动测试 | 平台定期测试工具可用性，展示 uptime |
| 15 | Agent Framework SDK | LangChain/CrewAI/AutoGen 原生集成 |
| 16 | 链上存证 | 工具注册哈希锚定 ClawNetwork |
| 17 | 发布者提现 (CLAW) | 链上 CLAW 提现通道 |

---

## 4. User Stories & 验收标准

### US-001: 工具浏览与搜索

**As** Alex (Agent 开发者), **I want to** 在市场中搜索和浏览工具, **so that** 我能快速找到 Agent 需要的能力。

**验收标准:**
- Given 用户访问首页, When 页面加载, Then 展示推荐工具和分类导航
- Given 用户输入搜索关键词, When 提交搜索, Then 在 <500ms 内返回相关工具列表
- Given 搜索结果列表, When 点击工具卡片, Then 跳转到工具详情页，含描述/定价/调用示例/评分
- Given 工具详情页, When 查看调用示例, Then 展示 curl/Python/JavaScript 代码示例（可复制）

### US-002: 工具发布

**As** Sam (工具发布者), **I want to** 将我的 API 发布到市场, **so that** AI Agent 可以发现并调用它，我获得收入。

**验收标准:**
- Given 已登录的发布者, When 点击"发布工具", Then 展示发布表单（名称/描述/分类/定价/API 端点/认证方式）
- Given 填写完发布表单, When 提交, Then 工具进入审核状态，发布者收到确认
- Given MCP Server URL, When 输入 URL, Then 平台自动解析 MCP Schema 生成工具描述和参数定义
- Given 工具已发布, When 其他用户搜索, Then 该工具出现在搜索结果中

### US-003: 统一 API Gateway 调用

**As** Alex, **I want to** 用一个 API Key 调用任何已订阅的工具, **so that** 我的 Agent 不需要管理多个认证凭据。

**验收标准:**
- Given 用户已生成 API Key, When 调用 `POST /api/v1/tools/{tool_id}/invoke`, Then 平台代理请求到实际工具并返回结果
- Given 调用成功, When 查看 Dashboard, Then 该次调用被计量并扣除相应 CP
- Given API Key 无效或过期, When 调用任意工具, Then 返回 401 错误及明确提示
- Given 工具端不可用, When 调用, Then 返回 503 并在 <3s 内超时，不无限挂起
- Given 调用, When 计时, Then 平台代理增加的延迟 < 100ms (p95)

### US-004: 计量与计费

**As** Alex, **I want to** 通过 CP 积分支付工具调用费用, **so that** 我可以统一管理所有工具的支出。

**验收标准:**
- Given 用户 CP 余额 > 工具单次价格, When 调用工具, Then CP 被扣除，调用成功
- Given 用户 CP 余额 = 0, When 调用工具, Then 返回 402 错误，提示充值
- Given 用户在充值页, When 选择金额并支付 (Stripe), Then CP 即时到账
- Given 每月免费额度 (100 次/工具), When 未超额, Then 不扣 CP

### US-005: 开发者 Dashboard

**As** Alex, **I want to** 查看我的工具使用情况和花费, **so that** 我能控制预算并优化 Agent 的工具选择。

**验收标准:**
- Given 已登录用户, When 访问 Dashboard, Then 展示: 总调用次数/本月花费/CP 余额/API Key 列表
- Given Dashboard, When 查看用量趋势, Then 展示按天/周/月的调用量折线图
- Given Dashboard, When 查看工具明细, Then 展示每个工具的调用次数和花费
- Given API Key 管理, When 点击"生成新 Key", Then 生成 Key 并只展示一次

### US-006: 发布者 Dashboard

**As** Sam, **I want to** 查看我发布的工具被调用的情况和收入, **so that** 我能跟踪变现效果。

**验收标准:**
- Given 已登录的发布者, When 访问发布者 Dashboard, Then 展示: 总调用量/总收入/待结算金额
- Given 发布者, When 查看工具列表, Then 每个工具展示调用量/收入/评分
- Given 待结算金额 > 最低提现额, When 点击提现, Then 发起 Stripe Connect 提现

### US-007: 用户认证

**As** 新用户, **I want to** 快速注册并开始使用, **so that** 我不被复杂的注册流程阻挡。

**验收标准:**
- Given 新用户, When 点击"GitHub 登录", Then 通过 OAuth 完成注册，自动创建账号
- Given 新用户, When 选择邮箱注册, Then 填写邮箱+密码完成注册
- Given 注册完成, When 首次登录, Then 自动赠送 1000 CP (约 $1 等值)，足够体验
- Given 已登录, When 访问设置页, Then 可管理个人信息、API Key、支付方式

---

## 5. 非功能需求

### 性能
- 页面加载: < 2s (3G 网络)
- API Gateway 代理延迟: < 100ms (p95)
- 搜索响应: < 500ms
- 工具列表页: < 1s

### 安全
- API Key 使用 SHA-256 哈希存储（不存明文）
- 所有 API 端点 HTTPS only
- 工具调用代理不记录请求/响应 body（隐私保护）
- Rate limiting: 认证端点 10 req/min, API Gateway 1000 req/min (可配)
- CSRF + XSS + SQL Injection 防护

### 可访问性
- WCAG AA 合规
- 键盘导航支持
- 响应式 320px-2560px

### 浏览器支持
- Chrome, Firefox, Safari, Edge (最新 2 版本)
- Mobile: iOS Safari, Chrome Android

---

## 6. 数据模型概要

### 核心实体

| 实体 | 描述 | 关键字段 |
|------|------|---------|
| User | 注册用户（开发者/发布者） | id, email, name, github_id, role |
| Tool | 发布的工具 | id, publisher_id, name, slug, description, category, pricing, endpoint_url, protocol, status |
| ApiKey | 用户的 API Key | id, user_id, key_hash, name, last_used_at, is_active |
| Invocation | 工具调用记录 | id, user_id, tool_id, api_key_id, status, latency_ms, cp_cost, created_at |
| PointAccount | CP 积分账户 | 复用 shared_point_accounts |
| PointLedger | CP 交易记录 | 复用 shared_point_ledger |
| ToolCategory | 工具分类 | id, name, slug, icon |

### 关系
- User 1:N Tool (一个发布者可发布多个工具)
- User 1:N ApiKey (一个用户可有多个 Key)
- User 1:N Invocation (调用记录)
- Tool 1:N Invocation
- Tool N:1 ToolCategory

---

## 7. 页面 / 路由清单

| 页面 | 路由 | 用途 | 关键组件 |
|------|------|------|---------|
| 首页 | `/` | 工具市场入口，推荐/分类导航 | Hero, CategoryGrid, FeaturedTools, SearchBar |
| 搜索结果 | `/search?q=xxx` | 工具搜索结果 | SearchBar, ToolCard list, Filters, Pagination |
| 工具详情 | `/tools/[slug]` | 工具信息、定价、文档、代码示例 | ToolHeader, PricingCard, CodeSample, Reviews |
| 发布工具 | `/publish` | 发布新工具表单 | PublishForm, MCP Import, Preview |
| 开发者 Dashboard | `/dashboard` | 调用统计、余额、API Key | UsageChart, CostBreakdown, ApiKeyManager |
| 发布者 Dashboard | `/dashboard/publisher` | 工具管理、收入 | ToolList, RevenueChart, WithdrawButton |
| 充值 | `/dashboard/topup` | CP 积分充值 | AmountSelector, StripePayment |
| 登录 | `/auth/login` | 登录页 | GitHubOAuth, EmailForm |
| 注册 | `/auth/signup` | 注册页 | SignupForm |
| 设置 | `/settings` | 个人信息、API Key | ProfileForm, ApiKeyList |
| API 文档 | `/docs` | API 参考文档 | MarkdownRenderer, CodeSamples |

---

## 8. MVP 范围

### In Scope
- 工具浏览、搜索、详情展示
- 工具发布（Web UI, 支持 MCP / REST API）
- 统一 API Gateway（代理调用，一个 Key 调所有工具）
- CP 积分计量计费 + Stripe 法币充值
- 开发者 Dashboard（用量/花费/API Key）
- 发布者 Dashboard（调用量/收入/提现）
- 用户认证（GitHub OAuth + 邮箱）
- 每工具每月 100 次免费调用

### Out of Scope (MVP 不做)
- CLAW Token 支付 (P1)
- 工具评分与评论系统 (P1)
- 工具版本管理 (P1)
- 工具组合/Bundle (P1)
- 企业团队版 (P2)
- Agent Framework SDK 集成 (P2)
- 链上存证 (P2)
- 管理后台/运营审核界面（初期手动数据库）
- 移动端 App

---

## 9. 成功指标

| 指标 | 目标 (上线 3 个月) | 度量方式 |
|------|-------------------|---------|
| 上架工具数 | ≥ 50 | DB count |
| 注册用户数 | ≥ 500 | DB count |
| 月活跃调用次数 | ≥ 10,000 | Invocation table |
| 付费转化率 | ≥ 3% | Paid users / Total users |
| API Gateway p95 延迟 | < 100ms | 监控 |
| 工具发布到可调用时间 | < 10 分钟 | 用户流程测试 |
| 发布者满意度 | ≥ 4/5 | 调研 |

---

## 10. 自审记录

### Round 1: 产品经理视角

| # | 严重度 | 发现 | 修改 |
|---|--------|------|------|
| 1 | HIGH | 缺少免费额度定义 — 用户不知道白嫖能用多少 | 添加: 每工具每月 100 次免费调用 |
| 2 | HIGH | MCP 导入功能描述不具体，验收标准模糊 | US-002 增加: "Given MCP Server URL, When 输入 URL, Then 自动解析 Schema 生成工具描述" |
| 3 | MEDIUM | 工具审核流程未定义 — 谁审核？自动还是人工？ | MVP 阶段标注为"人工审核"，后续自动化 |
| 4 | MEDIUM | 未定义工具下架/违规处理流程 | MVP 不做自动化，手动数据库操作。P1 加管理后台 |
| 5 | LOW | 新用户注册后没有引导 — 可能不知道下一步做什么 | 添加: 注册赠送 1000 CP + 引导提示 |

### Round 2: 用户 (Alex — Agent 开发者) 视角

| # | 严重度 | 发现 | 修改 |
|---|--------|------|------|
| 1 | HIGH | 我最关心的是调用示例 — 详情页必须有可直接复制的 curl/Python/JS 代码 | US-001 验收标准增加: 代码示例展示（可复制） |
| 2 | MEDIUM | API Gateway 超时策略不明确 — 工具如果挂了我的 Agent 会卡住吗？ | US-003 增加: "Given 工具端不可用, Then 返回 503 并在 <3s 内超时" |
| 3 | MEDIUM | 搜索能不能按可靠性/延迟排序？我不想用垃圾工具 | 评分系统是 P1，MVP 先按调用量排序作为质量信号 |
| 4 | LOW | 我更习惯通过 npm/pip 装工具而不是 Web UI — 有 CLI 吗？ | CLI 归入 P1，MVP 先做 Web UI + API |
| 5 | LOW | 月 100 次免费够我调试但不够生产用 — 定价示例？ | 添加: 在工具详情页展示定价区间 ($0.001-$0.10/次) |
