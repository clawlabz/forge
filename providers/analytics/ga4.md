# GA4 — Analytics Provider

## Configuration

```yaml
optional:
  analytics:
    ga4:
      enabled: true
      measurement_id: "G-XXXXXXXXXX"
```

## How It Works

Uses the `ga4-clarity-auto-integration` skill to automatically inject the GA4 tracking script into the application.

For Next.js apps, this typically means adding the Google Analytics script to the root layout or a dedicated analytics component.

## Required Credentials

| Credential | Where to Get |
|-----------|-------------|
| GA4 Measurement ID | Google Analytics → Admin → Data Streams → Measurement ID |

## Notes

- If `measurement_id` is empty, GA4 setup is skipped with a note in the deploy report
- The skill handles framework-specific injection (Next.js, React SPA, etc.)
- Verification: check Network tab for requests to `google-analytics.com`
