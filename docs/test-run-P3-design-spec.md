# ClawToolkit — Design Specification (P3)

> AI Agent Tool Marketplace · Next.js + Tailwind + shadcn/ui · OpenClaw Ecosystem

---

## 1. Design Direction

**Three Principles**

1. **Developer-first clarity** — Information density is a feature, not a flaw. Monospace accents, structured layouts, inline code everywhere.
2. **Purposeful dark** — Dark mode is the default experience. Light mode is not an afterthought — both are first-class.
3. **Earned trust** — Subtle motion, precise typography, and consistent spacing signal craftsmanship. Nothing screams; everything informs.

**Emotional Tone**: Vercel's precision + GitHub's community warmth + a hint of underground tech energy. Confident without being loud. The palette feels like late-night terminal sessions and polished demos.

---

## 2. Color System

### Base Palette (Hex)

| Token | Light | Dark | Usage |
|---|---|---|---|
| `primary-500` | `#6C3DE8` | `#7C52F0` | CTAs, links, active states |
| `primary-600` | `#5A2DD4` | `#6C3DE8` | Button hover |
| `primary-100` | `#EDE8FF` | `#1A1330` | Subtle backgrounds |
| `secondary-500` | `#3B82F6` | `#60A5FA` | Secondary actions, info |
| `accent-500` | `#A855F7` | `#C084FC` | Badges, highlights, tags |
| `accent-neon` | `#818CF8` | `#A5B4FC` | Hover glow, decorative |

### Semantic Colors

| Token | Light | Dark |
|---|---|---|
| `success` | `#22C55E` | `#4ADE80` |
| `warning` | `#F59E0B` | `#FCD34D` |
| `error` | `#EF4444` | `#F87171` |
| `info` | `#3B82F6` | `#60A5FA` |

### Neutrals

| Token | Light | Dark |
|---|---|---|
| `bg-base` | `#FFFFFF` | `#0A0A0F` |
| `bg-surface` | `#F8F7FF` | `#111118` |
| `bg-elevated` | `#F1EFF9` | `#1A1A24` |
| `border` | `#E2E0F0` | `#2A2A3A` |
| `border-subtle` | `#EDE8FF` | `#1E1E2C` |
| `text-primary` | `#0F0E17` | `#F1EFF9` |
| `text-secondary` | `#4B4869` | `#9390B0` |
| `text-muted` | `#7C7A96` | `#5E5C7A` |
| `text-inverse` | `#FFFFFF` | `#0F0E17` |

### Tailwind CSS Variables (in `globals.css`)

```css
:root {
  --primary: 263 70% 57%;        /* #6C3DE8 */
  --primary-foreground: 0 0% 100%;
  --accent: 270 60% 65%;         /* #A855F7 */
  --background: 0 0% 100%;
  --foreground: 248 80% 7%;
}
.dark {
  --primary: 255 80% 63%;        /* #7C52F0 */
  --background: 240 20% 4%;      /* #0A0A0F */
  --foreground: 255 30% 96%;
}
```

---

## 3. Typography

**Font Stack**

| Role | Font | Import |
|---|---|---|
| Display / Heading | `Space Grotesk` | `weights=400;500;600;700` |
| Body | `Inter` | `weights=400;500` |
| Monospace / Code | `JetBrains Mono` | `weights=400;500` |

**Type Scale**

| Name | Size | Line Height | Weight | Usage |
|---|---|---|---|---|
| `text-xs` | 12px | 16px | 400 | Badges, captions |
| `text-sm` | 14px | 20px | 400/500 | Body secondary, labels |
| `text-base` | 16px | 24px | 400 | Body primary |
| `text-lg` | 18px | 28px | 500 | Section leads |
| `text-xl` | 20px | 28px | 600 | Card titles |
| `text-2xl` | 24px | 32px | 600 | Page section headings |
| `text-3xl` | 30px | 36px | 700 | Page titles |
| `text-4xl` | 36px | 40px | 700 | Hero headline |
| `text-5xl` | 48px | 52px | 700 | Hero display |
| `mono-sm` | 13px | 20px | 400 | Inline code |
| `mono-base` | 14px | 22px | 400 | Code blocks |

---

## 4. Spacing & Layout

### Spacing Scale (Tailwind defaults, key values)

| Token | px | Usage |
|---|---|---|
| `space-1` | 4 | Tight gaps, icon padding |
| `space-2` | 8 | Component internal |
| `space-3` | 12 | Dense list items |
| `space-4` | 16 | Standard padding |
| `space-6` | 24 | Section gaps |
| `space-8` | 32 | Card padding |
| `space-12` | 48 | Section vertical rhythm |
| `space-16` | 64 | Page sections |
| `space-24` | 96 | Hero sections |

