# /forge:prd — P1 Product Requirements Document

## Overview

Generate a professional, comprehensive PRD based on the P0 brainstorm report. The PRD goes through 2 rounds of self-review before being pushed to GitHub for human approval.

## Usage

```
/forge:prd
```

## Prerequisites

- P0 brainstorm report must exist at `docs/forge/P0-brainstorm-report.md`
- P0 recommendation must be GO (or PIVOT with accepted modifications)

## Process

### Step 1: Read Context

Read these files:
- `docs/forge/P0-brainstorm-report.md` — brainstorm findings
- `forge.config.yaml` — project configuration

### Step 2: Draft PRD v1

Write a comprehensive PRD using the template structure. Include:

**Core sections:**
1. **Background & Objectives** — Why build this? What success looks like.
2. **Target User Personas** — 2-3 detailed personas with goals, pain points, tech proficiency.
3. **Core Features** — Prioritized feature list with P0 (must-have), P1 (should-have), P2 (nice-to-have).
4. **User Stories & Acceptance Criteria** — Given/When/Then format for every P0 feature.
5. **Non-functional Requirements** — Performance targets, security requirements, accessibility (WCAG), browser support, mobile responsiveness.
6. **Data Model Overview** — Key entities and relationships (not full schema, that's P2).
7. **Page/Route List** — Every page with its purpose and key components.
8. **MVP Scope** — Explicit boundary: what's IN and what's OUT.
9. **Success Metrics** — Quantifiable KPIs (DAU, conversion rate, load time, etc.).
10. **Self-Review Log** — Changes made during self-review rounds.

**Writing guidelines:**
- Be specific, not vague. "Fast" → "Page loads in <2 seconds on 3G"
- Every feature must have acceptance criteria
- YAGNI ruthlessly — remove anything not essential for MVP
- Use the competitor analysis from P0 to inform feature decisions

### Step 3: Self-Review Round 1 — Product Manager Perspective

Launch the `prd-reviewer` agent with role="product_manager":

Review checklist:
- [ ] All user personas have corresponding user stories
- [ ] Feature priorities are justified (why P0 vs P1?)
- [ ] Acceptance criteria are testable and unambiguous
- [ ] Edge cases and error states are covered
- [ ] No feature creep — everything ties back to core problem
- [ ] MVP scope is realistic and focused
- [ ] Success metrics are measurable
- [ ] No contradictions between sections

Record all changes in the Self-Review Log section.

### Step 4: Self-Review Round 2 — User Perspective

Launch the `prd-reviewer` agent with role="target_user":

Review checklist:
- [ ] Does this actually solve the user's problem?
- [ ] Is the proposed UX intuitive for the target user?
- [ ] Are there unnecessary features the user wouldn't care about?
- [ ] Is anything missing that the user would expect?
- [ ] Would the user pay for / use this over competitors?
- [ ] Are the non-functional requirements aligned with user expectations?

Record all changes in the Self-Review Log section.

### Step 5: Finalize Document

Write the final PRD to `docs/forge/P1-prd.md`.

### Step 6: Push to GitHub & Request Approval

1. `git add docs/forge/P1-prd.md`
2. `git commit -m "docs: P1 PRD for [product name]"`
3. `git push`
4. Output:
   ```
   🔴 审批门控

   请审批 PRD: https://github.com/<org>/<repo>/blob/main/docs/forge/P1-prd.md

   回复 "approved" 继续，或提出修改意见。
   ```

### Step 7: Handle Approval Response

- **Approved** → update state, continue to P2+P3
- **Changes requested** → incorporate feedback, re-run from Step 2 with modifications, increment attempt counter

## Output Template

```markdown
# PRD: [Product Name]

> Version: 1.0
> Date: [date]
> Status: Pending Approval
> Based on: P0 Brainstorm Report

## 1. Background & Objectives
### Problem Statement
### Product Vision
### Success Criteria

## 2. Target User Personas
### Persona 1: [Name]
- Role / Demographics
- Goals
- Pain Points
- Tech Proficiency

### Persona 2: [Name]
...

## 3. Core Features
### P0 — Must Have (MVP)
| # | Feature | Description | User Story Ref |
### P1 — Should Have (v1.1)
| # | Feature | Description |
### P2 — Nice to Have (Future)
| # | Feature | Description |

## 4. User Stories & Acceptance Criteria
### US-001: [Story Title]
**As a** [persona], **I want to** [action], **so that** [benefit].
**Acceptance Criteria:**
- Given [context], When [action], Then [result]
- Given ...

### US-002: ...

## 5. Non-functional Requirements
### Performance
### Security
### Accessibility
### Browser Support
### Mobile Responsiveness

## 6. Data Model Overview
### Key Entities
### Relationships (ERD description)

## 7. Page / Route List
| Page | Route | Purpose | Key Components |
|------|-------|---------|---------------|

## 8. MVP Scope
### In Scope
### Out of Scope (Explicitly)

## 9. Success Metrics
| Metric | Target | Measurement Method |

## 10. Self-Review Log
### Round 1: Product Manager Review
- [Change 1]: [reason]
- [Change 2]: [reason]
### Round 2: User Perspective Review
- [Change 1]: [reason]
- [Change 2]: [reason]
```

## Skills & Tools Used

- `writing-plans` skill — structured document writing
- `everything-claude-code:blueprint` — requirements decomposition
- `prd-reviewer` agent — self-review (launched twice with different roles)

## Important

- Do NOT include technical implementation details — that's P2's job
- Do NOT include visual design — that's P3's job
- Focus on WHAT, not HOW
- Every feature must trace back to a user problem identified in P0
