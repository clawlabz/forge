# Supabase — Database Provider

## Configuration

```yaml
providers:
  database:
    type: "supabase"
    strategy: "script_first"
    mcp: "supabase-mcp-server"
    fallback_script: "scripts/supabase-migrate.sh"
    config:
      project_ref: ""   # Supabase project reference ID
```

## Capabilities

| Action | MCP | Script |
|--------|-----|--------|
| Run migrations | ⚠️ Limited | ✅ |
| Read schema | ✅ | ✅ |
| Manage RLS policies | ⚠️ Limited | ✅ |
| Query data | ✅ | ✅ |

## How It Works

### Script Path (preferred for migrations)
Uses the Supabase Management API to apply SQL migrations:
```bash
./scripts/supabase-migrate.sh <migration-file>
```

### MCP Path (for read operations)
Uses the official Supabase MCP Server for schema inspection and data queries.

## Required Credentials

| Credential | Env Variable | How to Get |
|-----------|-------------|------------|
| Supabase URL | `SUPABASE_URL` | Project Settings → API |
| Service Role Key | `SUPABASE_SERVICE_ROLE_KEY` | Project Settings → API |

## Notes

- Migrations are SQL files in `supabase/migrations/` directory
- Migration filenames should be numbered: `001_initial.sql`, `002_add_users.sql`
- Service role key bypasses RLS — use only server-side
- Script path is preferred for migrations as MCP has limited write capabilities
