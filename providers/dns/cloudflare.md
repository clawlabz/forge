# Cloudflare — DNS Provider

## Configuration

```yaml
providers:
  dns:
    type: "cloudflare"
    strategy: "mcp_first"
    mcp: "cloudflare-mcp-server"
    fallback_script: "scripts/cloudflare-dns.sh"
    config:
      zone: ""          # Domain zone (e.g., "example.com")
```

## Capabilities

| Action | MCP | Script |
|--------|-----|--------|
| Add CNAME record | ✅ | ✅ |
| Add A record | ✅ | ✅ |
| List records | ✅ | ✅ |
| Delete record | ✅ | ✅ |
| Toggle proxy | ✅ | ✅ |

## How It Works

### Script Path
```bash
./scripts/cloudflare-dns.sh add <subdomain> <target> [type] [proxied]
./scripts/cloudflare-dns.sh list [filter]
./scripts/cloudflare-dns.sh delete <subdomain>
```

### MCP Path
Uses the official Cloudflare MCP Server for DNS management.

## Required Credentials

| Credential | Env Variable | How to Get |
|-----------|-------------|------------|
| API Token | `CLOUDFLARE_API_TOKEN` | dash.cloudflare.com → API Tokens |
| Zone ID | `CLOUDFLARE_ZONE_ID` | dash.cloudflare.com → Zone Overview |

## Notes

- CNAME records for Vercel should point to `cname.vercel-dns.com`
- Proxy (orange cloud) provides DDoS protection and caching
- DNS propagation typically takes < 5 minutes with Cloudflare
- Wildcard records require a paid plan
