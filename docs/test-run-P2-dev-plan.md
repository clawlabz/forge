# ClawToolkit — Development Plan

> Version: 1.0
> Date: 2026-03-15
> Status: Draft
> Architecture Reference: [P2-architecture.md](./P2-architecture.md)
> Estimated Duration: 4 weeks (1 developer)

---

## Dependency Graph

```
Phase 1: Foundation
  ├── T1.1 Project scaffold
  ├── T1.2 Database migrations          ← depends on T1.1
  ├── T1.3 Supabase client + RLS        ← depends on T1.2
  ├── T1.4 Auth (Supabase Auth)         ← depends on T1.3
  ├── T1.5 Rate limiter (Upstash)       ← depends on T1.1
  └── T1.6 R2 storage utility           ← depends on T1.1
      (T1.1 → T1.2 → T1.3 → T1.4 sequential)
      (T1.5, T1.6 parallel with T1.2-T1.4)

Phase 2: Core Tool System
  ├── T2.1 Tool CRUD APIs               ← depends on T1.3
  ├── T2.2 Category APIs                ← depends on T1.3 (parallel with T2.1)
  ├── T2.3 Search API (tsvector)        ← depends on T2.1
  ├── T2.4 MCP Schema parser            ← depends on T2.1
  ├── T2.5 Tool publishing form (UI)    ← depends on T2.1, T2.2, T2.4
  ├── T2.6 Tool listing pages (UI)      ← depends on T2.1, T2.2, T2.3
  └── T2.7 Tool detail page (UI)        ← depends on T2.1
      (T2.1, T2.2 parallel)
      (T2.3, T2.4 parallel after T2.1)
      (T2.5, T2.6, T2.7 parallel after deps met)

Phase 3: API Gateway + Billing
  ├── T3.1 API Key system               ← depends on T1.3, T1.4
  ├── T3.2 Gateway proxy engine          ← depends on T3.1, T2.1
  ├── T3.3 Protocol adapters (REST/MCP)  ← depends on T3.2
  ├── T3.4 Free tier tracking            ← depends on T3.2
  ├── T3.5 CP billing integration        ← depends on T3.2
  ├── T3.6 Invocation recording          ← depends on T3.2
  ├── T3.7 Stripe top-up flow            ← depends on T1.3
  ├── T3.8 Stripe Connect (publisher)    ← depends on T1.3
      (T3.1, T3.7, T3.8 parallel)
      (T3.2 after T3.1)
      (T3.3, T3.4, T3.5, T3.6 parallel after T3.2)

Phase 4: Dashboards + Polish
  ├── T4.1 Developer Dashboard (UI)      ← depends on T3.6, T3.5
  ├── T4.2 Publisher Dashboard (UI)      ← depends on T3.6, T3.8
  ├── T4.3 API Key management (UI)       ← depends on T3.1
  ├── T4.4 Top-up page (UI)             ← depends on T3.7
  ├── T4.5 Settings page (UI)           ← depends on T1.4
  ├── T4.6 API docs page                ← depends on T3.2
  ├── T4.7 Landing page (UI)            ← depends on T2.6
  ├── T4.8 Error handling + edge cases   ← depends on all above
      (T4.1-T4.7 all parallel)
      (T4.8 after all)

Phase 5: Testing + Deploy
  ├── T5.1 Unit tests                    ← parallel throughout
  ├── T5.2 Integration tests             ← depends on Phase 3
  ├── T5.3 E2E smoke tests              ← depends on Phase 4
  ├── T5.4 Security audit               ← depends on Phase 4
  ├── T5.5 Performance testing           ← depends on T3.2
  ├── T5.6 Vercel deployment             ← depends on T5.3
  ├── T5.7 DNS + domain setup            ← parallel with T5.6
  └── T5.8 Seed data + go-live           ← depends on T5.6, T5.7
```

---

## Phase 1: Foundation (Days 1-4)

### Goal
Project scaffold, database, auth, and utility infrastructure.

### Tasks

