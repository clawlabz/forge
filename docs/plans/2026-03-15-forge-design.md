# Forge — 从 Idea 到上线的全自动化产品锻造流水线

> **版本**: 1.0
> **日期**: 2026-03-15
> **状态**: 设计定稿
> **落地形式**: Claude Code Plugin（通用开源，兼容 Codex/OpenClaw）

---

## 1. 定位与核心理念

**Forge** 是一个全自动化的产品制造工作流引擎。

```
输入：一句话 idea
输出：线上可用的完整产品 + 项目文档 + 知识库
```

### 设计原则

| 原则 | 说明 |
|------|------|
| **Provider 可插拔** | 每个环节的工具/平台可替换，统一接口，默认实现 + 扩展预留 |
| **阶段可独立调用** | `/forge` 一键全流程，`/forge:prd` 单独跑某阶段 |
| **进度可中断恢复** | 全程状态文件驱动，session 断了从断点继续 |
| **审批门控** | PRD、架构、设计三个节点暂停，推送在线文档链接等人类审批 |
| **模型可路由** | 每个阶段可配置不同 LLM，平台不支持时 fallback 到当前模型 |
| **平台无关** | 当前 Claude Code plugin，设计上兼容 Codex/OpenClaw/Gemini CLI |

---

## 2. 流水线阶段总览

```
┌──────────────────────────────────────────────────────────────────┐
│                       /forge (orchestrator)                       │
├───────┬───────┬────────┬────────┬────────┬───────┬───────┬──────┤
│  P0   │  P1   │   P2   │   P3   │   P4   │  P5   │  P6   │  P7  │
│Brain  │ PRD   │ Archi  │Design  │ Dev    │ Test  │Verify │Deploy│
│storm  │       │ tect   │        │        │       │       │      │
├───────┴───────┴────────┴────────┴────────┴───────┴───────┴──────┤
│                                                                  │
│  P0 ──→ init ──→ P1 ──🔴──→ ┌ P2 ──🔴 ┐                        │
│                              │  并行    │──→ P4 ──→ P5           │
│                              └ P3 ──🔴 ┘                        │
│                                                                  │
│  P5 ──→ P6(×3轮) ──→ P7 ──→ ✅ 交付                              │
│                                                                  │
│  🔴 = 审批门控 (推送 GitHub 链接，等待人类审批)                      │
└──────────────────────────────────────────────────────────────────┘
```

| 阶段 | Skill 命令 | 输入 | 输出 | 审批 |
|------|-----------|------|------|------|
| **P0 Brainstorm** | `/forge:brainstorm` | idea 描述 | 调研报告、可行性分析、商业闭环分析 | 否 |
| **init** | (orchestrator 内置) | P0 GO 结论 | Git repo + 目录结构 + forge.config.yaml | 否 |
| **P1 PRD** | `/forge:prd` | P0 产出 | PRD 文档（2轮自审后定稿） | **是** |
| **P2 Architect** | `/forge:architect` | PRD | 技术架构 + 开发 Plan（自审后定稿） | **是** |
| **P3 Design** | `/forge:design` | PRD | 设计规范 / 设计稿 | **是** |
| **P4 Develop** | `/forge:develop` | 架构 + 设计 + Plan | 完整代码，多 Agent 并行 | 否 |
| **P5 Test** | `/forge:test` | 代码 | 自动化测试报告 + 浏览器截图 | 否 |
| **P6 Verify** | `/forge:verify` | 全部产出 | 3轮复验报告，多 Agent 并行修复 | 否 |
| **P7 Deploy** | `/forge:deploy` | 验收通过的代码 | 线上产品 + 项目说明 + 知识库 | 否 |

**P2 与 P3 并行执行**：两者输入都是 PRD，互不依赖。各自审批通过后才进 P4。

---

## 3. 各阶段详细设计

### P0 Brainstorm — `/forge:brainstorm`

**目标：** 把一句话 idea 变成经过验证的产品概念。

**执行流程：**
```
idea输入 → 市场调研 → 竞品分析 → 可行性评估 → 商业闭环分析 → 产出报告
```

