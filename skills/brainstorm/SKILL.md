---
name: brainstorm
description: "P0 brainstorm: market research, competitor analysis, business model validation"
---

# /forge:brainstorm — P0 Brainstorm & Validation

## Overview

Transform a one-line idea into a validated product concept through market research, competitor analysis, feasibility assessment, and business model validation.

## Usage

```
/forge:brainstorm "A real-time collaborative whiteboard for remote teams"
```

## Process

### Step 1: Understand the Idea

Parse the input idea and identify:
- Core problem being solved
- Target user segment
- Proposed solution approach
- Key assumptions to validate

### Step 2: Market Research

Use `agent-reach` (WebSearch) and `everything-claude-code:market-research` to research:
- **Market size**: TAM/SAM/SOM estimates
- **Market trends**: Is this market growing? What's driving growth?
- **Timing**: Why now? What has changed that makes this viable?
- **Regulatory landscape**: Any compliance concerns?

Search queries to run:
- "[idea domain] market size 2025 2026"
- "[idea domain] trends"
- "[idea domain] startup funding"
- Related industry reports

### Step 3: Competitor Analysis

Use `agent-reach` and `everything-claude-code:search-first` to find:
- **Direct competitors**: Products solving the same problem
- **Indirect competitors**: Alternative solutions users currently use
- **Open source alternatives**: Existing projects that could be leveraged

Build a comparison matrix:
| Competitor | Strengths | Weaknesses | Pricing | Differentiator |
|-----------|-----------|-----------|---------|---------------|
| ... | ... | ... | ... | ... |

### Step 4: Technical Feasibility

Use the Architect agent to assess:
- Can this be built with the default stack (Next.js + Vercel + Supabase)?
- Are there technical risks or unknowns?
- What third-party services/APIs are needed?
- Estimated complexity (simple / moderate / complex)

### Step 5: Existing Solutions Research

Use `everything-claude-code:search-first` to find:
- Open source projects solving 80%+ of the problem
- npm/PyPI packages that could be leveraged
- GitHub repos with relevant implementations
- Templates or boilerplates to accelerate development

### Step 6: Business Model Validation

Analyze:
- **Revenue model**: How does this make money? (SaaS, freemium, marketplace, ads, etc.)
- **Cost structure**: Hosting, APIs, third-party services
- **Unit economics**: Can this be profitable?
- **Growth strategy**: How to acquire users?
- **Moat**: What prevents competitors from copying?

### Step 7: Synthesize & Decide

Based on all research, make a recommendation:

- **GO** — Idea is validated, market exists, technically feasible, business model works
- **PIVOT** — Core idea has merit but needs modification (specify what to change)
- **KILL** — Fatal flaws found (specify why: no market, technically impossible, saturated, etc.)

### Step 8: Write Report

Create `docs/forge/P0-brainstorm-report.md`:

```markdown
# P0 Brainstorm Report: [Product Name]

Generated: [date]
Idea: "[original idea]"

## 1. Product Concept
### One-line Definition
### Detailed Description
### Core Problem
### Target Users

## 2. Market Research
### Market Size (TAM/SAM/SOM)
### Market Trends
### Timing Analysis

## 3. Competitor Analysis
### Direct Competitors
| Competitor | Strengths | Weaknesses | Pricing | Differentiator |
### Indirect Competitors
### Open Source Alternatives

## 4. Technical Feasibility
### Recommended Stack
### Technical Risks
### Third-party Dependencies
### Complexity Assessment

## 5. Existing Solutions
### Reusable Projects/Libraries
### Potential Starting Points

## 6. Business Model
### Revenue Model
### Cost Structure
### Unit Economics
### Growth Strategy
### Competitive Moat

## 7. SWOT Analysis
| Strengths | Weaknesses |
| Opportunities | Threats |

## 8. Recommendation: [GO / PIVOT / KILL]
### Rationale
### [If PIVOT] Suggested Modifications
### [If KILL] Fatal Flaws
```

### Step 9: Update State

Update `.forge/state.yaml`:
- Set `P0_brainstorm.status` to `completed`
- Record outputs
- Update summary

## Skills & Tools Used

- `agent-reach` / `WebSearch` — market and competitor research
- `brainstorming` skill — structured ideation
- `everything-claude-code:market-research` — deep market analysis
- `everything-claude-code:search-first` — find existing solutions
- Architect agent — technical feasibility assessment

## Important

- Research must be thorough but time-boxed — don't spend excessive time on any single research dimension
- If web search is unavailable, use knowledge-based analysis and clearly note limitations
- Always present evidence for the GO/PIVOT/KILL recommendation
- The PIVOT suggestion should be specific and actionable, not vague
