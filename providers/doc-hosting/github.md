# GitHub — Doc Hosting Provider

## Configuration

```yaml
providers:
  doc_hosting:
    type: "github"
    config:
      repo: ""          # Auto-detected from git remote, or set manually (org/repo)
      branch: "main"    # Branch to push documents to
```

## Capabilities

| Action | Supported |
|--------|-----------|
| Push document | ✅ |
| Generate shareable URL | ✅ |
| View on mobile | ✅ (GitHub mobile app or browser) |
| Markdown rendering | ✅ (native) |
| Comments/review | ✅ (via PR or issue) |
| Version history | ✅ (git log) |

## How It Works

### Push Document
```bash
git add <document-path>
git commit -m "docs: <description>"
git push origin <branch>
```

### Generate URL
```
https://github.com/<org>/<repo>/blob/<branch>/<document-path>
```

## Required Credentials

| Credential | Env Variable | How to Get |
|-----------|-------------|------------|
| GitHub Token | `GITHUB_TOKEN` or `gh` CLI auth | `gh auth login` |

## Notes

- GitHub renders markdown natively with good mobile support
- Private repos require GitHub login to view
- For public repos, anyone with the link can view
- Mermaid diagrams are rendered automatically by GitHub