**调用的 Skills / Agents / Tools：**
- `agent-reach` / `WebSearch` — 市场调研、竞品搜索
- `brainstorming` skill — 结构化头脑风暴
- `everything-claude-code:market-research` — 竞品/市场深度调研
- `everything-claude-code:search-first` — 搜索现有方案避免重复造轮子
- Architect agent — 技术可行性初判

**产出：** `docs/forge/P0-brainstorm-report.md`

```markdown
# P0 Brainstorm Report: [产品名]
## 1. 产品概念（一句话定义 + 详细描述）
## 2. 目标用户与使用场景
## 3. 市场调研（市场规模、趋势、时机）
## 4. 竞品分析（3-5个竞品对比矩阵）
## 5. 现有开源方案调研（可复用的项目/库）
## 6. 技术可行性（初步判断，标注风险点）
## 7. 商业闭环（收入模型、成本结构、盈利路径）
## 8. 结论：GO / PIVOT / KILL（附理由）
```

**关键逻辑：**
- **KILL** → 停止流水线，告知用户原因
- **PIVOT** → 展示修改建议，用户确认后以修改版继续
- **GO** → 自动创建 Git repo + 初始化目录结构 + 进 P1

---

### P1 PRD — `/forge:prd`

**目标：** 输出专业级产品需求文档，经过 2 轮自审后定稿。

**执行流程：**
```
P0报告 → 撰写PRD v1 → 自审Round1 → 修改 → 自审Round2 → 定稿 → 推送GitHub → 请求审批
```

**调用的 Skills / Agents：**
- `writing-plans` skill — 结构化文档撰写
- `everything-claude-code:blueprint` — 需求分解

**自审维度：**

| 轮次 | 角色 | 审查重点 |
|------|------|---------|
| Round 1 | 资深产品经理 | 需求完整性、优先级合理性、边界条件、用户故事覆盖 |
| Round 2 | 目标用户 | 易用性、功能是否真正解决痛点、是否过度设计（YAGNI） |

**产出：** `docs/forge/P1-prd.md`

```markdown
# PRD: [产品名]
## 1. 背景与目标
## 2. 目标用户画像
## 3. 核心功能列表（P0/P1/P2 优先级）
## 4. 用户故事与验收标准（Given/When/Then）
## 5. 非功能需求（性能、安全、可访问性）
## 6. 数据模型概要
## 7. 页面/路由清单
## 8. MVP 范围界定（明确做什么、不做什么）
## 9. 成功指标（可量化 KPI）
## 10. 自审记录（2轮修改日志）
```

**🔴 审批门控：** 推送到 GitHub → 输出：
```
请审批 PRD: https://github.com/xxx/blob/main/docs/forge/P1-prd.md
```

---

### P2 Architect — `/forge:architect`

**目标：** 顶级架构师视角产出技术方案 + 可 checklist 的开发计划。

**与 P3 并行执行。**

**执行流程：**
```
PRD → 技术选型 → 架构设计 → 开发Plan → 自审Round1(安全) → 自审Round2(性能) → 定稿 → 请求审批
```

**调用的 Skills / Agents：**
- Architect agent — 系统设计
- Security-reviewer agent — 安全审计
- `supabase-postgres-best-practices` — 数据库设计
- `vercel-react-best-practices` — 前端架构
- `everything-claude-code:api-design` — API 设计
- `everything-claude-code:backend-patterns` — 后端模式
- `everything-claude-code:database-migrations` — 数据库迁移策略
- `security-review` skill — 安全审查

**自审维度：**

| 轮次 | 视角 | 重点 |
|------|------|------|
| Round 1 | 安全工程师 | 认证方案、数据保护、注入防御、OWASP Top 10 |
| Round 2 | 性能工程师 | 瓶颈分析、缓存策略、数据库查询优化、CDN |

**产出：**

`docs/forge/P2-architecture.md`
```markdown
# 技术架构: [产品名]
## 1. 技术栈选型（附决策理由）
## 2. 系统架构图（Mermaid）
## 3. 项目目录结构
## 4. 数据库 Schema（表设计 + 关系 + 索引策略）
## 5. API 设计（路由清单 + 请求/响应格式）
## 6. 认证/授权方案
## 7. 第三方服务集成
## 8. 性能策略（缓存、CDN、懒加载、SSR/SSG）
## 9. 安全策略
## 10. 自审记录（2轮修改日志）
```