- [ ] **T1.1 — Project Scaffold** (Day 1)
  - [ ] Create `apps/toolkit/` in claw-platform monorepo
  - [ ] Initialize Next.js 15 (App Router) with TypeScript
  - [ ] Configure Tailwind v4 + shadcn/ui
  - [ ] Add to `pnpm-workspace.yaml` and `turbo.json`
  - [ ] Set up `tsconfig.json` extending root config
  - [ ] Create `.env.local.example` with all required env vars
  - [ ] Create root layout with basic HTML structure, metadata, fonts
  - [ ] Create `middleware.ts` skeleton (auth redirect + rate limit placeholder)
  - [ ] Add dev/build scripts to root `package.json`: `dev:toolkit`, `build:toolkit`
  - [ ] Verify `pnpm dev:toolkit` starts on port `:3400`

- [ ] **T1.2 — Database Migrations** (Day 1-2)
  - [ ] Migration 038: `toolkit_categories` table + seed 12 categories
  - [ ] Migration 039: `toolkit_organizations` table (P2 placeholder, empty)
  - [ ] Migration 040: `toolkit_org_members` table (P2 placeholder, empty)
  - [ ] Migration 041: `toolkit_users` table + indexes
  - [ ] Migration 042: `toolkit_tools` table + all indexes including GIN for tsvector and tags
  - [ ] Migration 043: `toolkit_api_keys` table + indexes
  - [ ] Migration 044: `toolkit_invocations` table + indexes
  - [ ] Migration 045: `toolkit_free_tier_usage` table + unique constraint
  - [ ] Migration 046: `toolkit_stripe_events` table
  - [ ] Migration 047: `toolkit_publisher_earnings` table + indexes
  - [ ] Migration 048: RLS policies for all toolkit tables
  - [ ] Run migrations against dev Supabase, verify all tables + indexes created
  - [ ] Document migration sequence in `supabase/migrations/` README

- [ ] **T1.3 — Supabase Client + Helpers** (Day 2)
  - [ ] Create `lib/supabase.ts` — service role client singleton (following Market pattern)
  - [ ] Create `lib/supabase-browser.ts` — anon client for browser
  - [ ] Create Zod type definitions for all DB tables in `lib/validators/`
  - [ ] Create `lib/constants.ts` with rate limits, free tier config, CP rates, pricing bounds

- [ ] **T1.4 — Authentication** (Day 2-3)
  - [ ] Configure Supabase Auth: enable GitHub OAuth provider + email/password
  - [ ] Create `POST /api/v1/auth/signup` — email registration + user provisioning + 1000 CP bonus
  - [ ] Create `POST /api/v1/auth/login` — email login
  - [ ] Create `GET /auth/callback` — OAuth redirect handler + user provisioning
  - [ ] Create `lib/auth.ts` — `getSessionUser()`, `requireAuth()` helpers
  - [ ] Create auth pages: `/auth/login`, `/auth/signup` with GitHub button + email form
  - [ ] Create `components/auth/auth-guard.tsx` — redirect unauthenticated users
  - [ ] Wire `middleware.ts` to protect `/dashboard/*`, `/publish`, `/settings` routes
  - [ ] Test: signup with email, login, GitHub OAuth, session persistence, logout

- [ ] **T1.5 — Rate Limiter** (Day 3, parallel with T1.4)
  - [ ] Install `@upstash/ratelimit` + `@upstash/redis`
  - [ ] Create `lib/rate-limiter.ts` — sliding window rate limiter factory
  - [ ] Define rate limit tiers: auth (10/min/IP), gateway (1000/min/key), publish (5/min/user), dashboard (60/min/user), public (100/min/IP)
  - [ ] Integrate into `middleware.ts` for auth + public routes
  - [ ] Gateway rate limiting integrated in T3.2

- [ ] **T1.6 — R2 Storage Utility** (Day 3, parallel with T1.4)
  - [ ] Create `lib/r2.ts` — upload, download, delete, presigned URL generation
  - [ ] Configure R2 bucket `claw-toolkit` via Cloudflare dashboard
  - [ ] Test upload/download of tool icon image

### Checkpoint Criteria
- [ ] `pnpm dev:toolkit` runs, pages render at `:3400`
- [ ] All 11 migration files applied, tables visible in Supabase dashboard
- [ ] User can sign up (email), log in, see session info on a test page
- [ ] GitHub OAuth flow completes end-to-end
- [ ] New user gets 1000 CP on registration (verified in `shared_point_accounts`)
- [ ] Rate limiter returns 429 when limit exceeded (unit test)

---

## Phase 2: Core Tool System (Days 5-10)

### Goal
Tool CRUD, search, MCP import, and all public-facing tool pages.

### Tasks

