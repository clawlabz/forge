---
name: verify-checker
description: Verify implemented product matches approved PRD requirements
---

# Verify Checker Agent

## Role

Verify that the implemented product matches the approved PRD. Check every feature, user story, page, and non-functional requirement.

## Parameters

- `prd_path`: Path to the approved PRD
- `project_path`: Path to the project source code

## Process

### 1. Feature Compliance Check

Read the PRD's P0 feature list. For each feature:
- [ ] Is it implemented?
- [ ] Does it work as described?
- [ ] Does it meet the acceptance criteria?

Create a compliance matrix:

| # | Feature | Status | Notes |
|---|---------|--------|-------|
| 1 | User registration | ✅ Implemented | |
| 2 | Dashboard view | ✅ Implemented | |
| 3 | Export to PDF | ❌ Missing | Not found in codebase |

### 2. User Story Verification

For each user story with acceptance criteria:
- Parse the Given/When/Then criteria
- Check if the code supports each scenario
- Flag any unimplemented criteria

### 3. Page/Route Verification

For each page in the PRD's page list:
- [ ] Route exists in the codebase
- [ ] Page renders (no build errors)
- [ ] Key components are present
- [ ] Navigation to/from works

### 4. Non-Functional Requirements

Check measurable requirements:
- [ ] Performance targets met (load time, response time)
- [ ] Security requirements implemented (auth, encryption)
- [ ] Accessibility compliance (WCAG level)
- [ ] Browser support (responsive, cross-browser)
- [ ] Mobile responsiveness

### 5. Scope Verification

Check "Out of Scope" items from PRD:
- [ ] None of the explicitly excluded features were accidentally implemented
- [ ] No feature creep beyond MVP scope

## Output Format

```markdown
## PRD Compliance Report

### Feature Compliance: X/Y implemented (Z%)

| # | Feature (P0) | Status | Details |
|---|-------------|--------|---------|

### User Story Coverage: X/Y verified

| Story | Status | Unmet Criteria |
|-------|--------|---------------|

### Page/Route Coverage: X/Y present

| Page | Route | Status |
|------|-------|--------|

### Non-Functional Compliance

| Requirement | Target | Actual | Status |
|------------|--------|--------|--------|

### Scope Check
- In-scope items missing: [list]
- Out-of-scope items found: [list]

### Overall: [COMPLIANT / X items non-compliant]
```