`docs/forge/P2-dev-plan.md`
```markdown
# 开发计划: [产品名]
## Phase 划分
### Phase 1: 基础框架
- [ ] Task 1.1: 项目初始化 + 依赖安装
- [ ] Task 1.2: 数据库建表
- [ ] ...
### Phase 2: 核心功能
- [ ] Task 2.1: ...
### Phase N: 收尾优化
- [ ] ...
## 依赖关系图（哪些 task 可并行）
## 风险点与应对方案
```

**🔴 审批门控：** 推送到 GitHub → 输出：
```
请审批技术架构: https://github.com/xxx/blob/main/docs/forge/P2-architecture.md
请审批开发计划: https://github.com/xxx/blob/main/docs/forge/P2-dev-plan.md
```

---

### P3 Design — `/forge:design`

**目标：** 产品视觉设计规范 + 页面设计方案。

**与 P2 并行执行。**

**执行流程：**
```
PRD → 设计风格探索 → 设计系统定义 → 页面布局设计 → 自审 → 定稿 → 请求审批
```

**调用的 Skills / Agents：**
- `ui-ux-pro-max` — 50+风格/21色板/50+字体组合的设计智能
- `frontend-design` — 高质量 UI 组件设计
- `everything-claude-code:frontend-patterns` — 前端设计模式
- Design Provider（默认 `manual_spec`，可配置 Figma MCP / Stitch）

**产出：** `docs/forge/P3-design-spec.md`

```markdown
# 设计规范: [产品名]
## 1. 设计理念与风格方向
## 2. 色彩系统（primary, secondary, accent, semantic colors, dark mode）
## 3. 字体系统（font family, type scale, weights）
## 4. 间距与栅格系统（spacing scale, breakpoints）
## 5. 核心组件规范（Button, Card, Input, Modal, Table, Navigation...）
## 6. 页面设计方案
###   6.1 首页 — 布局 + 关键元素 + 交互说明
###   6.2 [功能页A] — ...
###   6.N ...
## 7. 响应式策略（mobile-first breakpoints）
## 8. 动效规范（transition, hover, loading states）
## 9. 暗色模式方案
## 10. 无障碍设计（a11y checklist）
```

**当 Design Provider 为 Figma 时：** 自动生成 Figma 文件链接作为附加产出。

**🔴 审批门控：** 推送到 GitHub → 输出：
```
请审批设计方案: https://github.com/xxx/blob/main/docs/forge/P3-design-spec.md
```

---

### P4 Develop — `/forge:develop`

**目标：** 按 Plan 多 Agent 并行开发，全程进度追踪。

**前置条件：** P2 + P3 审批全部通过。

**执行流程：**
```
读取 P2-dev-plan.md
→ 按 Phase 逐步执行
  → 每个 Phase 内独立 task 多 Agent 并行开发
  → Phase 完成后自动 checkpoint (build + lint + 基础测试)
→ 阶段性进度更新到 .forge/state.yaml
→ 全部 Phase 完成 → 进 P5
```

**开发模式选择逻辑：**
```
复杂业务逻辑（金融、状态机、权限）→ TDD（先写测试再实现）
展示型项目（官网、Dashboard、Landing）→ 常规开发 + 后补测试
混合项目 → 核心模块 TDD + UI 层常规开发
```

**调用的 Skills / Agents：**
- `tdd-guide` agent — TDD 驱动开发
- `code-reviewer` agent — 每个 Phase 结束后 code review
- `build-error-resolver` agent — 构建失败自动修复
- `vercel-react-best-practices` — React/Next.js 最佳实践
- `fullstack-guardian` — 全栈安全开发
- `everything-claude-code:coding-standards` — 代码规范
- `everything-claude-code:tdd-workflow` — TDD 流程
- `dispatching-parallel-agents` — 多 Agent 并行调度
- `security-review` — 安全审查

**阶段性 Checkpoint（每个 Phase 结束后）：**
```
build → lint → 基础单元测试 → 通过才进下一 Phase
失败 → build-error-resolver 自动修复 → 重试
```

