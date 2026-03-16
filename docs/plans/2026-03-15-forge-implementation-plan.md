# Forge — 实现计划

> **基于**: [Forge 设计文档](./2026-03-15-forge-design.md)
> **日期**: 2026-03-15

---

## Phase 划分

共 4 个 Phase，按依赖关系串行执行，Phase 内的 task 尽可能并行。

---

## Phase 1: Plugin 骨架 + 配置系统

**目标：** 搭建可安装的 Claude Code plugin 结构，配置系统可用。

### Tasks

- [ ] **1.1** 创建 `package.json` — plugin 元数据（name: `@clawlabz/forge`、description、keywords、repository）
- [ ] **1.2** 创建 `SKILL.md` — plugin 入口，注册所有 skills 的命令名和触发描述
- [ ] **1.3** 创建 `README.md` — 项目说明、安装方式、命令速查、配置说明
- [ ] **1.4** 创建 `skills/init.md` — `/forge:init` skill，初始化项目目录结构 + `forge.config.yaml` + `.forge/state.yaml`，支持 `--preset claw|generic`
- [ ] **1.5** 创建 `templates/forge.config.yaml` — 配置文件模板
- [ ] **1.6** 创建 `presets/claw.yaml` + `presets/generic.yaml` — 预设模板
- [ ] **1.7** 创建 `skills/resume.md` — `/forge:resume` skill，读取 `.forge/state.yaml`，输出摘要，从断点继续
- [ ] **1.8** 更新 `.gitignore` — 适配 plugin 项目结构

### 并行关系
```
1.1 + 1.8 (独立)
  → 1.2 (依赖 1.1)
  → 1.5 + 1.6 (独立)
    → 1.4 (依赖 1.5, 1.6)
    → 1.7 (依赖 1.5)
1.3 (独立，最后写，汇总所有信息)
```

### 验收标准
- [ ] `claude plugin install /path/to/forge` 可安装
- [ ] `/forge:init --preset generic` 生成正确的目录结构和配置文件
- [ ] `/forge:resume` 能正确读取 state 文件并输出摘要

---

## Phase 2: 核心阶段 Skills (P0-P3)

**目标：** 实现前 4 个阶段的 skill + 审批门控 + 自审 agent。

### Tasks

- [ ] **2.1** 创建 `skills/forge.md` — orchestrator 主入口，编排 P0→P7 全流程，管理状态转移、审批门控、并行调度
- [ ] **2.2** 创建 `skills/brainstorm.md` — P0 skill，调用 agent-reach/brainstorming/market-research，产出 P0 报告
- [ ] **2.3** 创建 `skills/prd.md` — P1 skill，撰写 PRD + 2 轮自审 + GitHub 推送 + 审批请求
- [ ] **2.4** 创建 `agents/prd-reviewer.md` — PRD 自审 agent（产品经理视角 + 用户视角）
- [ ] **2.5** 创建 `skills/architect.md` — P2 skill，架构设计 + 开发 Plan + 2 轮自审 + 审批
- [ ] **2.6** 创建 `agents/arch-reviewer.md` — 架构自审 agent（安全视角 + 性能视角）
- [ ] **2.7** 创建 `skills/design.md` — P3 skill，设计规范 + 页面方案 + 审批
- [ ] **2.8** 创建 `agents/design-reviewer.md` — 设计自审 agent
- [ ] **2.9** 创建文档模板 `templates/prd-template.md`
- [ ] **2.10** 创建文档模板 `templates/architecture-template.md`
- [ ] **2.11** 创建文档模板 `templates/design-spec-template.md`

### 并行关系
```
2.1 (独立，但最复杂)
2.2 (独立)
2.3 + 2.4 + 2.9 (PRD 组，可并行开发)
2.5 + 2.6 + 2.10 (架构组，可并行开发)
2.7 + 2.8 + 2.11 (设计组，可并行开发)
```

### 验收标准
- [ ] `/forge:brainstorm "一句话idea"` 产出完整的 P0 调研报告
- [ ] `/forge:prd` 产出 PRD 并包含 2 轮自审记录
- [ ] `/forge:architect` 产出架构文档 + dev plan
- [ ] `/forge:design` 产出设计规范
- [ ] 审批门控：文档推送 GitHub 后输出审批链接并暂停
- [ ] `/forge "idea"` orchestrator 能串联 P0→P1→(P2∥P3) 并在审批点暂停

