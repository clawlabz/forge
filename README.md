# Forge

**From idea to deployed product, fully automated.**

Forge is a Claude Code plugin that automates the entire product manufacturing pipeline — from brainstorming through development, testing, and deployment. Give it an idea, and it delivers a live, production-ready product.

## Pipeline

```
Idea → P0 Brainstorm → P1 PRD 🔴 → ┌ P2 Architect (agent) 🔴 ┐
                                     │ P3 Design (Stitch)  🔴 │
                                     └────────────────────────┘
→ P4 Develop ⚡→ P5 Test ⚡→ P6 Verify ×3 ⚡→ P7 Deploy ⚡→ ✅ Live Product

🔴 = Approval gate (pushes to GitHub, waits for human review)
⚡ = Fully automatic, no pause
```

| Phase | Command | What It Does |
|-------|---------|-------------|
| P0 | `/forge:brainstorm` | Market research, competitor analysis, feasibility, business model |
| P1 | `/forge:prd` | PRD with 2 rounds of self-review |
| P2 | `/forge:architect` | Tech architecture + dev plan with security/performance review |
| P3 | `/forge:design` | UI design generation via Google Stitch MCP |
| P4 | `/forge:develop` | Multi-agent parallel development with checkpoints |
| P5 | `/forge:test` | Unit, integration, E2E tests |
| P6 | `/forge:verify` | 3-round multi-perspective verification |
| P7 | `/forge:deploy` | Git, database, hosting, DNS, analytics, knowledge base |

## Quick Start

### Install

```bash
# Add Forge marketplace
claude plugin marketplace add clawlabz/forge

# Install plugin
claude plugin install forge
```

### One-Command Full Pipeline

```bash
/forge:build "A real-time collaborative whiteboard for remote teams"
```

### Step by Step

```bash
/forge:init --preset generic       # Initialize project
/forge:brainstorm "your idea"      # P0: Research & validate
/forge:prd                         # P1: Write PRD (approval required)
/forge:architect                   # P2: Architecture (approval required)
/forge:design                      # P3: Design via Stitch (approval required)
/forge:develop                     # P4: Build it
/forge:test                        # P5: Test it
/forge:verify                      # P6: 3-round review
/forge:deploy                      # P7: Ship it
```

### Resume After Interruption

```bash
/forge:resume                      # Pick up from last checkpoint
```

## Design Generation (Google Stitch)

Forge uses [Google Stitch](https://stitch.withgoogle.com) as the default design provider. Claude writes product briefs, Stitch generates professional UI designs.

**How it works:**
1. Claude reads the PRD and crafts page-by-page prompts (product context only, no visual specs)
2. Stitch generates each page's UI design using Gemini
3. Homepage is generated first to establish the visual style
4. Subsequent pages maintain consistency via style reference prompts
5. Screenshots are downloaded and embedded in the design spec for approval
6. HTML code is saved locally for development reference

**Setup:**
```bash
# Install Stitch MCP dependencies
pip install mcp requests  # requires Python 3.10+

# Add Stitch MCP server to Claude Code
claude mcp add stitch -e STITCH_API_KEY=<your-key> -- python3 /path/to/stitch_mcp.py
```

Get your API key at [stitch.withgoogle.com](https://stitch.withgoogle.com).

**Fallback:** If Stitch is unavailable, Forge falls back to Claude-generated design specifications.

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
  brainstorm: "opus"       # Deep reasoning for research
  prd: "opus"              # Deep reasoning for requirements
  architect: "opus"        # Deep reasoning for architecture
  design: "sonnet"         # Design prompt crafting
  develop: "sonnet"        # Code generation
  test: "sonnet"           # Test writing
  verify: "opus"           # Deep audit
  deploy: "haiku"          # Fast execution tasks
  fallback: "current"      # When platform doesn't support model switching

# Pluggable providers — swap any service
providers:
  deployment: { type: "vercel" }        # vercel | cloudflare_pages | netlify
  database:   { type: "supabase" }      # supabase | planetscale | neon
  dns:        { type: "cloudflare" }    # cloudflare | vercel_dns | route53
  design:     { type: "stitch" }        # stitch (default) | manual_spec | figma
  storage:    { type: "cloudflare_r2" } # cloudflare_r2 | s3 | supabase_storage

# Optional features
optional:
  analytics:
    ga4:     { enabled: true, measurement_id: "" }
    clarity: { enabled: true, project_id: "" }
  knowledge_base: { enabled: true }
```

### Presets

```bash
/forge:init --preset claw           # ClawLabz ecosystem defaults
/forge:init --preset generic        # Empty config, configure manually
```

## Provider Architecture

Every integration point is a pluggable provider:

| Concern | Default | Alternatives |
|---------|---------|-------------|
| Design | **Stitch** (Google AI) | Manual Spec, Figma MCP |
| Deployment | Vercel | Cloudflare Pages, Netlify |
| Database | Supabase | PlanetScale, Neon |
| DNS | Cloudflare | Vercel DNS, Route53 |
| Storage | Cloudflare R2 | S3, Supabase Storage |
| Doc Hosting | GitHub | Notion, Cloudflare Pages |
| Testing | Playwright | Cypress |
| Analytics | GA4 + Clarity | — |

Providers use a dual-layer strategy: MCP server first, script fallback.

## Approval Flow

Three gates pause the pipeline and push documents to GitHub:

```
🔴 审批门控 — P1 PRD
请审批 PRD: https://github.com/org/project/blob/main/docs/forge/P1-prd.md

🔴 审批门控 — P2 Architecture
请审批技术架构: https://github.com/org/project/blob/main/docs/forge/P2-architecture.md

🔴 审批门控 — P3 Design
请审批设计方案: https://github.com/org/project/blob/main/docs/forge/P3-design-spec.md
(包含每个页面的截图，可直接在 GitHub 上查看)
```

Review on any device. Reply "approved" or provide revision feedback.

After P3 approval, P4→P5→P6→P7 runs fully automatically with zero human intervention.

## State & Resume

Forge tracks pipeline state in `.forge/state.yaml`. Every phase transition, approval, and sub-task completion is persisted. If your session is interrupted, `/forge:resume` reads the state file and continues from exactly where it stopped.

## Deliverables

When the pipeline completes, you get:

1. **Live product** at your configured domain
2. **Git repository** with clean commit history
3. **Documentation** in `docs/forge/` — PRD, architecture, design spec (with screenshots), test reports, verification reports, deploy report
4. **Knowledge base** in `knowledge/` — architecture overview, API reference, database schema, onboarding guide
5. **Project summary** — complete handoff document with all links and status

## Tested Products

Built with Forge during development:

| Product | Stack | Result |
|---------|-------|--------|
| [ClawToolkit](https://toolkit.clawlabz.xyz) | Next.js + Supabase + Stripe | 28 routes, 40 files |
| [ClawMark](https://mark.clawlabz.xyz) | Next.js + localStorage | 12 pages, 42 files |
| ClawHealth | Next.js + Supabase + DeepSeek | 12 pages, 93 files |

## License

MIT