**进度追踪：** `.forge/state.yaml` 实时更新每个 task 状态。

---

### P5 Test — `/forge:test`

**目标：** 系统性全面自动化测试。

**执行流程：**
```
单元测试 → 集成测试 → E2E浏览器测试(截图) → 测试报告 → 自动修复失败项 → 重测
```

**调用的 Skills / Agents：**
- `javascript-testing-patterns` — 单元/集成测试
- `e2e-testing-patterns` — E2E 测试模式
- `e2e-runner` agent — E2E 执行
- `browser-use` skill — 浏览器自动化 + 截图
- `webapp-testing` skill — Web 应用测试
- `everything-claude-code:e2e-testing` — E2E 测试
- `everything-claude-code:verification-loop` — 验证循环
- `build-error-resolver` agent — 测试失败修复

**产出：** `docs/forge/P5-test-report.md`

```markdown
# 测试报告: [产品名]
## 覆盖率: XX% (目标 ≥80%)
## 单元测试: X passed / Y failed / Z skipped
## 集成测试: X passed / Y failed
## E2E 测试: X passed / Y failed
## 关键页面浏览器截图
## 失败项修复记录（修复前→修复后）
## 性能基线（Lighthouse 分数）
```

---

### P6 Verify — `/forge:verify`

**目标：** 3 轮多维度复验，确保产品完成度达到最高。

**执行流程：**
```
Round 1: 多Agent并行审查 → 生成待办清单 → 多Agent并行修复 → 验证修复
Round 2: 同上（聚焦 Round 1 遗留 + 新发现的问题）
Round 3: 同上（最终确认，预期待办趋近于零）
```

**每轮审查的并行 Agent 维度：**

| Agent/Skill | 审查重点 |
|-------------|---------|
| Code Reviewer | 代码质量、命名、结构、死代码 |
| Security Reviewer | 安全漏洞、敏感数据泄露、OWASP |
| PRD 对照审查 | 功能完整性 vs PRD 需求逐项勾对 |
| UX 审查 (browser-use) | 浏览器实际操作体验、交互流畅度 |
| Performance 审查 | Lighthouse 评分、加载速度、bundle 大小 |
| SEO 审查 (seo-audit) | SEO 基础项检查 |
| `refactor-cleaner` | 死代码、重复代码清理 |

**产出：** `docs/forge/P6-verify-round-{1,2,3}.md`

```markdown
# 复验报告 Round X: [产品名]
## 发现问题清单
| # | 维度 | 严重度 | 描述 | 状态 |
|---|------|--------|------|------|
| 1 | Security | CRITICAL | ... | ✅ Fixed |
| 2 | UX | MEDIUM | ... | ✅ Fixed |
| 3 | ... | ... | ... | ... |
## 修复记录
## 遗留问题（转入下一轮）
## 本轮结论：PASS / NEEDS_WORK
```

---

### P7 Deploy — `/forge:deploy`

**目标：** 全自动部署上线 + 交付完整文档 + 知识库生成。

**执行流程：**
```
Git 提交推送
→ 数据库建表 (Supabase MCP / migrate script)
→ 部署 (Vercel MCP / deploy script)
→ DNS 解析 (Cloudflare MCP / dns script)
→ 健康检查 (访问线上 URL + 截图)
→ [可选] GA4 + Clarity 埋点
→ [可选] Smoke test (关键页面自动化验证)
→ 知识库生成
→ 项目总结文档
```

**调用的 Skills / Agents / MCP：**
- **Vercel MCP Server** (优先) / `scripts/vercel-deploy.sh` (fallback)
- **Supabase MCP Server** (优先) / `scripts/supabase-migrate.sh` (fallback)
- **Cloudflare MCP Server** (优先) / `scripts/cloudflare-dns.sh` (fallback)
- `devops` skill — 部署流程
- `everything-claude-code:deployment-patterns` — 部署模式
- `ga4-clarity-auto-integration` skill — GA4 + Clarity 埋点（可选）
- `browser-use` — 部署后 smoke test + 截图

**可选步骤：GA4 + Clarity 集成**

