# Clarity — Analytics Provider

## Configuration

```yaml
optional:
  analytics:
    clarity:
      enabled: true
      project_id: "xxxxxxxxxx"
```

## How It Works

Uses the `ga4-clarity-auto-integration` skill to inject the Microsoft Clarity tracking script.

Clarity provides session recordings, heatmaps, and user behavior analytics — complementary to GA4's quantitative metrics.

## Required Credentials

| Credential | Where to Get |
|-----------|-------------|
| Clarity Project ID | clarity.microsoft.com → Settings → Project ID |

## Notes

- If `project_id` is empty, Clarity setup is skipped with a note in the deploy report
- Clarity is free with no traffic limits
- Provides session recordings, heatmaps, and rage click detection
- GDPR considerations: Clarity auto-masks sensitive content
