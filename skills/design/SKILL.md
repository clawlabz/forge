# /forge:design — P3 Visual Design System & Page Layouts

## Overview

Create a complete visual design system and page-by-page layout specification. This phase runs in parallel with P2 (Architect).

## Usage

```
/forge:design
```

## Prerequisites

- P1 PRD must be approved (`docs/forge/P1-prd.md`)

## Process

### Step 1: Read Context

Read these files:
- `docs/forge/P1-prd.md` — approved PRD (page list, features, user personas)
- `docs/forge/P0-brainstorm-report.md` — competitor analysis (for design differentiation)
- `forge.config.yaml` — design provider configuration

### Step 2: Design Direction Exploration

Use `ui-ux-pro-max` skill to explore design directions:

1. **Analyze the product type** — Is it a dashboard? A consumer app? A marketplace? A tool?
2. **Identify the emotional tone** — Professional, playful, minimal, bold, warm, technical?
3. **Study competitor designs** — What works? What can we do differently?
4. **Propose 2-3 style directions** with mood descriptions:
   - Direction A: [description + why it fits]
   - Direction B: [description + why it fits]
   - Direction C: [description + why it fits]
5. **Recommend one** with rationale

### Step 3: Design System Definition

Using the chosen direction, define the complete design system:

**Color System:**
- Primary, Secondary, Accent colors (with hex values)
- Semantic colors: success, warning, error, info
- Neutral scale (gray-50 through gray-900)
- Dark mode variant colors
- Ensure WCAG AA contrast ratios

**Typography:**
- Font families (heading + body, from Google Fonts or system fonts)
- Type scale (text-xs through text-5xl with rem values)
- Font weights used
- Line heights

**Spacing & Layout:**
- Spacing scale (4px base: 1, 2, 3, 4, 6, 8, 12, 16, 24, 32, 48, 64)
- Grid system (12-column, max-width, gutters)
- Responsive breakpoints (sm: 640, md: 768, lg: 1024, xl: 1280)

**Border & Shadow:**
- Border radius scale
- Shadow system (sm, md, lg, xl)

### Step 4: Component Specifications

Define core UI components with their variants and states:

| Component | Variants | States |
|-----------|----------|--------|
| Button | primary, secondary, outline, ghost, destructive | default, hover, active, disabled, loading |
| Input | text, password, textarea, select | default, focus, error, disabled |
| Card | default, interactive, featured | default, hover |
| Modal | default, drawer, fullscreen | open, closing |
| Navigation | top bar, sidebar, bottom (mobile) | expanded, collapsed |
| Table | default, compact, striped | sortable, selectable |
| Toast | success, error, warning, info | entering, visible, exiting |
| Badge | default, outline | various colors |
| Avatar | image, initials, icon | xs, sm, md, lg |
| Skeleton | text, card, image | loading |

For each component, specify:
- Tailwind CSS class composition
- Responsive behavior
- Accessibility attributes (ARIA)

### Step 5: Page Layout Design

For every page listed in the PRD, design:

1. **Layout structure** — Header/sidebar/main/footer arrangement
2. **Content hierarchy** — What's most important, visual flow
3. **Key components used** — Which design system components appear
4. **Responsive behavior** — How it adapts at each breakpoint
5. **Interactive states** — Loading, empty, error states
6. **Micro-interactions** — Hover effects, transitions, animations

Use `frontend-design` skill for high-quality component/page descriptions.

### Step 6: Design Provider Integration (if configured)

If `forge.config.yaml` design provider is not `manual_spec`:

- **Figma**: Use Figma MCP to generate or export design specifications
- **Stitch**: Use Stitch API to generate design mockups

If `manual_spec` (default): The design spec document IS the deliverable.

### Step 7: Self-Review

Review the design system for:
- [ ] Consistency — All components follow the same visual language
- [ ] Accessibility — WCAG AA compliance, focus states, screen reader support
- [ ] Responsiveness — Every page works at all breakpoints
- [ ] Dark mode — All colors have dark mode variants
- [ ] Component completeness — Every page's needs are covered by the component library
- [ ] Brand differentiation — Doesn't look like a generic template

### Step 8: Write Design Spec

Write `docs/forge/P3-design-spec.md`:

```markdown
# Design Specification: [Product Name]

> Version: 1.0
> Date: [date]
> Status: Pending Approval
> Based on: P1 PRD

## 1. Design Direction
### Style: [chosen direction name]
### Emotional Tone
### Design Principles

## 2. Color System
### Primary Palette
| Name | Light Mode | Dark Mode | Usage |
### Semantic Colors
| Name | Color | Usage |
### Neutral Scale

## 3. Typography
### Font Families
### Type Scale
| Name | Size | Weight | Line Height | Usage |
### Font Pairings

## 4. Spacing & Layout
### Spacing Scale
### Grid System
### Responsive Breakpoints

## 5. Border & Shadow
### Border Radius
### Shadow System

## 6. Component Library
### Button
(variants, states, Tailwind classes, accessibility)
### Input
...
### [Each Component]
...

## 7. Page Designs
### 7.1 [Page Name]
#### Layout (ASCII wireframe or description)
#### Content Hierarchy
#### Components Used
#### Responsive Notes
#### States (loading, empty, error)

### 7.2 [Page Name]
...

## 8. Animation & Transitions
### Page Transitions
### Micro-interactions
### Loading States

## 9. Accessibility Checklist
- [ ] Color contrast AA compliant
- [ ] Focus indicators visible
- [ ] Alt text for images
- [ ] Keyboard navigation
- [ ] Screen reader labels
- [ ] Reduced motion support

## 10. Dark Mode
### Color Mapping
### Component Adjustments

## 11. Self-Review Log
```

### Step 9: Push & Request Approval

1. Git add, commit, push
2. Output:
   ```
   🔴 审批门控

   请审批设计方案: https://github.com/<org>/<repo>/blob/main/docs/forge/P3-design-spec.md

   回复 "approved" 继续，或提出修改意见。
   ```

## Skills & Tools Used

- `ui-ux-pro-max` — 50+ styles, 21 color palettes, 50+ font pairings
- `frontend-design` — high-quality UI component design
- `everything-claude-code:frontend-patterns` — frontend design patterns
- Figma MCP Server (optional) — design file integration
- `design-reviewer` agent — self-review

## Important

- Design must cover EVERY page listed in the PRD
- Components must be implementable with Tailwind CSS
- Dark mode is NOT optional — always include it
- Mobile-first: design mobile layout first, then scale up
- Avoid generic "template" aesthetics — create distinctive design