在 `forge.config.yaml` 中配置：
```yaml
optional:
  analytics:
    ga4:
      enabled: true           # 默认 true，可设为 false 跳过
      measurement_id: ""      # 留空则跳过
    clarity:
      enabled: true
      project_id: ""          # 留空则跳过
```

启用后自动：
- 注入 GA4 tracking script
- 注入 Clarity script
- 验证数据上报正常

**知识库生成：**

```
knowledge/
├── index.json           # 结构化索引（文件→功能映射）
├── architecture.md      # 架构速览（P2 精简版）
├── api-reference.md     # API 文档（自动从代码生成）
├── database-schema.md   # 数据库表说明
├── decisions.md         # 关键技术决策记录
└── onboarding.md        # 新人上手指南（5分钟了解项目）
```

**最终交付：** `docs/forge/P7-project-summary.md`

```markdown
# 项目交付报告: [产品名]

## 线上地址
- 产品: https://xxx.clawlabz.xyz
- 仓库: https://github.com/clawlabz/xxx

## 技术栈
- Frontend: ...
- Backend: ...
- Database: ...
- Deploy: ...

## 功能完成度
| PRD 需求 | 状态 | 说明 |
|---------|------|------|
| 功能A | ✅ | |
| 功能B | ✅ | |

## 数据库表清单
## API 路由清单
## 环境变量清单
## 测试覆盖率
## Lighthouse 评分
## 已知限制与后续优化建议
## 知识库入口: knowledge/
## 全部文档索引
```

---

## 4. 配置系统

### 4.1 主配置文件 — `forge.config.yaml`

```yaml
version: "1.0"

# ---- 项目信息 ----
project:
  name: ""                         # P0 后自动填充
  description: ""
  domain: ""                       # 如 xxx.clawlabz.xyz

# ---- 模型路由 ----
models:
  brainstorm: "opus"               # 深度推理
  prd: "opus"                      # 深度推理
  architect: "opus"                # 深度推理
  design: "sonnet"                 # 默认，可配 gemini-pro 等
  develop: "sonnet"                # 代码能力强
  test: "sonnet"
  verify: "opus"                   # 复验需要深度审计
  deploy: "haiku"                  # 执行性任务
  fallback: "current"             # 平台不支持时 fallback

# ---- Provider 配置 ----
providers:
  doc_hosting:
    type: "github"                 # github | notion | cloudflare_pages
    config:
      repo: ""                     # auto-detect or manual
      branch: "main"

  deployment:
    type: "vercel"                 # vercel | cloudflare_pages | netlify
    strategy: "mcp_first"          # mcp_first | script_only | mcp_only
    mcp: "vercel-mcp-server"
    fallback_script: "scripts/vercel-deploy.sh"
    config:
      team: ""

  database:
    type: "supabase"               # supabase | planetscale | neon
    strategy: "script_first"       # 迁移场景脚本更可靠
    mcp: "supabase-mcp-server"
    fallback_script: "scripts/supabase-migrate.sh"
    config:
      project_ref: ""

  dns:
    type: "cloudflare"             # cloudflare | vercel_dns | route53
    strategy: "mcp_first"
    mcp: "cloudflare-mcp-server"
    fallback_script: "scripts/cloudflare-dns.sh"
    config:
      zone: ""

  storage:
    type: "cloudflare_r2"          # cloudflare_r2 | s3 | supabase_storage
    config:
      bucket: ""

  design:
    type: "manual_spec"            # manual_spec | figma | stitch
    config: {}

  testing:
    type: "playwright"             # playwright | cypress
    config: {}

# ---- 可选功能 ----
optional:
  analytics:
    ga4:
      enabled: true
      measurement_id: ""           # 留空则跳过
    clarity:
      enabled: true
      project_id: ""               # 留空则跳过
  knowledge_base:
    enabled: true                  # 是否生成知识库

# ---- 审批配置 ----
approval:
  method: "github"                 # github | interactive | notion
  gates:
    - prd
    - architect
    - design

# ---- Preset ----
preset: null                       # "claw" | "generic" | 自定义路径
```

### 4.2 Secrets 配置 — `.forge/secrets.yaml`

**此文件加入 `.gitignore`，不提交到 Git。**

