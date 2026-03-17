---
name: architect
description: "P2 technical architecture and development plan with security/performance review"
---

# /forge:architect — P2 Technical Architecture & Dev Plan

## Overview

Produce a production-grade technical architecture document and a checklist-based development plan, reviewed from security and performance perspectives.

## Usage

```
/forge:architect
```

## Prerequisites

- P1 PRD must be approved (`docs/forge/P1-prd.md`)

## HARD RULE: Execution Order

Steps MUST execute in strict order: Design architecture → Write dev plan → Self-Review Round 1 (security) → Incorporate fixes → Self-Review Round 2 (performance) → Incorporate fixes → Finalize → Push → Request Approval.

**NEVER push to GitHub or request approval BEFORE both self-review rounds are completed and incorporated.**

## Process

### Step 1: Read Context

Read these files:
- `docs/forge/P1-prd.md` — approved PRD
- `docs/forge/P0-brainstorm-report.md` — technical feasibility section
- `forge.config.yaml` — provider configuration

### Step 2: Technology Selection

Based on PRD requirements and config providers, decide:
- **Frontend framework** — Next.js (default), or alternatives if justified
- **Styling** — Tailwind CSS + component library
- **Backend / API** — Next.js API routes, or separate backend if needed
- **Database** — Per config provider (default: Supabase PostgreSQL)
- **Auth** — Supabase Auth, NextAuth, or custom
- **Storage** — Per config provider (default: Cloudflare R2)
- **Hosting** — Per config provider (default: Vercel)
- **Third-party services** — As identified in PRD

Every choice must include a **rationale** (why this over alternatives).

### Step 3: Architecture Design

Design the system architecture:

1. **System Architecture Diagram** — Mermaid diagram showing all components, data flows, external services
2. **Project Directory Structure** — Full tree with explanations
3. **Database Schema** — Tables, columns, types, relationships, indexes, RLS policies
4. **API Design** — Every route with method, path, request/response format, auth requirement
5. **Authentication & Authorization** — Flow diagram, role-based access, session management
6. **Third-party Integrations** — Each service, what it's used for, fallback plan
7. **Performance Strategy** — Caching (Redis/in-memory), CDN, SSR/SSG/ISR decisions, lazy loading, image optimization
8. **Security Strategy** — Input validation, CSRF, XSS prevention, rate limiting, secrets management, CORS

### Step 4: Development Plan

Create a phased development plan with checklistable tasks:

- Break into 3-6 phases based on dependency order
- Each phase has concrete tasks (max 10 per phase)
- Mark which tasks can run in parallel
- Each task should be completable in a single focused session
- Include checkpoint criteria for each phase (build + lint + test)

### Step 5: Self-Review Round 1 — Security Engineer

Launch the `arch-reviewer` agent with role="security_engineer":

Review checklist:
- [ ] Authentication flow has no bypass paths
- [ ] All user inputs validated at API boundary
- [ ] SQL injection prevention (parameterized queries / ORM)
- [ ] XSS prevention (output encoding, CSP headers)
- [ ] CSRF protection on state-changing endpoints
- [ ] Rate limiting on auth and public endpoints
- [ ] Secrets never in client-side code or git
- [ ] CORS configured restrictively
- [ ] File upload validation (type, size, content)
- [ ] Error messages don't leak internals

Record changes in self-review log.

### Step 6: Self-Review Round 2 — Performance Engineer

Launch the `arch-reviewer` agent with role="performance_engineer":

Review checklist:
- [ ] Database queries have appropriate indexes
- [ ] N+1 query patterns avoided
- [ ] Caching strategy for frequently accessed data
- [ ] Static assets served from CDN
- [ ] Images optimized (WebP, lazy loading, srcset)
- [ ] Bundle size considered (code splitting, tree shaking)
- [ ] SSR/SSG/ISR used appropriately per page
- [ ] API responses paginated where applicable
- [ ] WebSocket or SSE used instead of polling (if real-time needed)
- [ ] Database connection pooling configured

Record changes in self-review log.

### Step 7: Write Documents

Write two documents:

**`docs/forge/P2-architecture.md`:**
```markdown
# Technical Architecture: [Product Name]

> Version: 1.0
> Date: [date]
> Status: Pending Approval
> Based on: P1 PRD

## 1. Technology Stack
| Layer | Choice | Rationale |

## 2. System Architecture
### Architecture Diagram (Mermaid)
### Component Descriptions

## 3. Project Structure
(full directory tree with annotations)

## 4. Database Schema
### Tables
### Relationships (ERD Mermaid)
### Index Strategy
### RLS Policies

## 5. API Design
| Method | Path | Auth | Description | Request | Response |

## 6. Authentication & Authorization
### Auth Flow Diagram
### Roles & Permissions
### Session Management

## 7. Third-party Integrations
| Service | Purpose | Fallback |

## 8. Performance Strategy
### Caching
### Rendering Strategy (SSR/SSG/ISR per page)
### Asset Optimization
### Database Optimization

## 9. Security Strategy
### Input Validation
### CSRF/XSS/Injection Prevention
### Rate Limiting
### Secrets Management
### CORS Policy

## 10. Self-Review Log
### Round 1: Security Review
### Round 2: Performance Review
```

**`docs/forge/P2-dev-plan.md`:**
```markdown
# Development Plan: [Product Name]

> Based on: P2 Architecture

## Phase Overview
| Phase | Focus | Tasks | Parallel Tasks |

## Phase 1: Foundation
- [ ] 1.1: Project scaffolding (Next.js + dependencies)
- [ ] 1.2: Database schema + migrations
- [ ] 1.3: Auth setup
- [ ] 1.4: Base layout + routing
**Checkpoint:** build ✅ lint ✅ auth flow works ✅

## Phase 2: Core Features
- [ ] 2.1: ...
(parallel: 2.1 ∥ 2.2, then 2.3 depends on both)
**Checkpoint:** build ✅ lint ✅ core tests pass ✅

## Phase N: Polish & Integration
- [ ] N.1: ...
**Checkpoint:** build ✅ lint ✅ all tests pass ✅

## Dependency Graph
(Mermaid diagram showing task dependencies)

## Risk Register
| Risk | Impact | Mitigation |
```

### Step 8: Push & Request Approval

1. Git add, commit, push both documents
2. Output:
   ```
   🔴 审批门控

   请审批技术架构: https://github.com/<org>/<repo>/blob/main/docs/forge/P2-architecture.md
   请审批开发计划: https://github.com/<org>/<repo>/blob/main/docs/forge/P2-dev-plan.md

   回复 "approved" 继续，或提出修改意见。
   ```

## Skills & Tools Used

- Architect agent — system design
- Security-reviewer agent — security audit
- `supabase-postgres-best-practices` — database design
- `vercel-react-best-practices` — frontend architecture
- `everything-claude-code:api-design` — API patterns
- `everything-claude-code:backend-patterns` — backend architecture
- `everything-claude-code:database-migrations` — migration strategy
- `security-review` skill — security checklist
- `arch-reviewer` agent — self-review (security + performance perspectives)

## Important

- Architecture must align with `forge.config.yaml` provider choices
- Database schema must support all data entities from PRD
- API routes must cover all user stories from PRD
- Dev plan tasks must be concrete enough to execute without ambiguity
- Do NOT start implementation — this phase is design only
