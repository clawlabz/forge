# Provider Development Guide

Forge uses a **pluggable provider architecture**. Every external integration (hosting, database, DNS, design tools, analytics) is abstracted behind a provider interface, making it easy to add support for new platforms.

## How Providers Work

Each provider handles a specific concern in the Forge pipeline:

| Concern | Interface | Default Provider |
|---------|-----------|-----------------|
| `doc_hosting` | Push documents, generate shareable URLs | GitHub |
| `deployment` | Deploy application to hosting platform | Vercel |
| `database` | Run migrations, manage schema | Supabase |
| `dns` | Create/update DNS records | Cloudflare |
| `storage` | Upload/manage static files | Cloudflare R2 |
| `design` | Generate or import design specifications | Manual Spec |
| `testing` | Run E2E tests in browser | Playwright |
| `analytics` | Inject tracking scripts | GA4 + Clarity |

## Provider Resolution Strategy

Providers support a dual-layer strategy configured in `forge.config.yaml`:

```yaml
providers:
  deployment:
    type: "vercel"
    strategy: "mcp_first"          # Try MCP server first, fall back to script
    mcp: "vercel-mcp-server"       # MCP server name
    fallback_script: "scripts/vercel-deploy.sh"  # Fallback script path
```

**Strategies:**
- `mcp_first` — Try MCP server, if unavailable/fails use script
- `script_first` — Use script primarily, MCP as enhancement
- `mcp_only` — MCP server only, fail if unavailable
- `script_only` — Script only, no MCP

## Adding a New Provider

### 1. Create Provider Documentation

Create a file at `providers/<concern>/<provider-name>.md`:

```markdown
# [Provider Name] — [Concern] Provider

## Configuration

```yaml
providers:
  <concern>:
    type: "<provider-name>"
    config:
      <key>: <description>
```

## Required Credentials

| Credential | Environment Variable | Description |
|-----------|---------------------|-------------|

## Capabilities

| Action | Supported | Notes |
|--------|-----------|-------|

## Implementation Notes

[How the skill should interact with this provider]
```

### 2. Update the Skill

Each phase skill checks `forge.config.yaml` for the provider type and adapts its behavior:

```
Read forge.config.yaml
→ Get provider type for this concern
→ If type matches known provider → use that provider's logic
→ If unknown → warn user, suggest configuring a known provider
```

### 3. Register in Templates

Update `templates/forge.config.yaml` to include the new provider as a comment option.

## Current Provider Implementations

### Doc Hosting

| Provider | Status | How It Works |
|----------|--------|-------------|
| **GitHub** | ✅ Default | Push to repo, share blob URL |
| Notion | 🔮 Planned | Create page via Notion API |
| Cloudflare Pages | 🔮 Planned | Deploy markdown as static site |

### Deployment

| Provider | Status | How It Works |
|----------|--------|-------------|
| **Vercel** | ✅ Default | MCP or `vercel` CLI |
| Cloudflare Pages | 🔮 Planned | MCP or `wrangler` CLI |
| Netlify | 🔮 Planned | `netlify` CLI |

### Database

| Provider | Status | How It Works |
|----------|--------|-------------|
| **Supabase** | ✅ Default | MCP or Management API |
| PlanetScale | 🔮 Planned | `pscale` CLI |
| Neon | 🔮 Planned | Neon API |

### DNS

| Provider | Status | How It Works |
|----------|--------|-------------|
| **Cloudflare** | ✅ Default | MCP or API |
| Vercel DNS | 🔮 Planned | Vercel API |
| Route53 | 🔮 Planned | AWS CLI |

### Design

| Provider | Status | How It Works |
|----------|--------|-------------|
| **Manual Spec** | ✅ Default | Design spec markdown document |
| Figma | 🔮 Planned | Figma MCP Server |
| Stitch | 🔮 Planned | Stitch API |

### Analytics

| Provider | Status | How It Works |
|----------|--------|-------------|
| **GA4** | ✅ Default | Script injection via skill |
| **Clarity** | ✅ Default | Script injection via skill |