```yaml
# API Keys & Tokens（按需填写，留空则使用环境变量 fallback）
github_token: ""                   # 或 $GITHUB_TOKEN
vercel_token: ""                   # 或 $VERCEL_TOKEN
supabase_service_key: ""           # 或 $SUPABASE_SERVICE_ROLE_KEY
cloudflare_api_token: ""           # 或 $CLOUDFLARE_API_TOKEN

# 可选
figma_token: ""                    # Design Provider: Figma
notion_api_key: ""                 # Doc Hosting Provider: Notion
ga4_measurement_id: ""             # GA4 集成
clarity_project_id: ""             # Clarity 集成
```

**优先级：** `secrets.yaml` > 环境变量 > `project-info.md` (legacy)

### 4.3 Preset 机制

预设配置模板，快速初始化特定生态的默认值。

```bash
/forge init --preset claw          # ClawLabz 生态预设
/forge init --preset generic       # 通用预设（全留空，逐项配置）
/forge init                        # 交互式逐项配置
```

```yaml
# presets/claw.yaml
project:
  domain_suffix: "clawlabz.xyz"
providers:
  deployment:
    type: "vercel"
  database:
    type: "supabase"
    config:
      project_ref: "existing-claw-project"
  dns:
    type: "cloudflare"
    config:
      zone: "clawlabz.xyz"
  storage:
    type: "cloudflare_r2"
    config:
      bucket: "claw-market"
```

---

## 5. 状态管理与中断恢复

### 5.1 状态文件 — `.forge/state.yaml`

```yaml
version: "1.0"
project: "my-product"
started_at: "2026-03-15T10:00:00Z"
current_phase: "P2_architect"
status: "awaiting_approval"        # running | awaiting_approval | paused | completed | failed

phases:
  P0_brainstorm:
    status: "completed"
    started_at: "2026-03-15T10:00:00Z"
    completed_at: "2026-03-15T10:15:00Z"
    attempt: 1
    outputs:
      - "docs/forge/P0-brainstorm-report.md"

  P1_prd:
    status: "completed"
    attempt: 1
    review_rounds: 2
    approval:
      status: "approved"
      approved_at: "2026-03-15T11:00:00Z"
      doc_url: "https://github.com/clawlabz/xxx/blob/main/docs/forge/P1-prd.md"
    outputs:
      - "docs/forge/P1-prd.md"

  P2_architect:
    status: "awaiting_approval"
    attempt: 1
    review_rounds: 2
    approval:
      status: "pending"
      doc_url: "https://github.com/clawlabz/xxx/blob/main/docs/forge/P2-architecture.md"
    outputs:
      - "docs/forge/P2-architecture.md"
      - "docs/forge/P2-dev-plan.md"

  P3_design:
    status: "not_started"

  P4_develop:
    status: "not_started"
    sub_phases: []

  P5_test:
    status: "not_started"

  P6_verify:
    status: "not_started"
    rounds: []

  P7_deploy:
    status: "not_started"

# ---- 进度摘要 ----
summary: |
  产品: xxx
  当前: P2 架构设计已完成，等待审批中
  已完成: P0 调研(GO) + P1 PRD(已审批通过)
  下一步: P2 审批通过后，P3 设计并行启动；两者都审批通过后进 P4 开发
```

### 5.2 子阶段追踪（P4 开发阶段）

```yaml
P4_develop:
  status: "running"
  dev_mode: "tdd"                  # tdd | standard | hybrid
  sub_phases:
    - name: "Phase 1: 基础框架"
      status: "completed"
      checkpoint: { build: "pass", lint: "pass", test: "pass" }
    - name: "Phase 2: 核心功能"
      status: "running"
      progress: "3/7 tasks completed"
      tasks:
        - { name: "用户认证", status: "completed", agent: "agent-1" }
        - { name: "数据库 CRUD", status: "completed", agent: "agent-2" }
        - { name: "API 路由", status: "completed", agent: "agent-1" }
        - { name: "状态管理", status: "running", agent: "agent-3" }
        - { name: "表单验证", status: "not_started" }
        - { name: "错误处理", status: "not_started" }
        - { name: "Loading states", status: "not_started" }
    - name: "Phase 3: UI 完善"
      status: "not_started"
```

