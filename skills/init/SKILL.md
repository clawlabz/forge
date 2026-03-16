# /forge:init — Initialize Forge Project

## Overview

Initialize a new project with Forge pipeline structure, configuration, and state tracking.

## Usage

```
/forge:init                        # Interactive setup
/forge:init --preset claw          # Use ClawLabz preset
/forge:init --preset generic       # Use generic preset (all empty)
```

## Process

### Step 1: Determine Project Location

Check if we're in an existing project directory or need to create one:
- If `forge.config.yaml` already exists → ask user if they want to reinitialize
- If not → proceed with initialization

### Step 2: Load Preset (if specified)

Read the preset file from the Forge plugin's `presets/` directory:
- `claw` → `presets/claw.yaml` (ClawLabz ecosystem defaults)
- `generic` → `presets/generic.yaml` (all empty, configure manually)
- If no preset specified → use `generic` as base and ask key questions interactively

### Step 3: Interactive Configuration (if no preset)

Ask the user these questions **one at a time**:
1. Project name?
2. Brief description?
3. Domain/subdomain? (e.g., `myapp.example.com`)
4. Deployment platform? (Vercel / Cloudflare Pages / Netlify)
5. Database? (Supabase / PlanetScale / Neon)
6. Enable GA4 analytics? (y/n)
7. Enable Clarity analytics? (y/n)

### Step 4: Create Directory Structure

Create the following directories and files in the current working directory:

```
./
├── forge.config.yaml              # From template + preset/answers
├── .forge/
│   ├── state.yaml                 # Initial state (all phases: not_started)
│   ├── secrets.yaml               # Empty secrets template
│   └── logs/                      # Empty logs directory
├── docs/
│   └── forge/                     # Will hold all Forge output documents
├── knowledge/                     # Will hold generated knowledge base
├── src/                           # Product source code
├── tests/                        # Test code
└── public/                       # Static assets
```

### Step 5: Initialize Git

- If not already a git repo → `git init`
- Add `.forge/secrets.yaml` to `.gitignore`
- Create initial commit: `chore: initialize Forge project`

### Step 6: Create GitHub Repo (if doc_hosting is github)

- Check if remote already exists
- If not → `gh repo create <org>/<project-name> --public --source . --remote origin --push`
- Confirm repo URL

### Step 7: Output Summary

Print:
```
✅ Forge project initialized

  Project: <name>
  Config:  ./forge.config.yaml
  State:   ./.forge/state.yaml
  Repo:    https://github.com/<org>/<name>

  Next: Run /forge:brainstorm "<your idea>" to start the pipeline
        Or:  /forge "<your idea>" to run the full pipeline
```

## State After Completion

`.forge/state.yaml` should contain:
```yaml
version: "1.0"
project: "<name>"
started_at: "<ISO timestamp>"
current_phase: "initialized"
status: "ready"
phases:
  P0_brainstorm: { status: "not_started" }
  P1_prd: { status: "not_started" }
  P2_architect: { status: "not_started" }
  P3_design: { status: "not_started" }
  P4_develop: { status: "not_started" }
  P5_test: { status: "not_started" }
  P6_verify: { status: "not_started" }
  P7_deploy: { status: "not_started" }
summary: "Project initialized, ready to start P0 Brainstorm."
```

## Important

- NEVER write secrets/API keys into `forge.config.yaml` — those go in `.forge/secrets.yaml`
- Always add `.forge/secrets.yaml` to `.gitignore`
- If preset is `claw`, auto-fill domain_suffix as `clawlabz.xyz`
