# Manual Spec — Design Provider

## Configuration

```yaml
providers:
  design:
    type: "manual_spec"
    config: {}
```

## Description

The default design provider. Instead of generating visual mockups in an external tool, the P3 Design phase produces a comprehensive markdown specification document that serves as the design deliverable.

## What It Produces

`docs/forge/P3-design-spec.md` containing:
- Color system with hex values
- Typography scale
- Spacing and grid system
- Component specifications with Tailwind classes
- Page layout descriptions (ASCII wireframes)
- Responsive behavior notes
- Animation and transition specs
- Dark mode specifications
- Accessibility checklist

## When to Use

- Default for all projects
- When no external design tool access is available
- When the design spec document is sufficient for implementation
- When speed is prioritized over pixel-perfect mockups

## Alternatives

- **Figma** (`figma`) — Requires Figma MCP Server and token. Generates or reads from Figma files.
- **Stitch** (`stitch`) — Requires Stitch API key. Generates design mockups via AI.

## Required Credentials

None.