### 5.3 并行分支状态

```yaml
parallel_group: "arch_and_design"
branches:
  P2_architect:
    status: "approved"
  P3_design:
    status: "awaiting_approval"
gate: "all_approved"               # 全部通过才进 P4
```

### 5.4 审批重试

```yaml
P1_prd:
  attempt: 2
  history:
    - attempt: 1
      status: "rejected"
      reason: "缺少竞品分析，商业模式不够清晰"
      rejected_at: "2026-03-15T10:30:00Z"
  current:
    attempt: 2
    status: "awaiting_approval"
    doc_url: "https://github.com/..."
```

### 5.5 恢复机制

当新 session 启动 `/forge` 时：
1. 检测 `.forge/state.yaml` 是否存在
2. 读取当前阶段和状态
3. 输出摘要：`"上次进行到 P2 架构审批中，P3 设计尚未开始。是否继续？"`
4. 用户确认后从断点继续

单独调用 `/forge:resume` 也可触发恢复。

---

## 6. 项目目录结构

```
my-product/
├── forge.config.yaml              # Forge 配置（提交到 Git）
├── .forge/                        # Forge 运行时（部分 gitignore）
│   ├── state.yaml                 # 流水线状态
│   ├── secrets.yaml               # API keys（.gitignore）
│   └── logs/                      # 各阶段执行日志
│       ├── P0-brainstorm.log
│       ├── P1-prd.log
│       └── ...
├── docs/
│   └── forge/                     # 所有 Forge 产出文档（提交到 Git）
│       ├── P0-brainstorm-report.md
│       ├── P1-prd.md
│       ├── P2-architecture.md
│       ├── P2-dev-plan.md
│       ├── P3-design-spec.md
│       ├── P5-test-report.md
│       ├── P6-verify-round-1.md
│       ├── P6-verify-round-2.md
│       ├── P6-verify-round-3.md
│       ├── P7-deploy-report.md
│       └── P7-project-summary.md  # 最终交付文档
├── knowledge/                     # 知识库
│   ├── index.json
│   ├── architecture.md
│   ├── api-reference.md
│   ├── database-schema.md
│   ├── decisions.md
│   └── onboarding.md
├── src/                           # 产品源码
├── tests/                         # 测试代码
├── public/                        # 静态资源
└── package.json
```

---

## 7. Plugin 目录结构

```
forge/
├── README.md                      # Plugin 说明
├── package.json                   # Plugin 元数据
├── SKILL.md                       # Plugin 入口（注册所有 skills）
│
├── skills/                        # 各阶段 Skill 定义
│   ├── forge.md                   # Orchestrator 主入口 (/forge)
│   ├── brainstorm.md              # P0 (/forge:brainstorm)
│   ├── prd.md                     # P1 (/forge:prd)
│   ├── architect.md               # P2 (/forge:architect)
│   ├── design.md                  # P3 (/forge:design)
│   ├── develop.md                 # P4 (/forge:develop)
│   ├── test.md                    # P5 (/forge:test)
│   ├── verify.md                  # P6 (/forge:verify)
│   ├── deploy.md                  # P7 (/forge:deploy)
│   ├── resume.md                  # 中断恢复 (/forge:resume)
│   └── init.md                    # 初始化配置 (/forge:init)
│
├── agents/                        # 专用 Agent 定义
│   ├── prd-reviewer.md            # PRD 自审 agent
│   ├── arch-reviewer.md           # 架构自审 agent
│   ├── design-reviewer.md         # 设计自审 agent
│   └── verify-checker.md          # 复验审查 agent
│
├── presets/                       # 预设配置模板
│   ├── claw.yaml                  # ClawLabz 生态预设
│   └── generic.yaml               # 通用预设
│
├── templates/                     # 文档模板
│   ├── forge.config.yaml          # 配置文件模板
│   ├── prd-template.md
│   ├── architecture-template.md
│   ├── design-spec-template.md
│   ├── test-report-template.md
│   ├── verify-report-template.md
│   └── project-summary-template.md
│
└── providers/                     # Provider 接口说明（扩展指南）
    ├── README.md                  # Provider 开发指南
    ├── doc-hosting/
    │   └── github.md
    ├── deployment/
    │   └── vercel.md
    ├── database/
    │   └── supabase.md
    ├── dns/
    │   └── cloudflare.md
    ├── design/
    │   └── manual-spec.md
    └── analytics/
        ├── ga4.md
        └── clarity.md
```

