---
name: deploy
description: "P7 automated deployment: git, database, hosting, DNS, analytics, knowledge base"
---

# /forge:deploy — P7 Automated Deployment & Delivery

## Overview

Full automated deployment pipeline: Git push → database migration → hosting deployment → DNS configuration → health check → analytics setup → knowledge base generation → project summary.

## Usage

```
/forge:deploy
```

## Prerequisites

- P6 Verify completed with PASS status
- All 3 verification rounds done
- No outstanding CRITICAL or HIGH issues

## Process

### Step 1: Pre-Deployment Checklist

Before deploying, verify:
- [ ] All tests pass (`npm test`)
- [ ] Build succeeds (`npm run build`)
- [ ] No CRITICAL/HIGH issues from P6
- [ ] Environment variables documented
- [ ] `.env.local` exists with required values (or secrets configured)
- [ ] `forge.config.yaml` has deployment provider configured

### Step 2: Git Commit & Push

1. Stage all project files (exclude secrets, node_modules, build artifacts)
2. Create comprehensive commit:
   ```
   feat: [Product Name] — initial release

   Full product implementation including:
   - [key feature 1]
   - [key feature 2]
   - [key feature 3]

   Built with Forge automated pipeline.
   ```
3. Push to remote

If GitHub repo doesn't exist yet:
```bash
gh repo create <org>/<project-name> --public --source . --remote origin --push
```

### Step 3: Database Setup

**Provider: Supabase (default)**

Strategy: Check `forge.config.yaml` for `database.strategy`:
- `script_first` → use migration script
- `mcp_first` → try Supabase MCP, fallback to script

Execute migrations:
1. Read all SQL migration files from `supabase/migrations/`
2. Apply in order via Supabase Management API or MCP
3. Verify tables exist and schema matches P2 architecture
4. Set up RLS policies if defined