- [ ] **T2.1 — Tool CRUD APIs** (Day 5-6)
  - [ ] Create `lib/validators/tool.ts` — Zod schemas for create, update, list query params
  - [ ] Create `GET /api/v1/tools` — list tools with pagination (page, limit, sort)
    - Sort options: `popular` (total_calls DESC), `newest` (created_at DESC), `price_low`, `price_high`
    - Filter: `category` (slug), `protocol` (mcp/rest/custom), `status` (active only for public)
    - Response: `{ tools, total, page, limit }`
  - [ ] Create `GET /api/v1/tools/:slug` — tool detail with publisher info
  - [ ] Create `POST /api/v1/tools/publish` — create tool (auth required, sets status=draft)
    - Validate: slug uniqueness, endpoint_url is HTTPS, pricing_per_call in range 1-100
    - Auto-generate slug from name if not provided (slugify + uniqueness check)
    - Upload icon to R2 if provided
  - [ ] Create `PATCH /api/v1/tools/:slug` — update tool (owner only)
    - Cannot change slug once active
    - Status transitions: draft→pending, pending→active (admin only), active→suspended (admin), any→archived (owner)
  - [ ] Create `DELETE /api/v1/tools/:slug` — soft archive tool (owner only, sets status=archived)

- [ ] **T2.2 — Category APIs** (Day 5, parallel with T2.1)
  - [ ] Create `GET /api/v1/categories` — list all categories with tool counts
  - [ ] Verify seed data: 12 categories present from migration

- [ ] **T2.3 — Search API** (Day 6-7)
  - [ ] Create `lib/search.ts` — full-text search using `tsvector` + `ts_rank`
  - [ ] Extend `GET /api/v1/tools` with `q` parameter for full-text search
    - Query: `to_tsquery('english', sanitized_input)` with prefix matching
    - Rank: `ts_rank(search_vector, query)` as relevance score
    - Combined sort: relevance first, then total_calls for tie-breaking
  - [ ] Support tag filtering: `tags` query param (comma-separated)
  - [ ] Test: search by name, description keyword, tag; verify <500ms response

- [ ] **T2.4 — MCP Schema Parser** (Day 7, parallel with T2.3)
  - [ ] Create `lib/mcp-parser.ts`
    - Accept MCP server URL
    - Send `tools/list` JSON-RPC request to the server
    - Parse response: extract tool names, descriptions, input schemas
    - Return array of `ParsedMcpTool` objects ready for publishing form
  - [ ] Create `POST /api/v1/tools/import-mcp` — trigger MCP import
    - Timeout: 10s for MCP server response
    - Validate URL (HTTPS only, SSRF protection)
  - [ ] Handle errors: connection refused, invalid response, timeout

- [ ] **T2.5 — Tool Publishing Form (UI)** (Day 8-9)
  - [ ] Create `/publish` page with multi-section form:
    - Basic info: name, slug (auto-generated), summary, description (markdown editor)
    - Category: dropdown from categories API
    - Protocol: radio group (MCP Server / REST API / Custom)
    - Endpoint: URL input + method selector (for REST) + auth config
    - MCP import: URL input + "Import" button → auto-fill fields from parsed schema
    - Pricing: CP per call input (1-100 range slider)
    - Tags: tag input with autocomplete
    - Icon: file upload (R2)
  - [ ] Preview panel: show how tool will appear in listing
  - [ ] Form validation with Zod (client-side) + server-side
  - [ ] Submit: POST to `/api/v1/tools/publish`, redirect to tool detail on success
  - [ ] Create `components/publish/publish-form.tsx`, `mcp-import.tsx`, `endpoint-tester.tsx`

- [ ] **T2.6 — Tool Listing Pages (UI)** (Day 9, parallel with T2.5)
  - [ ] Create `/` (home page):
    - Hero section with search bar
    - Category grid (12 categories with icons)
    - Featured tools section (tools where `is_featured=true`)
    - Recent tools section (newest 8)
  - [ ] Create `/search` page:
    - Search bar (pre-filled from query)
    - Sidebar filters: category, protocol, price range
    - Tool card grid with pagination
    - Sort dropdown: Popular, Newest, Price Low/High
    - Empty state for no results
  - [ ] Create `components/tools/tool-card.tsx` — card with icon, name, summary, price, call count
  - [ ] Create `components/tools/tool-grid.tsx` — responsive grid layout
  - [ ] Create `components/tools/category-grid.tsx` — category navigation
  - [ ] Create `components/search/search-bar.tsx` — with debounced input, URL sync
  - [ ] Create `components/search/search-filters.tsx`
  - [ ] Create `components/layout/header.tsx` — nav with search, auth state, dashboard link
  - [ ] Create `components/layout/footer.tsx`