---

## 8. Skill 增强矩阵（完整版）

各阶段调用的 Skills / Agents / MCP 完整清单：

| 阶段 | 核心 Skills & Agents | 补充 Skills (ECC / 三方) | MCP Servers |
|------|---------------------|------------------------|-------------|
| **P0** | brainstorming, agent-reach, architect agent | market-research, search-first | — |
| **P1** | writing-plans | blueprint | — |
| **P2** | architect agent, security-reviewer agent | api-design, backend-patterns, database-migrations, supabase-postgres-best-practices, vercel-react-best-practices, security-review | — |
| **P3** | ui-ux-pro-max, frontend-design | frontend-patterns | Figma MCP (可选) |
| **P4** | tdd-guide, code-reviewer, build-error-resolver | coding-standards, tdd-workflow, fullstack-guardian, vercel-react-best-practices, dispatching-parallel-agents | — |
| **P5** | e2e-runner, browser-use, webapp-testing | e2e-testing, javascript-testing-patterns, verification-loop | — |
| **P6** | code-reviewer, security-reviewer | security-scan, refactor-cleaner, seo-audit | — |
| **P7** | devops | deployment-patterns, ga4-clarity-auto-integration | Vercel MCP, Supabase MCP, Cloudflare MCP |

---

## 9. 所需 API Keys / 配置清单

### 必需（默认 Provider）

| 配置项 | 用途 | 当前状态 |
|-------|------|---------|
| `GITHUB_TOKEN` | Git 推送、文档托管 | ✅ 已有 |
| `VERCEL_TOKEN` | Vercel 部署 | ✅ 已有 |
| `SUPABASE_URL` | 数据库连接 | ✅ 已有 |
| `SUPABASE_SERVICE_ROLE_KEY` | 数据库管理 | ✅ 已有 |
| `CLOUDFLARE_API_TOKEN` | DNS 管理 | ✅ 已有 |
| `CLOUDFLARE_ZONE_ID` | DNS Zone | ✅ 已有 |

### 可选（按启用的 Provider / 功能）

| 配置项 | 用途 | 何时需要 |
|-------|------|---------|
| `FIGMA_TOKEN` | Figma 设计稿读取 | Design Provider 切 Figma 时 |
| `NOTION_API_KEY` | Notion 文档托管 | Doc Hosting 切 Notion 时 |
| `GA4_MEASUREMENT_ID` | Google Analytics 4 | 启用 GA4 埋点时 |
| `CLARITY_PROJECT_ID` | Microsoft Clarity | 启用 Clarity 时 |
| `STITCH_API_KEY` | Google Stitch 设计 | Design Provider 切 Stitch 时 |

**当前所有必需 key 已具备，无需额外提供。**

---

## 10. 后续扩展路线

1. **CLI Config 命令** — `forge config` 终端交互式配置面板
2. **Web Config UI** — 可视化配置界面
3. **多 Provider 实现** — Notion、Netlify、PlanetScale 等
4. **Codex / OpenClaw / Gemini CLI 适配** — 各平台的工具调用层适配
5. **自定义 Preset** — 用户可发布自己的 preset 模板
6. **插件市场集成** — 支持从 Claude Code Plugin Marketplace 安装

---

## 附录 A: 命令速查

```bash
# 一键全流程
/forge "一句话描述你的产品 idea"

# 独立调用各阶段
/forge:init                     # 初始化项目 + 配置
/forge:brainstorm               # P0 头脑风暴
/forge:prd                      # P1 PRD
/forge:architect                # P2 技术架构
/forge:design                   # P3 设计
/forge:develop                  # P4 开发
/forge:test                     # P5 测试
/forge:verify                   # P6 复验
/forge:deploy                   # P7 部署

# 恢复中断的流程
/forge:resume                   # 从断点继续

# 配置管理
/forge:init --preset claw       # 使用 ClawLabz 预设初始化
/forge:init --preset generic    # 通用预设
```