**Fallback:** `scripts/supabase-migrate.sh` (if available in user's project)

### Step 4: Deploy to Hosting

**Provider: Vercel (default)**

Strategy: Check `forge.config.yaml` for `deployment.strategy`:
- `mcp_first` → try Vercel MCP Server, fallback to CLI
- `script_first` → use deploy script

Deploy steps:
1. Link project to Vercel: `vercel link`
2. Set environment variables: `vercel env add`
3. Deploy to production: `vercel --prod`
4. Capture deployment URL

**Fallback:** `scripts/vercel-deploy.sh` (if available)

### Step 5: DNS Configuration

**Provider: Cloudflare (default)**

Strategy: Check `forge.config.yaml` for `dns.strategy`.

1. Read target domain from `forge.config.yaml` project.domain
2. Add CNAME record pointing to Vercel:
   ```
   <subdomain>.example.com → cname.vercel-dns.com
   ```
3. Enable Cloudflare proxy (if desired)
4. Verify DNS propagation

**Fallback:** `scripts/cloudflare-dns.sh` (if available)

### Step 6: Add Custom Domain to Hosting

1. Add domain to Vercel project: `vercel domains add <domain>`
2. Verify SSL certificate is provisioned
3. Test HTTPS access

### Step 7: Health Check

1. Wait for deployment to be live (check deployment status)
2. HTTP GET the production URL — expect 200
3. Check key pages:
   - Homepage → 200
   - Login page → 200
   - API health endpoint → 200 with expected response
4. Use `browser-use` to take screenshots of:
   - Homepage (desktop + mobile)
   - Key feature page
   - Login/signup page

Save screenshots to `.forge/screenshots/production/`

If health check fails:
1. Check deployment logs
2. Check environment variables
3. Fix issues and redeploy
4. Retry health check (max 3 attempts)

### Step 8: Analytics Setup (Optional)

Check `forge.config.yaml` optional.analytics:

**GA4 (if enabled and measurement_id provided):**
- Use `ga4-clarity-auto-integration` skill
- Inject GA4 tracking script into the app
- Verify tracking is working (check real-time report or network tab)

**Clarity (if enabled and project_id provided):**
- Use `ga4-clarity-auto-integration` skill
- Inject Clarity script
- Verify Clarity is recording

If measurement_id or project_id is empty → skip with note in report.

### Step 9: Smoke Test

After deployment + analytics:
1. Run E2E tests against production URL (subset of critical flows)
2. Verify:
   - User can sign up / log in
   - Core feature works end-to-end
   - No console errors in browser
3. Take final screenshots

### Step 10: Knowledge Base Generation

If `forge.config.yaml` optional.knowledge_base.enabled is true:

Generate `knowledge/` directory:

**`knowledge/index.json`:**
```json
{
  "project": "[name]",
  "generated": "[date]",
  "files": {
    "src/app/page.tsx": "Homepage component",
    "src/app/api/auth/route.ts": "Authentication API endpoint",
    ...
  },
  "routes": ["/", "/dashboard", "/api/auth", ...],
  "tables": ["users", "projects", ...],
  "external_services": ["Supabase", "Vercel", ...]
}
```

**`knowledge/architecture.md`:**
Condensed version of P2 architecture (key decisions, tech stack, system diagram).

**`knowledge/api-reference.md`:**
Auto-generated from API route files — method, path, auth, description, request/response.

**`knowledge/database-schema.md`:**
Auto-generated from migration files — tables, columns, relationships.

**`knowledge/decisions.md`:**
Key technical decisions made during the project with rationale (extracted from P2 + P0).

**`knowledge/onboarding.md`:**
```markdown
# Onboarding Guide: [Product Name]

## Quick Start
1. Clone: `git clone <repo>`
2. Install: `npm install`
3. Env: Copy `.env.example` to `.env.local`, fill in values
4. DB: Run `npm run db:migrate`
5. Dev: `npm run dev` → http://localhost:3000

## Project Structure
[simplified directory tree with explanations]

## Key Files
[most important files and what they do]

## Common Tasks
- Adding a new page: ...
- Adding a new API endpoint: ...
- Running tests: ...
- Deploying: ...
```

### Step 11: Project Summary

Write `docs/forge/P7-project-summary.md`:

```markdown
# Project Delivery Report: [Product Name]

> Generated: [date]
> Pipeline: Forge v1.0

## Live Product

| | |
|---|---|
| **URL** | https://[domain] |
| **Repository** | https://github.com/[org]/[repo] |
| **Status** | ✅ Live |

## Technology Stack

| Layer | Choice |
|-------|--------|
| Frontend | [Next.js 15] |
| Styling | [Tailwind CSS + shadcn/ui] |
| Database | [Supabase PostgreSQL] |
| Auth | [Supabase Auth] |
| Hosting | [Vercel] |
| DNS | [Cloudflare] |
| Storage | [Cloudflare R2] |
| Analytics | [GA4 + Clarity] |

## Feature Completion

| PRD Feature (P0) | Status |
|-------------------|--------|
| [Feature 1] | ✅ |
| [Feature 2] | ✅ |

## Database Tables

| Table | Description | Rows |
|-------|-------------|------|

## API Routes

| Method | Path | Auth | Description |
|--------|------|------|-------------|

## Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL | Yes |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase admin key | Yes |

## Test Coverage
- Unit: XX%
- Integration: XX passed
- E2E: XX passed
- Lighthouse: Performance XX, Accessibility XX

## Verification Summary
- Round 1: XX found, XX fixed
- Round 2: XX found, XX fixed
- Round 3: XX found, XX fixed

## Known Limitations
- [Any acknowledged limitations or future improvements]

## Document Index
| Document | Path |
|----------|------|
| Brainstorm Report | docs/forge/P0-brainstorm-report.md |
| PRD | docs/forge/P1-prd.md |
| Architecture | docs/forge/P2-architecture.md |
| Dev Plan | docs/forge/P2-dev-plan.md |
| Design Spec | docs/forge/P3-design-spec.md |
| Test Report | docs/forge/P5-test-report.md |
| Verify Round 1 | docs/forge/P6-verify-round-1.md |
| Verify Round 2 | docs/forge/P6-verify-round-2.md |
| Verify Round 3 | docs/forge/P6-verify-round-3.md |
| This Report | docs/forge/P7-project-summary.md |

## Knowledge Base
See `knowledge/` directory for:
- `architecture.md` — Architecture overview
- `api-reference.md` — API documentation
- `database-schema.md` — Database documentation
- `decisions.md` — Technical decisions log
- `onboarding.md` — Developer onboarding guide
```

### Step 12: Final Commit & Push

1. Commit knowledge base + project summary + any deployment config changes
2. Push to remote
3. Update `.forge/state.yaml` → all phases completed

### Step 13: Delivery Output

Output to user:
```
✅ Forge Pipeline Complete!

🌐 Live: https://[domain]
📦 Repo: https://github.com/[org]/[repo]
📋 Summary: docs/forge/P7-project-summary.md
📚 Knowledge Base: knowledge/

The product is live and ready for users.
```

## Skills & Tools Used

- `devops` skill — deployment workflow
- `everything-claude-code:deployment-patterns` — deployment patterns
- Vercel MCP Server / `scripts/vercel-deploy.sh` — Vercel deployment
- Supabase MCP Server / `scripts/supabase-migrate.sh` — database migration
- Cloudflare MCP Server / `scripts/cloudflare-dns.sh` — DNS configuration
- `ga4-clarity-auto-integration` skill — analytics setup (optional)
- `browser-use` — health check screenshots + smoke test

## Important

- NEVER deploy if P6 verification has outstanding CRITICAL issues
- ALWAYS verify health check passes before declaring success
- ALWAYS generate knowledge base (unless explicitly disabled)
- If deployment fails, attempt rollback and inform user
- Environment variables with secrets go to Vercel env vars, NEVER to git