### Grid & Container

- Max content width: `1280px` (`max-w-7xl`)
- Narrow (docs/auth): `768px` (`max-w-3xl`)
- Side padding: `px-4 sm:px-6 lg:px-8`
- Main layout: CSS Grid `grid-cols-12`, gap `gap-6`
- Dashboard aside: `240px` fixed, content fills remainder

### Breakpoints

| Name | Min Width | Notes |
|---|---|---|
| `sm` | 640px | Mobile landscape |
| `md` | 768px | Tablet |
| `lg` | 1024px | Desktop |
| `xl` | 1280px | Wide desktop |
| `2xl` | 1536px | Ultra-wide |

---

## 5. Border & Shadow

### Radius Scale

| Token | Value | Usage |
|---|---|---|
| `rounded-sm` | 4px | Badges, inline chips |
| `rounded` | 6px | Inputs, buttons |
| `rounded-md` | 8px | Cards, dropdowns |
| `rounded-lg` | 12px | Modals, panels |
| `rounded-xl` | 16px | Feature cards, hero blocks |
| `rounded-full` | 9999px | Avatars, pill badges |

### Shadow Scale

| Token | Value | Usage |
|---|---|---|
| `shadow-sm` | `0 1px 3px rgba(0,0,0,0.06)` | Inputs on focus |
| `shadow` | `0 2px 8px rgba(0,0,0,0.1)` | Cards |
| `shadow-md` | `0 4px 16px rgba(0,0,0,0.15)` | Dropdowns |
| `shadow-lg` | `0 8px 32px rgba(0,0,0,0.2)` | Modals |
| `shadow-glow` | `0 0 20px rgba(108,61,232,0.3)` | Primary CTA hover |
| `shadow-glow-sm` | `0 0 8px rgba(108,61,232,0.2)` | Active nav items |

Dark mode: all shadows increase opacity by ~50%; glow becomes `rgba(124,82,240,0.4)`.

---

## 6. Component Library

### Button

**Variants**: `primary`, `secondary`, `ghost`, `danger`, `outline`
**Sizes**: `sm` (h-8 px-3 text-sm), `md` (h-10 px-4 text-sm), `lg` (h-12 px-6 text-base)
**States**: default, hover, active, disabled, loading (spinner left)

```
primary: bg-primary-500 text-white hover:bg-primary-600 hover:shadow-glow rounded
ghost:   bg-transparent text-primary-500 hover:bg-primary-100 dark:hover:bg-primary-100/10
outline: border border-border text-text-primary hover:border-primary-500
danger:  bg-error text-white hover:bg-red-600
```

### Input

**Variants**: `default`, `search` (with icon left), `code` (monospace)
**States**: default, focus (ring-2 ring-primary-500/50), error (border-error), disabled

```
base: h-10 w-full rounded border border-border bg-bg-surface px-3 text-sm
      focus:outline-none focus:ring-2 focus:ring-primary-500/50 transition-shadow
```

### Card

**Variants**: `default`, `hover` (lift on hover), `bordered`, `ghost`

```
default: rounded-md bg-bg-surface border border-border p-6 shadow-sm
hover:   + hover:-translate-y-0.5 hover:shadow-md hover:border-primary-500/40 transition-all
```

### Modal

```
overlay: fixed inset-0 bg-black/60 backdrop-blur-sm z-50
panel:   relative bg-bg-surface rounded-lg shadow-lg max-w-lg w-full mx-4 p-6
header:  flex items-center justify-between mb-4 pb-4 border-b border-border
```

### Nav (Top)

```
container: sticky top-0 z-40 h-14 bg-bg-base/80 backdrop-blur-md border-b border-border
inner:     max-w-7xl mx-auto px-6 flex items-center gap-6 h-full
logo:      font-display font-bold text-lg text-primary-500
links:     text-sm text-text-secondary hover:text-text-primary transition-colors
cta:       Button primary sm ml-auto
```

### Table

```
wrapper: w-full overflow-x-auto rounded-md border border-border
table:   w-full text-sm
thead:   bg-bg-elevated text-text-secondary text-xs uppercase tracking-wide
td/th:   px-4 py-3 border-b border-border last:border-0
row:     hover:bg-bg-elevated/50 transition-colors
```

### Toast

**Variants**: `success`, `error`, `warning`, `info`

```
base: fixed bottom-4 right-4 z-50 flex items-center gap-3
      min-w-72 rounded-md border shadow-lg px-4 py-3 text-sm
      animate-in slide-in-from-bottom-2
```

### Badge

**Variants**: `default`, `primary`, `success`, `warning`, `error`, `outline`

