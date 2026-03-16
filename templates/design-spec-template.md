# Design Specification: [Product Name]

> Version: 1.0
> Date: [date]
> Status: Pending Approval
> Based on: P1 PRD

## 1. Design Direction

### Style
[Chosen style direction name and description]

### Emotional Tone
[Professional / Playful / Minimal / Bold / Warm / Technical]

### Design Principles
1. [Principle 1] — [explanation]
2. [Principle 2] — [explanation]
3. [Principle 3] — [explanation]

## 2. Color System

### Primary Palette

| Name | Light Mode | Dark Mode | Usage |
|------|-----------|-----------|-------|
| Primary | `#` | `#` | CTAs, key actions |
| Secondary | `#` | `#` | Secondary elements |
| Accent | `#` | `#` | Highlights, links |

### Semantic Colors

| Name | Color | Usage |
|------|-------|-------|
| Success | `#22c55e` | Confirmations, positive status |
| Warning | `#f59e0b` | Caution, attention needed |
| Error | `#ef4444` | Errors, destructive actions |
| Info | `#3b82f6` | Information, tips |

### Neutral Scale

| Name | Value | Usage |
|------|-------|-------|
| gray-50 | `#f9fafb` | Background (light) |
| gray-100 | `#f3f4f6` | Subtle background |
| gray-200 | `#e5e7eb` | Borders |
| gray-300 | `#d1d5db` | Disabled |
| gray-400 | `#9ca3af` | Placeholder text |
| gray-500 | `#6b7280` | Secondary text |
| gray-600 | `#4b5563` | Body text |
| gray-700 | `#374151` | Headings |
| gray-800 | `#1f2937` | Background (dark) |
| gray-900 | `#111827` | Dark surface |

## 3. Typography

### Font Families
- **Headings**: [Font Name] (Google Fonts / System)
- **Body**: [Font Name] (Google Fonts / System)
- **Monospace**: [Font Name] (for code blocks)

### Type Scale

| Name | Size | Weight | Line Height | Usage |
|------|------|--------|-------------|-------|
| text-xs | 0.75rem | 400 | 1rem | Labels, captions |
| text-sm | 0.875rem | 400 | 1.25rem | Secondary text |
| text-base | 1rem | 400 | 1.5rem | Body text |
| text-lg | 1.125rem | 500 | 1.75rem | Large body |
| text-xl | 1.25rem | 600 | 1.75rem | Section headers |
| text-2xl | 1.5rem | 700 | 2rem | Page subtitles |
| text-3xl | 1.875rem | 700 | 2.25rem | Page titles |
| text-4xl | 2.25rem | 800 | 2.5rem | Hero text |

## 4. Spacing & Layout

### Spacing Scale (base: 4px)
`1(4px) 2(8px) 3(12px) 4(16px) 5(20px) 6(24px) 8(32px) 10(40px) 12(48px) 16(64px) 20(80px) 24(96px)`

### Grid System
- Columns: 12
- Max width: 1280px
- Gutter: 24px (desktop), 16px (mobile)
- Margin: 24px (desktop), 16px (mobile)

### Responsive Breakpoints

| Name | Width | Typical Devices |
|------|-------|----------------|
| sm | 640px | Large phones |
| md | 768px | Tablets |
| lg | 1024px | Small laptops |
| xl | 1280px | Desktops |
| 2xl | 1536px | Large screens |

## 5. Border & Shadow

### Border Radius

| Name | Value | Usage |
|------|-------|-------|
| rounded-sm | 0.125rem | Subtle rounding |
| rounded | 0.25rem | Default |
| rounded-md | 0.375rem | Cards, inputs |
| rounded-lg | 0.5rem | Modals, large cards |
| rounded-xl | 0.75rem | Featured elements |
| rounded-full | 9999px | Avatars, pills |

### Shadow System

| Name | Value | Usage |
|------|-------|-------|
| shadow-sm | `0 1px 2px rgba(0,0,0,0.05)` | Subtle depth |
| shadow | `0 1px 3px rgba(0,0,0,0.1)` | Default cards |
| shadow-md | `0 4px 6px rgba(0,0,0,0.1)` | Elevated cards |
| shadow-lg | `0 10px 15px rgba(0,0,0,0.1)` | Modals, dropdowns |

## 6. Component Library

### Button

**Variants:**
| Variant | Classes | Usage |
|---------|---------|-------|
| Primary | `bg-primary text-white hover:bg-primary/90` | Main CTAs |
| Secondary | `bg-secondary text-secondary-foreground` | Secondary actions |
| Outline | `border border-input bg-transparent hover:bg-accent` | Tertiary actions |
| Ghost | `hover:bg-accent hover:text-accent-foreground` | Subtle actions |
| Destructive | `bg-destructive text-destructive-foreground` | Delete, remove |

**Sizes:** sm (h-8 px-3 text-xs), md (h-10 px-4 text-sm), lg (h-12 px-6 text-base)
**States:** default, hover, active, disabled (opacity-50 cursor-not-allowed), loading (spinner)
**Accessibility:** `role="button"`, `aria-disabled`, `aria-busy` when loading

### Input

[Similar detailed spec for each component...]

### Card
### Modal
### Navigation
### Table
### Toast
### Badge
### Avatar
### Skeleton

## 7. Page Designs

### 7.1 [Page Name] — [Route]

**Layout:**
```
┌──────────────────────────────────┐
│           Navigation             │
├──────────────────────────────────┤
│                                  │
│         Hero / Header            │
│                                  │
├──────────────────────────────────┤
│                                  │
│        Main Content              │
│                                  │
├──────────────────────────────────┤
│            Footer                │
└──────────────────────────────────┘
```

**Content Hierarchy:** [What's most important, visual flow]
**Components Used:** [List of design system components]
**Responsive Notes:** [How layout changes at breakpoints]
**States:**
- Loading: [skeleton / spinner description]
- Empty: [empty state message + illustration]
- Error: [error state handling]

### 7.2 [Page Name] — [Route]
...

## 8. Animation & Transitions

### Page Transitions
- Route change: [fade / slide / none]
- Duration: 200ms
- Easing: ease-in-out

### Micro-interactions
- Button hover: scale(1.02), 150ms
- Card hover: shadow-md → shadow-lg, translateY(-2px), 200ms
- Input focus: ring-2 ring-primary, 150ms

### Loading States
- Skeleton pulse animation: 1.5s infinite
- Spinner: 0.75s linear infinite rotate
- Progress bar: determinate with percentage

## 9. Accessibility Checklist

- [ ] Color contrast ≥ 4.5:1 (normal text), ≥ 3:1 (large text)
- [ ] Focus indicators: 2px ring, visible on all interactive elements
- [ ] Alt text defined for all meaningful images
- [ ] Keyboard navigation: Tab order, Enter/Space activation
- [ ] ARIA labels on icon-only buttons
- [ ] Form errors announced to screen readers
- [ ] `prefers-reduced-motion` respected for animations
- [ ] Skip-to-content link

## 10. Dark Mode

### Color Mapping
| Light Mode | Dark Mode |
|-----------|-----------|
| white bg | gray-900 bg |
| gray-50 bg | gray-800 bg |
| gray-700 text | gray-200 text |
| white cards | gray-800 cards |

### Component Adjustments
- Shadows: reduce opacity in dark mode
- Borders: use gray-700 instead of gray-200
- Images: consider `brightness(0.9)` filter

## 11. Self-Review Log

- [Finding 1]: [change made]