---

## Phase 3: 执行阶段 Skills (P4-P7)

**目标：** 实现开发、测试、复验、部署四个执行阶段。

### Tasks

- [ ] **3.1** 创建 `skills/develop.md` — P4 skill，读取 dev plan → 按 phase 执行 → 多 Agent 并行 → checkpoint → 进度更新
- [ ] **3.2** 创建 `skills/test.md` — P5 skill，单元/集成/E2E 测试 → 浏览器截图 → 失败自动修复 → 测试报告
- [ ] **3.3** 创建 `skills/verify.md` — P6 skill，3 轮多 Agent 并行复验 → 生成待办 → 并行修复 → 复验报告
- [ ] **3.4** 创建 `agents/verify-checker.md` — 复验审查 agent（多维度并行审查）
- [ ] **3.5** 创建 `skills/deploy.md` — P7 skill，Git推送 → DB迁移 → 部署 → DNS → 健康检查 → GA4/Clarity(可选) → 知识库 → 项目总结
- [ ] **3.6** 创建文档模板 `templates/test-report-template.md`
- [ ] **3.7** 创建文档模板 `templates/verify-report-template.md`
- [ ] **3.8** 创建文档模板 `templates/project-summary-template.md`

### 并行关系
```
3.1 → 3.2 → 3.3 (串行，有依赖)
3.4 (与 3.3 并行开发)
3.5 (独立)
3.6 + 3.7 + 3.8 (模板，独立并行)
```

### 验收标准
- [ ] `/forge:develop` 能读取 plan 并按 phase 执行开发，支持多 Agent 并行
- [ ] `/forge:test` 运行完整测试套件并产出报告
- [ ] `/forge:verify` 执行 3 轮复验并产出报告
- [ ] `/forge:deploy` 完成全自动部署 + 知识库生成 + 项目总结
- [ ] `/forge "idea"` 全流程端到端可运行

---

## Phase 4: Provider 文档 + 发布

**目标：** 完善 Provider 扩展文档，发布 plugin。

### Tasks

- [ ] **4.1** 创建 `providers/README.md` — Provider 开发指南（接口约定、扩展步骤）
- [ ] **4.2** 创建各 Provider 说明文档 `providers/{doc-hosting,deployment,database,dns,design,analytics}/`
- [ ] **4.3** 端到端集成测试 — 用一个真实 idea 跑完 `/forge` 全流程
- [ ] **4.4** 完善 `README.md` — 最终版（含 GIF 演示、完整文档）
- [ ] **4.5** 发布到 Claude Code Plugin Marketplace

### 并行关系
```
4.1 + 4.2 (并行)
  → 4.3 (集成测试)
    → 4.4 → 4.5 (串行)
```

### 验收标准
- [ ] Provider 文档清晰可读，第三方开发者能按指南扩展新 Provider
- [ ] 端到端测试通过：一个真实 idea 从 P0 到 P7 全流程成功
- [ ] Plugin 可通过 marketplace 安装使用

---

## 风险点与应对

| 风险 | 影响 | 应对方案 |
|------|------|---------|
| Skill 文件过长导致 Claude 执行不稳定 | 阶段执行失败 | 每个 skill 控制在 500 行以内，复杂逻辑拆分到 agent |
| 审批门控中断后状态丢失 | 无法恢复 | state.yaml 在每个关键节点写入，resume skill 验证 |
| 多 Agent 并行时资源竞争 | 代码冲突 | 并行 task 必须操作不同文件/模块，Plan 中标注依赖关系 |
| MCP Server 不可用 | 部署失败 | 双层 Provider (mcp_first + script fallback) |
| 模型路由平台不支持 | 无法切模型 | fallback 到 current 模型，静默降级 |

---

## 时间预估

| Phase | 预计 Tasks | 说明 |
|-------|-----------|------|
| Phase 1 | 8 tasks | Plugin 骨架，基础设施 |
| Phase 2 | 11 tasks | 核心阶段，最复杂 |
| Phase 3 | 8 tasks | 执行阶段 |
| Phase 4 | 5 tasks | 文档 + 发布 |
| **合计** | **32 tasks** | |