```
base:    inline-flex items-center rounded-sm px-2 py-0.5 text-xs font-medium
primary: bg-primary-100 text-primary-600 dark:bg-primary-100/20 dark:text-primary-400
```

### CodeBlock

```
wrapper: relative rounded-md bg-[#0D0D14] border border-border overflow-hidden
header:  flex items-center justify-between px-4 py-2 border-b border-border text-xs text-text-muted
         bg-[#111118] font-mono
pre:     overflow-x-auto p-4 text-sm font-mono leading-relaxed text-[#E2E0F0]
copy:    Button ghost sm absolute top-2 right-2
```
Syntax highlighting via `shiki` with `vesper` theme.

### PricingCard

```
wrapper: Card hover with border-2; active tier uses border-primary-500
badge:   "Most Popular" Badge primary absolute -top-3 left-1/2 -translate-x-1/2
price:   text-4xl font-bold + /month text-text-muted text-base
feature: flex gap-2 items-start text-sm + Check icon text-success
```

---

## 7. Page Layouts

### / (Home)

```
[NAV]
[HERO: headline + sub + search bar + CTA row]
[FEATURED TOOLS: horizontal scroll cards row]
[CATEGORIES: 3-col icon grid]
[TRENDING: 4-col tool cards]
[HOW IT WORKS: 3-step horizontal]
[FOOTER]
```
Content: headline "The Tool Layer for AI Agents", sub "Discover, install, and publish tools for your AI agents.", search centered, 2 CTAs (Browse Tools / Publish Your Tool). Responsive: single col on mobile, hero stacks vertically.

### /search

```
[NAV]
[SEARCH BAR full-width sticky below nav]
[2-COL: FILTERS sidebar (240px) | RESULTS grid]
  Filters: category, pricing, rating, sort
  Results: 3-col card grid → 2-col tablet → 1-col mobile
[PAGINATION]
```
Key components: Input search, Badge filters, ToolCard, Skeleton loaders. Filter sidebar collapses to bottom sheet on mobile.

### /tools/[slug]

```
[NAV]
[BREADCRUMB]
[2-COL: MAIN (8/12) | SIDEBAR (4/12)]
  Main: Title + meta row + description + README (markdown)
        Tabs: Overview | API Docs | Changelog | Reviews
  Sidebar: PricingCard + install snippet + author card + stats
[RELATED TOOLS]
```
Key: CodeBlock for install/usage, Tab component, Review list with stars. Sidebar sticks on scroll (sticky top-20).

### /publish

```
[NAV]
[STEPPER: 1.Basics → 2.Schema → 3.Pricing → 4.Review]
[FORM CARD centered max-w-2xl]
  Step 1: name, slug, description, category, tags
  Step 2: JSON schema editor (CodeBlock editable) + preview
  Step 3: PricingCard builder (Free/Pro/Enterprise)
  Step 4: Preview card + submit
[FOOTER minimal]
```
Multi-step form with progress bar. Schema step has split view (editor left, preview right) on lg+.

### /dashboard

```
[NAV]
[DASHBOARD LAYOUT: ASIDE (240px fixed) | MAIN]
  Aside: logo, nav links (Overview, My Tools, Purchases, Billing, Settings)
  Main: Stats row (4 cards: installs, revenue, rating, CP balance)
        Recent Activity table
        Quick actions
```
Stats cards use large numbers with trend arrows. Dark bg-elevated sidebar. Aside collapses to top nav on mobile.

### /dashboard/publisher

```
[DASHBOARD LAYOUT]
  Main: [Tools Table with status badges]
        [Revenue chart — simple bar chart]
        [Payout history table]
        [Publish new tool CTA]
```
Table rows: tool name, installs, revenue, status (Live/Draft/Review). Status uses Badge variants.

### /dashboard/topup

```
[DASHBOARD LAYOUT]
  Main: [CP Balance card — current balance prominent]
        [Amount selector: preset chips (100/500/1000/5000 CP) + custom input]
        [Payment method: card/crypto]
        [Order summary card]
        [Pay button primary lg]
```
PricingCard style for CP packages with "Best Value" badge. Stripe Elements iframe for card.

### /auth/login

```
[CENTER LAYOUT: logo + card max-w-sm]
  Card: "Sign in to ClawToolkit"
        Email input + Password input
        Forgot password link
        Button primary full-width
        Divider "or"
        GitHub OAuth button
        Link to /auth/signup
```
Minimal chrome, logo top, no nav. Background: subtle grid pattern with primary glow.

### /auth/signup

```
[CENTER LAYOUT: logo + card max-w-sm]
  Card: "Create your account"
        Name + Email + Password inputs
        Terms checkbox
        Button primary full-width
        GitHub OAuth
        Link to /auth/login
```
Same pattern as login. Password strength indicator below field.

### /settings

