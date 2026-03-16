# Vercel — Deployment Provider

## Configuration

```yaml
providers:
  deployment:
    type: "vercel"
    strategy: "mcp_first"
    mcp: "vercel-mcp-server"
    fallback_script: "scripts/vercel-deploy.sh"
    config:
      team: ""          # Vercel team slug (optional, for org accounts)
```

## Capabilities

| Action | MCP | CLI/Script |
|--------|-----|-----------|
| Link project | ✅ | ✅ |
| Set env vars | ✅ | ✅ |
| Deploy to production | ✅ | ✅ |
| Add custom domain | ✅ | ✅ |
| Check deployment status | ✅ | ✅ |
| Rollback | ✅ | ✅ |

## How It Works

### MCP Path (preferred)
Uses the official Vercel MCP Server with OAuth 2.1 authentication.

### CLI/Script Path (fallback)
```bash
vercel link
vercel env add <key> production < <(echo "<value>")
vercel --prod
vercel domains add <domain>
```

## Required Credentials

| Credential | Env Variable | How to Get |
|-----------|-------------|------------|
| Vercel Token | `VERCEL_TOKEN` | vercel.com/account/tokens |

## Notes

- Vercel automatically provisions SSL for custom domains
- First deploy creates the project on Vercel
- Environment variables must be set before production deploy
- Build command and output directory are auto-detected from framework