- [ ] **T2.7 — Tool Detail Page (UI)** (Day 10, parallel with T2.5)
  - [ ] Create `/tools/[slug]` page:
    - Tool header: icon, name, publisher, category badge, status
    - Description: rendered markdown
    - Pricing card: CP per call, free tier info, estimated monthly cost
    - Code samples: curl, Python, JavaScript — with copy button
    - API endpoint display: `POST https://toolkit.clawlabz.xyz/api/v1/tools/{slug}/invoke`
    - Request/response schema viewer (if available)
    - Publisher info sidebar: name, avatar, other tools
  - [ ] Create `components/tools/tool-detail.tsx`
  - [ ] Create `components/tools/tool-code-sample.tsx` — tabbed code display with syntax highlighting
  - [ ] Create `components/tools/tool-pricing-card.tsx`
  - [ ] ISR with 60s revalidation

### Checkpoint Criteria
- [ ] Tool CRUD: create, read, update, archive all work via API
- [ ] Search: keyword search returns relevant results in <500ms
- [ ] MCP import: given a running MCP server URL, auto-populates tool fields
- [ ] Home page renders with categories and featured tools
- [ ] Search page filters and paginates correctly
- [ ] Tool detail page shows all info including code samples
- [ ] Publishing form validates and creates tools end-to-end

---

## Phase 3: API Gateway + Billing (Days 11-18)

### Goal
The core value proposition: one API key invokes any tool, with CP billing.

### Tasks

- [ ] **T3.1 — API Key System** (Day 11-12)
  - [ ] Create `lib/api-keys.ts`:
    - `generateApiKey()`: `tk_` + 48 random bytes (Base62) → return full key + SHA-256 hash + prefix
    - `hashApiKey(key)`: SHA-256 hash
    - `validateApiKey(key)`: lookup hash in DB, return user_id + key_id or null
  - [ ] Create `POST /api/v1/keys` — generate new key (max 20 per user)
    - Response includes `full_key` (shown once only)
  - [ ] Create `GET /api/v1/keys` — list user's keys (prefix, name, last_used, created, active status)
  - [ ] Create `DELETE /api/v1/keys/:id` — revoke key (soft: set is_active=false)
  - [ ] Extend `middleware.ts`: extract `Authorization: Bearer tk_xxx`, validate, attach user context
  - [ ] Unit tests: key generation, hashing, validation, revocation

- [ ] **T3.2 — Gateway Proxy Engine** (Day 12-14)
  - [ ] Create `lib/gateway.ts` — core proxy orchestrator:
    ```
    async function invokeGateway(req, toolSlug, apiKeyContext):
      1. lookupTool(slug) — with in-memory cache (60s TTL)
      2. checkFreeTier(userId, toolId) — query toolkit_free_tier_usage
      3. if not free: checkAndDeductCP(userId, tool.pricing_per_call)
      4. proxyRequest(tool, req.body) — delegate to protocol adapter
      5. recordInvocation(metadata) — fire-and-forget
      6. return upstream response
    ```
  - [ ] Create `POST /api/v1/tools/:slug/invoke` route:
    - Auth: API Key only (no session auth)
    - Rate limit: 1000/min per API key (via Upstash, checked in middleware)
    - Pass through request body to gateway engine
    - Return upstream response directly (no envelope wrapping)
    - Add response headers: `X-Toolkit-Request-Id`, `X-Toolkit-Latency-Ms`
  - [ ] Implement in-memory tool cache with Map + TTL
  - [ ] Implement SSRF protection in URL validation (block private IPs, require HTTPS)
  - [ ] Implement response size limit (10MB)
  - [ ] Implement configurable timeout per tool (default 30s, max 60s)