```
[DASHBOARD LAYOUT]
  Main: [Tabbed sections: Profile | Security | API Keys | Notifications | Danger Zone]
        Profile: avatar upload + display name + bio + email
        API Keys: Table + generate new key button + CodeBlock for key display
        Danger Zone: delete account (red border card)
```
Each section is a Card. API key revealed once in CodeBlock with copy button.

### /docs

```
[NAV]
[2-COL: DOC SIDEBAR (260px) | CONTENT (max-w-3xl)]
  Sidebar: nested navigation (Getting Started, API Reference, SDK, Guides)
  Content: markdown rendered, h2 anchors, CodeBlocks, prev/next nav
```
Sidebar active link: `text-primary-500 bg-primary-100/20 rounded`. Content uses `prose` class with custom overrides. Mobile: sidebar as drawer.

---

## 8. Animation

### Page Transitions

```
// Next.js layout animation via Framer Motion
initial: { opacity: 0, y: 8 }
animate: { opacity: 1, y: 0 }
transition: { duration: 0.2, ease: "easeOut" }
```

### Micro-interactions

| Interaction | Animation |
|---|---|
| Button hover | `transition-all duration-150` + shadow-glow on primary |
| Card hover | `transition-all duration-200 -translate-y-0.5` |
| Nav link | `transition-colors duration-150` |
| Toast appear | `animate-in slide-in-from-bottom-2 duration-300` |
| Modal open | `animate-in fade-in zoom-in-95 duration-200` |
| Dropdown | `animate-in slide-in-from-top-1 duration-150` |
| Skeleton | `animate-pulse bg-border rounded` |

### Loading States

- **Page load**: Skeleton cards matching layout shape
- **Button loading**: Spinner (16px) replacing left icon, `cursor-not-allowed opacity-75`
- **Table loading**: 5 skeleton rows, column widths matching real data
- **Search**: Debounced 300ms, spinner in search input right

---

## 9. Accessibility

- [ ] All interactive elements reachable by keyboard (Tab order logical)
- [ ] Focus rings visible: `focus-visible:ring-2 ring-primary-500 ring-offset-2`
- [ ] Color contrast AA minimum (4.5:1 body, 3:1 large text)
- [ ] All images have descriptive `alt` text; decorative images `alt=""`
- [ ] Form inputs have associated `<label>` or `aria-label`
- [ ] Error messages linked via `aria-describedby`
- [ ] Modals trap focus; `Escape` closes; returns focus on close
- [ ] `role="alert"` on Toast messages
- [ ] Skip-to-main link as first focusable element
- [ ] Icons that convey meaning have `aria-label`; decorative icons `aria-hidden`
- [ ] Reduced motion: wrap animations in `@media (prefers-reduced-motion: no-preference)`
- [ ] `lang="en"` on `<html>`; page `<title>` unique per page
- [ ] Semantic HTML: `<nav>`, `<main>`, `<aside>`, `<header>`, `<footer>`

---

## 10. Dark Mode

Dark mode via Tailwind `class` strategy (`darkMode: 'class'`). Toggle stored in `localStorage`, applied on `<html>`.

| Purpose | Light | Dark |
|---|---|---|
| Page background | `#FFFFFF` | `#0A0A0F` |
| Surface (cards) | `#F8F7FF` | `#111118` |
| Elevated surface | `#F1EFF9` | `#1A1A24` |
| Border default | `#E2E0F0` | `#2A2A3A` |
| Border subtle | `#EDE8FF` | `#1E1E2C` |
| Text primary | `#0F0E17` | `#F1EFF9` |
| Text secondary | `#4B4869` | `#9390B0` |
| Text muted | `#7C7A96` | `#5E5C7A` |
| Primary CTA | `#6C3DE8` | `#7C52F0` |
| Primary hover | `#5A2DD4` | `#6C3DE8` |
| Primary subtle bg | `#EDE8FF` | `#1A1330` |
| Accent | `#A855F7` | `#C084FC` |
| Success | `#22C55E` | `#4ADE80` |
| Warning | `#F59E0B` | `#FCD34D` |
| Error | `#EF4444` | `#F87171` |
| Code block bg | `#F5F3FF` | `#0D0D14` |
| Code block border | `#E2E0F0` | `#1E1E2C` |
| Nav background | `rgba(255,255,255,0.8)` | `rgba(10,10,15,0.8)` |
| Sidebar background | `#F8F7FF` | `#0E0E16` |

**Implementation**: All color tokens use `bg-bg-surface dark:bg-[#111118]` pattern or CSS variables with `hsl()`. Prefer Tailwind `dark:` variants over manual `[color]` for maintainability. CodeBlock always uses dark theme regardless of mode (code is always on dark bg).
