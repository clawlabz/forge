# /forge:test — P5 Automated Testing Suite

## Overview

Run comprehensive automated testing: unit tests, integration tests, E2E browser tests with screenshots. Auto-fix failures and retest until the suite is green.

## Usage

```
/forge:test
```

## Prerequisites

- P4 Development completed
- All build checkpoints passed

## Process

### Step 1: Assess Test Coverage

Read existing test files and determine what needs additional coverage:

1. Scan `tests/` and any co-located test files (`*.test.ts`, `*.spec.ts`)
2. Check if P4 was done in TDD mode (tests should already exist)
3. Identify untested areas:
   - API routes without integration tests
   - Components without unit tests
   - User flows without E2E tests
   - Edge cases not covered

### Step 2: Fill Testing Gaps

Write additional tests for uncovered areas:

**Unit Tests** (using Vitest or Jest):
- Utility functions
- Data transformations
- Validation logic
- Custom hooks
- Component rendering and interactions

**Integration Tests:**
- API route handlers (request → response)
- Database operations (CRUD through service layer)
- Auth flows (login, logout, protected routes)
- Third-party service integrations (with mocks)

**Skills to use:**
- `javascript-testing-patterns` — test patterns and best practices
- `everything-claude-code:tdd-workflow` — test structure

### Step 3: Run Unit & Integration Tests

Execute the test suite:
```bash
npm run test -- --coverage
```

**If tests fail:**
1. Analyze failure reason
2. Determine if it's a test bug or implementation bug
3. Fix the bug (prefer fixing implementation over tests, unless test is wrong)
4. Rerun
5. Repeat until all pass

**Coverage target:** ≥ 80%

### Step 4: E2E Browser Tests

Use Playwright (or Cypress per config) for end-to-end testing:

**Setup:**
1. Install Playwright if not already: `npx playwright install`
2. Create Playwright config if missing

**Test critical user flows:**
- User registration / login
- Main feature workflow (the core P0 user story)
- Navigation between all pages
- Form submissions and validation
- Error states (invalid input, network failure)
- Responsive layouts (mobile, tablet, desktop viewports)

**Skills to use:**
- `e2e-testing-patterns` — Playwright patterns
- `e2e-runner` agent — E2E test execution
- `browser-use` skill — browser automation and screenshots
- `webapp-testing` skill — web app testing toolkit
- `everything-claude-code:e2e-testing` — E2E best practices

**Screenshots:**
Capture screenshots of every key page at:
- Desktop (1280px)
- Mobile (375px)
- Dark mode (if applicable)

Save to: `.forge/screenshots/`

### Step 5: Auto-Fix Failures

For each failing test:

1. Read the error message and stack trace
2. Identify root cause:
   - **Implementation bug** → fix the code, rerun test
   - **Test bug** → fix the test, rerun
   - **Environment issue** → fix config, rerun
   - **Flaky test** → add retry or fix timing issue
3. Use `build-error-resolver` agent for complex build/type errors
4. Maximum 3 fix attempts per failure — if still failing, add to known issues

### Step 6: Performance Baseline

Run Lighthouse (or equivalent) on key pages:
```bash
npx lighthouse <url> --output json --output html
```

Record scores:
- Performance
- Accessibility
- Best Practices
- SEO

### Step 7: Generate Test Report

Write `docs/forge/P5-test-report.md`:

```markdown
# Test Report: [Product Name]

> Date: [date]
> Test Framework: [Vitest/Jest] + Playwright

## Summary

| Category | Passed | Failed | Skipped | Coverage |
|----------|--------|--------|---------|----------|
| Unit | X | 0 | 0 | XX% |
| Integration | X | 0 | 0 | — |
| E2E | X | 0 | 0 | — |

## Coverage Report
- Statements: XX%
- Branches: XX%
- Functions: XX%
- Lines: XX%

## E2E Test Results

| Test | Status | Duration |
|------|--------|----------|
| User registration flow | ✅ | 2.3s |
| Login flow | ✅ | 1.8s |
| [Core feature flow] | ✅ | 3.1s |

## Browser Screenshots

### Desktop (1280px)
- [Page 1]: screenshot path
- [Page 2]: screenshot path

### Mobile (375px)
- [Page 1]: screenshot path
- [Page 2]: screenshot path

## Lighthouse Scores

| Page | Performance | Accessibility | Best Practices | SEO |
|------|------------|---------------|----------------|-----|
| / | XX | XX | XX | XX |
| /dashboard | XX | XX | XX | XX |

## Failure Fix Log

| # | Test | Error | Root Cause | Fix |
|---|------|-------|-----------|-----|

## Known Issues
- [Any tests that couldn't be fixed within 3 attempts]
```

### Step 8: Update State

```yaml
P5_test:
  status: "completed"
  coverage: "85%"
  results:
    unit: { passed: 42, failed: 0 }
    integration: { passed: 15, failed: 0 }
    e2e: { passed: 8, failed: 0 }
  lighthouse:
    performance: 92
    accessibility: 98
    best_practices: 95
    seo: 100
```

## Important

- Tests must be deterministic — no random failures
- E2E tests should use test data, not production data
- Screenshots are REQUIRED for the test report
- Coverage ≥ 80% is the target, but don't write meaningless tests to hit the number
- If E2E test environment (dev server) is needed, start it before running tests
