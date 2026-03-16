# /forge:develop — P4 Multi-Agent Parallel Development

## Overview

Execute the development plan from P2, building the product phase by phase with multi-agent parallel development, automatic checkpoints, and continuous progress tracking.

## Usage

```
/forge:develop
```

## Prerequisites

- P2 Architecture approved (`docs/forge/P2-architecture.md`)
- P2 Dev Plan approved (`docs/forge/P2-dev-plan.md`)
- P3 Design Spec approved (`docs/forge/P3-design-spec.md`)

## Process

### Step 1: Read Context & Plan

Read these files:
- `docs/forge/P2-architecture.md` — tech stack, schema, API design
- `docs/forge/P2-dev-plan.md` — phased task list with dependencies
- `docs/forge/P3-design-spec.md` — design system, component specs, page layouts
- `forge.config.yaml` — provider configuration
- `.forge/state.yaml` — check if resuming mid-development

### Step 2: Determine Development Mode

Analyze the project type to choose the development approach:

```
Complex business logic (financial, state machines, permissions, data processing)
  → TDD: write tests first, then implement

Display-focused (landing page, dashboard, portfolio, documentation site)
  → Standard: implement first, add tests after

Mixed project
  → Hybrid: TDD for core logic modules, standard for UI/pages
```

Record the chosen mode in `.forge/state.yaml` under `P4_develop.dev_mode`.

### Step 3: Project Scaffolding

Before starting phase-by-phase development:

1. **Initialize project** — `npx create-next-app@latest` or equivalent per tech stack
2. **Install dependencies** — All packages from P2 architecture
3. **Configure tooling** — ESLint, Prettier, TypeScript config
4. **Set up database** — Run initial migration with schema from P2
5. **Implement design system** — Create base Tailwind config from P3 design spec (colors, fonts, spacing)
6. **Create component library** — Implement core UI components from P3 design spec
7. **Set up project structure** — Create directory tree from P2 architecture

Verify: `build` passes, `lint` passes.

### Step 4: Phase-by-Phase Execution

For each phase in the dev plan:

#### 4a. Identify Parallel Tasks

Read the phase's task list and dependency graph:
- Tasks with no dependencies on each other → **run in parallel** as separate Agents
- Tasks with dependencies → **run sequentially**

#### 4b. Execute Tasks

For each task (or parallel group of tasks):

**If TDD mode:**
1. Write test file first (unit test for the module)
2. Run test — verify it FAILS (RED)
3. Implement the minimum code to pass
4. Run test — verify it PASSES (GREEN)
5. Refactor if needed (IMPROVE)

**If Standard mode:**
1. Implement the feature/module
2. Write tests after implementation
3. Run tests — verify they pass

**Skills & Agents to use during development:**
- `tdd-guide` agent — when in TDD mode
- `vercel-react-best-practices` — for React/Next.js code
- `fullstack-guardian` — for security-aware full-stack code
- `everything-claude-code:coding-standards` — code quality
- `supabase-postgres-best-practices` — for database operations
- `security-review` — for auth, input handling, API endpoints

**For parallel execution:**
```
Launch multiple Agents simultaneously:
- Agent 1: Implement [Task A] — operates on src/features/featureA/
- Agent 2: Implement [Task B] — operates on src/features/featureB/
- Agent 3: Implement [Task C] — operates on src/features/featureC/
```

**Critical rule for parallel tasks:** Each parallel agent must work on DIFFERENT files/directories to avoid conflicts.

#### 4c. Update Progress

After each task completes, update `.forge/state.yaml`:
```yaml
P4_develop:
  sub_phases:
    - name: "Phase 1: Foundation"
      tasks:
        - { name: "Project scaffolding", status: "completed" }
        - { name: "Database setup", status: "completed" }
        - { name: "Auth flow", status: "running" }
```

#### 4d. Phase Checkpoint

After ALL tasks in a phase complete:

1. **Build check**: `npm run build` (or equivalent)
2. **Lint check**: `npm run lint`
3. **Test check**: Run all tests written so far
4. **Code review**: Launch `code-reviewer` agent on the phase's code

Record checkpoint results:
```yaml
checkpoint: { build: "pass", lint: "pass", test: "14/14 pass", review: "2 medium issues fixed" }
```

If checkpoint fails:
1. Launch `build-error-resolver` agent for build/lint failures
2. Fix failing tests
3. Re-run checkpoint
4. If still failing after 3 attempts → pause and inform user

### Step 5: Cross-Phase Integration

After all development phases complete:
1. Run full build
2. Run full test suite
3. Verify all pages render correctly
4. Check all API endpoints respond
5. Fix any integration issues

### Step 6: Final Development Summary

Update `.forge/state.yaml`:
```yaml
P4_develop:
  status: "completed"
  dev_mode: "hybrid"
  sub_phases: [...]
  summary:
    total_tasks: 28
    completed: 28
    files_created: 45
    test_count: 67
    test_pass_rate: "100%"
```

## Skills & Tools Used

- `tdd-guide` agent — TDD workflow enforcement
- `code-reviewer` agent — phase-end code review
- `build-error-resolver` agent — automatic build fix
- `vercel-react-best-practices` — React/Next.js patterns
- `fullstack-guardian` — security-aware development
- `everything-claude-code:coding-standards` — code quality standards
- `everything-claude-code:tdd-workflow` — TDD patterns
- `supabase-postgres-best-practices` — database operations
- `security-review` — security patterns
- `frontend-design` — UI implementation from design spec
- `dispatching-parallel-agents` — multi-agent orchestration

## Important Rules

- NEVER skip a checkpoint — every phase must pass build + lint + test
- NEVER have parallel agents edit the same file
- ALWAYS update state.yaml after each task completion
- If a phase has more than 10 tasks, break it into sub-phases
- Implement design system components BEFORE page-specific code
- Database migrations run BEFORE application code that depends on them
- Follow the architecture document — do not deviate without updating it