- [ ] **T3.3 — Protocol Adapters** (Day 14-15, parallel after T3.2)
  - [ ] Create `lib/gateway-protocols/rest.ts`:
    - Forward body with tool's HTTP method
    - Inject upstream auth headers from encrypted `auth_config`
    - Forward Content-Type
  - [ ] Create `lib/gateway-protocols/mcp.ts`:
    - Wrap user body in JSON-RPC 2.0 envelope (`tools/call`)
    - Set `mcp_tool_name` from tool config
    - Unwrap JSON-RPC response, extract result or error
  - [ ] Create `lib/gateway-protocols/custom.ts`:
    - Raw passthrough (body + headers)
  - [ ] Create `lib/encryption.ts` — AES-256-GCM encrypt/decrypt for auth_config tokens
  - [ ] Integration test: invoke a mock REST endpoint, invoke a mock MCP server

- [ ] **T3.4 — Free Tier Tracking** (Day 15, parallel after T3.2)
  - [ ] Implement `checkFreeTier(userId, toolId)`:
    - Query `toolkit_free_tier_usage` for current month
    - If no row or count < limit → free
    - Atomic increment: `INSERT ... ON CONFLICT (user_id, tool_id, month) DO UPDATE SET call_count = call_count + 1`
  - [ ] Monthly reset: no cron needed — `month` column naturally partitions by month
  - [ ] Unit test: verify free tier exhaustion, month rollover

- [ ] **T3.5 — CP Billing Integration** (Day 15-16, parallel after T3.2)
  - [ ] Create `lib/billing.ts`:
    - `checkBalance(userId)`: query `shared_point_accounts`
    - `deductCP(userId, amount, ref)`: atomic `UPDATE ... WHERE balance >= amount` + `INSERT shared_point_ledger` with source=`toolkit_invocation`
    - `creditPublisher(publisherId, toolId, amount)`: increment `toolkit_publisher_earnings`
  - [ ] Platform fee: 20% deducted at earnings aggregation time (not per-call)
  - [ ] Handle insufficient balance: return 402 with clear error before proxying
  - [ ] Unit test: deduction, overdraw prevention, ledger entry creation

- [ ] **T3.6 — Invocation Recording** (Day 16, parallel after T3.2)
  - [ ] Create `lib/metering.ts`:
    - `recordInvocation(data)`: fire-and-forget INSERT into `toolkit_invocations`
    - `incrementToolStats(toolId)`: update `toolkit_tools.total_calls` (fire-and-forget)
    - `getUsageStats(userId, period)`: aggregate invocations for dashboard
    - `getPublisherStats(publisherId, period)`: aggregate by tool for publisher dashboard
  - [ ] Ensure async recording does not block response
  - [ ] Unit test: verify invocation records created with correct fields

- [ ] **T3.7 — Stripe Top-up Flow** (Day 13-14, parallel with T3.2)
  - [ ] Create `lib/stripe.ts` — Stripe client singleton
  - [ ] Create `POST /api/v1/billing/topup`:
    - Input: `amount_usd` (min $5, max $500)
    - Create Stripe Checkout Session (mode: payment)
    - Line item: "{amount_usd * 1000} CP — ClawToolkit Credits"
    - Success URL: `/dashboard?topup=success`
    - Cancel URL: `/dashboard/topup?cancelled=true`
    - Metadata: `user_id`, `cp_amount`
    - Return: `{ checkout_url }`
  - [ ] Create `POST /api/v1/billing/webhook`:
    - Verify Stripe signature
    - Check idempotency: lookup in `toolkit_stripe_events`
    - Handle `checkout.session.completed`:
      - Extract `user_id` and `cp_amount` from metadata
      - Credit CP to `shared_point_accounts` + insert `shared_point_ledger` (source=`stripe_topup`)
    - Return 200 immediately
  - [ ] Create `GET /api/v1/billing/balance` — return current CP balance
  - [ ] Test: Stripe test mode end-to-end (checkout → webhook → balance increase)

- [ ] **T3.8 — Stripe Connect Publisher Payout** (Day 16-17, parallel with T3.7)
  - [ ] Publisher onboarding: create Stripe Connect Express account
    - Triggered when user first accesses publisher dashboard
    - Store `stripe_connect_id` on `toolkit_users`
  - [ ] Create `POST /api/v1/dashboard/publisher/withdraw`:
    - Input: `amount_cp` (min 10,000 CP)
    - Convert CP to USD (1000 CP = $1)
    - Verify `toolkit_publisher_earnings` has sufficient unsettled net earnings
    - Create Stripe Transfer to Connect account
    - Mark earnings as settled
  - [ ] Test: Stripe Connect test mode

