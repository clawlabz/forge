# Forge

**From idea to deployed product, fully automated.**

Forge is a Claude Code plugin that automates the entire product manufacturing pipeline — from initial brainstorming through development, testing, and deployment. Give it an idea, and it delivers a live, production-ready product.

## Pipeline

```
Idea → P0 Brainstorm → P1 PRD 🔴 → P2 Architect 🔴 → P4 Develop → P5 Test
                                    P3 Design    🔴 ↗
       → P6 Verify (×3) → P7 Deploy → ✅ Live Product

🔴 = Approval gate (pushes doc to GitHub, waits for human review)
```

| Phase | Command | What It Does |
|-------|---------|-------------|
| P0 | `/forge:brainstorm` | Market research, competitor analysis, feasibility, business model |
| P1 | `/forge:prd` | PRD with 2 rounds of self-review |
| P2 | `/forge:architect` | Tech architecture + dev plan with security/performance review |
| P3 | `/forge:design` | Design system, component specs, page layouts |
| P4 | `/forge:develop` | Multi-agent parallel development with checkpoints |
| P5 | `/forge:test` | Unit, integration, E2E tests with browser screenshots |
| P6 | `/forge:verify` | 3-round multi-perspective verification |
| P7 | `/forge:deploy` | Git, database, hosting, DNS, analytics, knowledge base |

## Quick Start

### Install

```bash
claude plugin add clawlabz/forge
```

### One-Command Full Pipeline

```bash
/forge "A real-time collaborative whiteboard for remote teams"
```

### Step by Step

```bash
/forge:init --preset generic       # Initialize project
/forge:brainstorm "your idea"      # P0: Research & validate
/forge:prd                         # P1: Write PRD (approval required)
/forge:architect                   # P2: Architecture (approval required)
/forge:design                      # P3: Design (approval required)
/forge:develop                     # P4: Build it
/forge:test                        # P5: Test it
/forge:verify                      # P6: 3-round review
/forge:deploy                      # P7: Ship it
```

### Resume After Interruption

```bash
/forge:resume                      # Pick up from last checkpoint
```

## Configuration

Forge uses `forge.config.yaml` at the project root:

```yaml
version: "1.0"

project:
  name: "my-product"
  description: "A real-time collaborative whiteboard"
  domain: "whiteboard.example.com"

# Route different LLMs to different phases
models:
  brainstorm: "opus"      # Deep reasoning
  prd: "opus"
  architect: "opus"
  design: "sonnet"        # Or gemini-pro for design-focused models
  develop: "sonnet"       # Best coding model
  test: "sonnet"
  verify: "opus"          # Deep audit
  deploy: "haiku"         # Fast execution
  fallback: "current"     # When platform doesn't support switching

# Pluggable providers — swap any service
providers:
  deployment: { type: "vercel" }       # vercel | cloudflare_pages | netlify
  database:   { type: "supabase" }     # supabase | planetscale | neon
  dns:        { type: "cloudflare" }   # cloudflare | vercel_dns | route53
  design:     { type: "manual_spec" }  # manual_spec | figma | stitch

# Optional features
optional:
  analytics:
    ga4:     { enabled: true, measurement_id: "" }
    clarity: { enabled: true, project_id: "" }
  knowledge_base: { enabled: true }
```

### Presets

```bash
/forge:init --preset claw           # ClawLabz ecosystem (Vercel + Supabase + Cloudflare)
/forge:init --preset generic        # Empty config, configure manually
```

## Provider Architecture

Every integration point is a pluggable provider:

| Concern | Default | Alternatives |
|---------|---------|-------------|
| Doc Hosting | GitHub | Notion, Cloudflare Pages |
| Deployment | Vercel | Cloudflare Pages, Netlify |
| Database | Supabase | PlanetScale, Neon |
| DNS | Cloudflare | Vercel DNS, Route53 |
| Storage | Cloudflare R2 | S3, Supabase Storage |
| Design | Manual Spec | Figma MCP, Stitch |
| Testing | Playwright | Cypress |
| Analytics | GA4 + Clarity | — |

Providers use a dual-layer strategy: MCP server first, script fallback.

## State & Resume

Forge tracks pipeline state in `.forge/state.yaml`. Every phase transition, approval, and sub-task completion is persisted. If your session is interrupted, `/forge:resume` reads the state file and continues from exactly where it stopped.

## Approval Flow

Three approval gates pause the pipeline and push the document to GitHub:

```
📋 请审批 PRD: https://github.com/org/project/blob/main/docs/forge/P1-prd.md
```

Review on any device (phone, tablet, desktop). Reply with approval or revision requests to continue.

## Skills & Tools Used

Forge orchestrates 20+ skills and agents across the pipeline:

- **Research**: agent-reach, market-research, search-first
- **Planning**: brainstorming, blueprint, writing-plans
- **Architecture**: architect agent, api-design, backend-patterns, database-migrations
- **Design**: ui-ux-pro-max, frontend-design, frontend-patterns, Figma MCP
- **Development**: tdd-guide, code-reviewer, build-error-resolver, fullstack-guardian, vercel-react-best-practices
- **Testing**: e2e-runner, browser-use, webapp-testing, javascript-testing-patterns
- **Security**: security-review, security-scan
- **Deployment**: devops, deployment-patterns, Vercel/Supabase/Cloudflare MCP servers
- **Analytics**: ga4-clarity-auto-integration

## Deliverables

When the pipeline completes, you get:

1. **Live product** at your configured domain
2. **Git repository** with clean commit history
3. **Documentation** in `docs/forge/` (PRD, architecture, design, test reports, deploy report)
4. **Knowledge base** in `knowledge/` (architecture overview, API reference, onboarding guide)
5. **Project summary** — complete handoff document with all links and status

## License

MIT
