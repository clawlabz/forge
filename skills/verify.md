# /forge:verify — P6 Three-Round Multi-Agent Verification

## Overview

Conduct 3 rounds of comprehensive, multi-perspective verification. Each round: parallel agents audit different dimensions → generate fix list → parallel agents fix issues → verify fixes. Goal: drive issues to zero.

## Usage

```
/forge:verify
```

## Prerequisites

- P5 Test completed and passed

## Process

### For Each Round (1, 2, 3):

#### Step A: Multi-Agent Parallel Audit

Launch 7 verification agents in parallel, each auditing a different dimension:

**Agent 1 — Code Quality Review:**
Use `code-reviewer` agent.
- Code readability and naming conventions
- Function size (< 50 lines)
- File size (< 800 lines)
- No deep nesting (> 4 levels)
- Proper error handling
- No hardcoded values
- Immutable patterns used
- Dead code detection

**Agent 2 — Security Audit:**
Use `security-reviewer` agent + `everything-claude-code:security-scan`.
- OWASP Top 10 checklist
- Hardcoded secrets scan
- Input validation on all endpoints
- SQL injection prevention
- XSS prevention
- CSRF protection
- Auth/authz bypass attempts
- Rate limiting verification
- Error message information leakage

**Agent 3 — PRD Compliance Check:**
Use `verify-checker` agent with the PRD.
- Every P0 feature from PRD is implemented
- Acceptance criteria met for each user story
- All pages/routes from PRD exist and function
- Non-functional requirements met (performance, accessibility)
- MVP scope respected (nothing missing, nothing extra)

**Agent 4 — UX Verification:**
Use `browser-use` skill + `webapp-testing`.
- Navigate every page manually via browser
- Test all interactive elements (buttons, forms, links)
- Verify responsive layout at mobile/tablet/desktop
- Check loading states, empty states, error states
- Verify dark mode (if applicable)
- Test keyboard navigation
- Check for visual glitches

**Agent 5 — Performance Audit:**
- Re-run Lighthouse on all key pages
- Check bundle size (no unexpectedly large dependencies)
- Verify caching headers on API responses
- Check for N+1 query patterns
- Measure Time to First Byte, First Contentful Paint, Largest Contentful Paint

**Agent 6 — SEO Audit:**
Use `seo-audit` skill.
- Meta tags (title, description, og:*)
- Semantic HTML (h1-h6 hierarchy, landmarks)
- Sitemap presence
- Robots.txt
- Canonical URLs
- Structured data (if applicable)

**Agent 7 — Dead Code & Cleanup:**
Use `everything-claude-code:refactor-cleaner` (or `refactor-cleaner` agent).
- Unused imports
- Unused variables and functions
- Duplicate code
- Unused dependencies in package.json
- Unnecessary files
- Console.log statements left in production code

#### Step B: Aggregate Findings

Collect all findings from all agents into a unified list:

```markdown
| # | Agent | Severity | Finding | File:Line | Suggestion |
|---|-------|----------|---------|-----------|------------|
| 1 | Security | CRITICAL | API key exposed in client code | src/lib/api.ts:15 | Move to env var |
| 2 | PRD | HIGH | Missing password reset flow | — | Implement US-007 |
| 3 | Code Quality | MEDIUM | Function exceeds 80 lines | src/components/Dashboard.tsx:45 | Extract sub-components |
```

Prioritize: CRITICAL → HIGH → MEDIUM → LOW

#### Step C: Fix Issues

**CRITICAL and HIGH issues:** Must be fixed in this round.
**MEDIUM issues:** Fix if possible, otherwise carry to next round.
**LOW issues:** Fix if trivial, otherwise note for future.

For fixes, launch parallel agents when possible:
- Group fixes by file/module to avoid conflicts
- Each fix agent handles a group of related issues
- After fixes, re-run the specific verification that found the issue

#### Step D: Verify Fixes

After all fixes applied:
1. Run full build
2. Run full test suite
3. Re-check the specific issues that were fixed
4. Confirm each issue is resolved

#### Step E: Write Round Report

Write `docs/forge/P6-verify-round-{N}.md`:

```markdown
# Verification Report — Round [N]: [Product Name]

> Date: [date]
> Round: [N] of 3

## Audit Summary

| Dimension | Issues Found | Critical | High | Medium | Low |
|-----------|-------------|----------|------|--------|-----|
| Code Quality | X | 0 | 1 | 2 | 1 |
| Security | X | 1 | 0 | 0 | 0 |
| PRD Compliance | X | 0 | 1 | 0 | 0 |
| UX | X | 0 | 0 | 2 | 1 |
| Performance | X | 0 | 0 | 1 | 0 |
| SEO | X | 0 | 0 | 0 | 2 |
| Dead Code | X | 0 | 0 | 1 | 3 |
| **Total** | **XX** | **1** | **2** | **6** | **7** |

## Issues & Resolutions

| # | Severity | Dimension | Issue | Resolution | Status |
|---|----------|-----------|-------|------------|--------|
| 1 | CRITICAL | Security | API key in client | Moved to env var | ✅ Fixed |
| 2 | HIGH | PRD | Missing password reset | Implemented | ✅ Fixed |

## Carried to Next Round
| # | Severity | Issue | Reason |
|---|----------|-------|--------|

## Round Conclusion: [PASS / NEEDS_NEXT_ROUND]
- Issues found: XX
- Issues fixed: XX
- Issues carried: XX
```

### After All 3 Rounds

Update `.forge/state.yaml`:
```yaml
P6_verify:
  status: "completed"
  rounds:
    - { round: 1, found: 16, fixed: 14, carried: 2 }
    - { round: 2, found: 5, fixed: 5, carried: 0 }
    - { round: 3, found: 1, fixed: 1, carried: 0 }
  final_status: "PASS"
```

## Skills & Tools Used

- `code-reviewer` agent — code quality
- `security-reviewer` agent + `everything-claude-code:security-scan` — security
- `verify-checker` agent — PRD compliance
- `browser-use` + `webapp-testing` — UX verification
- `seo-audit` — SEO check
- `refactor-cleaner` agent — dead code cleanup
- `everything-claude-code:verification-loop` — structured verification

## Important

- ALL 3 rounds are mandatory — even if Round 1 finds zero issues
- CRITICAL issues MUST be fixed before proceeding to next round
- Each round should find fewer issues than the previous (convergence)
- If Round 3 still has CRITICAL/HIGH issues → run additional rounds until resolved
- Browser-based UX testing is required, not optional