### Checkpoint Criteria
- [ ] API Key: generate, list, revoke all work; invalid key returns 401
- [ ] Gateway: `POST /api/v1/tools/{slug}/invoke` proxies to upstream and returns response
- [ ] REST adapter: proxies to a test REST endpoint correctly
- [ ] MCP adapter: wraps/unwraps JSON-RPC correctly
- [ ] Free tier: first 100 calls free, 101st deducts CP
- [ ] CP billing: insufficient balance returns 402; successful deduction logs to ledger
- [ ] Stripe top-up: Checkout flow → webhook → CP credited
- [ ] Gateway p95 overhead < 100ms (measured against local mock endpoint)
- [ ] Invocation records created in DB with correct latency/status/cost

---

## Phase 4: Dashboards + Polish (Days 19-24)

### Goal
All dashboard UIs, settings, documentation, landing page, and error handling.

### Tasks

- [ ] **T4.1 — Developer Dashboard (UI)** (Day 19-20)
  - [ ] Create `/dashboard` page layout with sidebar navigation:
    - Overview (default), Publisher, Top Up, Settings
  - [ ] Overview section:
    - Summary cards: total calls (this month), total CP spent, CP balance, active API keys
    - Usage chart: line chart of daily calls (last 30 days) using recharts (lazy-loaded)
    - Cost breakdown: bar chart by tool
    - Tool-by-tool table: tool name, calls, CP spent, avg latency, last used
  - [ ] Create `components/dashboard/usage-chart.tsx`
  - [ ] Create `components/dashboard/cost-breakdown.tsx`
  - [ ] Create `hooks/use-usage.ts` — SWR/React Query hook for usage data
  - [ ] Period selector: 7d / 30d / 90d toggle

- [ ] **T4.2 — Publisher Dashboard (UI)** (Day 20-21)
  - [ ] Create `/dashboard/publisher` page:
    - Summary cards: total tool calls, gross CP earned, net CP earned, pending withdrawal
    - Revenue chart: line chart of daily revenue (last 30 days)
    - Tool table: each published tool with calls, revenue, avg latency, status
    - Withdraw button: opens modal, amount input, triggers `/dashboard/publisher/withdraw`
    - Stripe Connect onboarding: if no `stripe_connect_id`, show "Set up payouts" CTA
  - [ ] Create `components/dashboard/revenue-chart.tsx`
  - [ ] Create `components/dashboard/tool-stats-table.tsx`
  - [ ] Create `hooks/use-publisher-stats.ts`

- [ ] **T4.3 — API Key Management (UI)** (Day 20, parallel with T4.1)
  - [ ] Integrate into `/dashboard` page (tab or section):
    - List existing keys: name, prefix, created, last used, status
    - Create key button → modal → show full key once (with copy button + warning)
    - Revoke button per key (with confirmation dialog)
  - [ ] Create `components/dashboard/api-key-manager.tsx`
  - [ ] Create `hooks/use-api-keys.ts`

- [ ] **T4.4 — Top-up Page (UI)** (Day 21, parallel with T4.2)
  - [ ] Create `/dashboard/topup` page:
    - Current balance display
    - Amount selector: preset buttons ($5, $10, $25, $50, $100) + custom input
    - CP conversion display (e.g., "$10 = 10,000 CP")
    - "Pay with Stripe" button → redirect to Stripe Checkout
    - Success/cancelled states (from URL params)
  - [ ] Create `components/billing/amount-selector.tsx`

- [ ] **T4.5 — Settings Page (UI)** (Day 21, parallel)
  - [ ] Create `/settings` page:
    - Profile section: name, email (read-only for OAuth), avatar URL
    - Update profile: PATCH `/api/v1/users/me`
    - Account info: registration date, role, GitHub connection status
    - Danger zone: (placeholder for account deletion — P1)

- [ ] **T4.6 — API Documentation Page** (Day 22)
  - [ ] Create `/docs` page:
    - Authentication section: API key usage, header format
    - Gateway invocation: endpoint, request format, response format
    - Code examples: curl, Python (requests), JavaScript (fetch), TypeScript
    - Error codes reference table
    - Rate limits documentation
    - Free tier explanation
    - CP billing explanation
  - [ ] Static content (SSG), rendered from markdown or JSX

- [ ] **T4.7 — Landing Page Polish** (Day 22, parallel with T4.6)
  - [ ] Polish `/` page:
    - Hero: headline, subheadline, CTA buttons ("Browse Tools", "Publish a Tool")
    - How it works: 3-step illustration (Discover → Connect → Pay-per-use)
    - Stats section: tool count, total invocations (from DB, ISR)
    - For Developers section: key benefits
    - For Publishers section: key benefits
    - Footer: links, ClawLabz branding
  - [ ] Meta tags, OG image, favicon
  - [ ] Mobile responsive (320px-2560px)

- [ ] **T4.8 — Error Handling + Edge Cases** (Day 23-24)
  - [ ] Create `error.tsx` — global error boundary
  - [ ] Create `not-found.tsx` — 404 page
  - [ ] Loading states: skeleton loaders for all data-fetching pages
  - [ ] Empty states: no tools, no invocations, no API keys
  - [ ] Toast notifications: success/error feedback (shadcn/ui toast)
  - [ ] Form validation errors: inline error messages
  - [ ] Graceful degradation: if Stripe/Redis unavailable, show appropriate errors
  - [ ] CSP headers in `next.config.ts`
  - [ ] Security headers: X-Frame-Options, X-Content-Type-Options, Referrer-Policy

### Checkpoint Criteria
- [ ] Developer dashboard shows real usage data with charts
- [ ] Publisher dashboard shows earnings and supports withdrawal
- [ ] API key management: create, view, revoke all work in UI
- [ ] Top-up flow: select amount → Stripe Checkout → return → balance updated
- [ ] Settings page: update profile name saves correctly
- [ ] Docs page: comprehensive API reference renders correctly
- [ ] Landing page: polished, responsive, all sections render
- [ ] All pages handle loading, empty, and error states
- [ ] No console errors or warnings in production build
- [ ] Lighthouse score > 90 (Performance, Accessibility)

---

## Phase 5: Testing + Deploy (Days 25-28)

### Goal
Comprehensive testing, security hardening, and production deployment.

### Tasks

- [ ] **T5.1 — Unit Tests** (ongoing throughout, finalize Day 25)
  - [ ] `lib/api-keys.ts` — key generation, hashing, validation
  - [ ] `lib/billing.ts` — CP deduction, overdraw prevention, ledger
  - [ ] `lib/gateway.ts` — proxy orchestration logic (mocked DB)
  - [ ] `lib/gateway-protocols/*.ts` — each adapter with mock upstream
  - [ ] `lib/mcp-parser.ts` — schema parsing with fixture data
  - [ ] `lib/rate-limiter.ts` — window enforcement
  - [ ] `lib/search.ts` — query sanitization
  - [ ] `lib/encryption.ts` — encrypt/decrypt round-trip
  - [ ] `lib/validators/*.ts` — Zod schema validation
  - [ ] Target: 80%+ code coverage on `lib/`

- [ ] **T5.2 — Integration Tests** (Day 25-26)
  - [ ] Auth flow: signup → login → session → protected route
  - [ ] Tool lifecycle: publish → list → search → detail → update → archive
  - [ ] Gateway flow: create key → invoke tool → verify CP deducted → verify invocation recorded
  - [ ] Free tier flow: invoke 100 times free → 101st charges CP
  - [ ] Stripe flow: create checkout → simulate webhook → verify CP credited
  - [ ] Rate limiting: exceed limit → verify 429 response
  - [ ] SSRF protection: attempt invoke with private IP endpoint → verify rejected

- [ ] **T5.3 — E2E Smoke Tests** (Day 26-27)
  - [ ] Define smoke test script (similar to `pnpm smoke:arena`):
    - Health check endpoint returns 200
    - Home page renders with categories
    - Search returns results for seeded tools
    - Tool detail page renders
    - Auth: signup, login, logout
    - Create API key, invoke a test tool, verify response
    - Dashboard loads with data
  - [ ] Add `pnpm smoke:toolkit` script

- [ ] **T5.4 — Security Audit** (Day 27)
  - [ ] Verify all API routes validate input with Zod
  - [ ] Verify API keys stored as SHA-256 hash only
  - [ ] Verify upstream auth tokens encrypted at rest
  - [ ] Verify SSRF protection blocks private IPs
  - [ ] Verify rate limiting active on all endpoint groups
  - [ ] Verify RLS policies enforced (test cross-user access attempts)
  - [ ] Verify Stripe webhook signature validation
  - [ ] Verify CSP + security headers present
  - [ ] Verify no secrets in client bundle (check build output)
  - [ ] Verify request/response bodies not logged by gateway

- [ ] **T5.5 — Performance Testing** (Day 27, parallel with T5.4)
  - [ ] Gateway latency test: 100 sequential invocations against mock upstream
    - Measure: p50, p95, p99 gateway overhead
    - Target: p95 < 100ms
  - [ ] Search performance: 50 concurrent search queries
    - Target: p95 < 500ms
  - [ ] Page load: Lighthouse audit on key pages
    - Target: Performance > 90, <2s LCP
  - [ ] Bundle size check: verify <150KB shared JS

- [ ] **T5.6 — Vercel Deployment** (Day 28)
  - [ ] Configure Vercel project for `apps/toolkit/`
  - [ ] Set all environment variables in Vercel dashboard
  - [ ] Configure build settings: root directory, build command, output directory
  - [ ] Deploy to preview branch, run smoke tests against preview URL
  - [ ] Configure Stripe webhook URL to production endpoint
  - [ ] Configure Supabase redirect URLs for OAuth
  - [ ] Deploy to production

- [ ] **T5.7 — DNS + Domain Setup** (Day 28, parallel with T5.6)
  - [ ] Add `toolkit.clawlabz.xyz` CNAME to Cloudflare DNS → Vercel
  - [ ] Configure custom domain in Vercel
  - [ ] Verify SSL certificate provisioned
  - [ ] Verify R2 custom domain for toolkit assets

- [ ] **T5.8 — Seed Data + Go-Live** (Day 28, after T5.6 + T5.7)
  - [ ] Seed 5-10 demo tools (mix of MCP and REST) for launch showcase
  - [ ] Verify all pages work on production domain
  - [ ] Verify Stripe live mode (if ready) or keep in test mode for soft launch
  - [ ] Run `pnpm smoke:toolkit` against production
  - [ ] Final manual walkthrough: signup → browse → publish → invoke → dashboard

### Checkpoint Criteria (Go-Live)
- [ ] Unit test coverage >=80% on `lib/`
- [ ] All integration tests pass
- [ ] E2E smoke test passes on production URL
- [ ] Security audit checklist fully green
- [ ] Gateway p95 < 100ms confirmed
- [ ] All pages load < 2s on simulated 3G
- [ ] `toolkit.clawlabz.xyz` resolves and serves the app over HTTPS
- [ ] At least 5 demo tools browsable and invocable
- [ ] Stripe Checkout flow works end-to-end (test or live mode)
- [ ] No CRITICAL or HIGH severity issues open

---

## Parallelism Summary

| Day(s) | Parallel Tracks |
|--------|----------------|
| 1 | T1.1 |
| 1-2 | T1.2 |
| 2 | T1.3 |
| 2-3 | T1.4 |
| 3 | T1.5 (parallel) + T1.6 (parallel) |
| 5-6 | T2.1 + T2.2 (parallel) |
| 6-7 | T2.3 + T2.4 (parallel) |
| 8-10 | T2.5 + T2.6 + T2.7 (parallel) |
| 11-12 | T3.1 + T3.7 (parallel) |
| 12-14 | T3.2 + T3.8 (parallel with T3.7) |
| 14-16 | T3.3 + T3.4 + T3.5 + T3.6 (all parallel after T3.2) |
| 19-21 | T4.1 + T4.2 + T4.3 + T4.4 + T4.5 (all parallel) |
| 22 | T4.6 + T4.7 (parallel) |
| 23-24 | T4.8 |
| 25-26 | T5.1 + T5.2 |
| 26-27 | T5.3 + T5.4 + T5.5 (parallel) |
| 28 | T5.6 + T5.7 (parallel) → T5.8 (after both) |

---

## Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Gateway p95 > 100ms on Vercel cold starts | Medium | High | Keep gateway function minimal imports; measure early in Phase 3; fallback: Vercel edge function |
| MCP server ecosystem fragile — servers may not respond to `tools/list` | Medium | Medium | Graceful error handling; manual fallback for tool field entry; validate during import |
| Stripe Connect onboarding friction for publishers | Low | Medium | Provide clear documentation; defer mandatory onboarding until first withdrawal |
| Free tier abuse — bots creating accounts for free calls | Medium | Medium | Rate limit per IP on signup; CAPTCHA consideration for P1; monitor registration patterns |
| Supabase connection pool exhaustion under load | Low | High | Connection pooling via PgBouncer (built-in); fire-and-forget pattern reduces connection hold time |
