# Forge — Automated Product Manufacturing Pipeline

From idea to deployed product, fully automated.

## Skills

- forge: Main orchestrator — run the full pipeline from idea to deployment. Trigger: user wants to build a new product, says "forge", "build product", "create product from idea", or provides a product idea to manufacture.
- forge:init: Initialize a new Forge project with directory structure, config, and state files. Supports --preset (claw|generic). Trigger: "forge init", "initialize forge", "setup forge project".
- forge:brainstorm: P0 — Market research, competitor analysis, feasibility study, business model validation. Trigger: "brainstorm", "research idea", "validate idea", "feasibility analysis".
- forge:prd: P1 — Generate professional PRD with 2 rounds of self-review, push to GitHub for approval. Trigger: "write prd", "product requirements", "requirements document".
- forge:architect: P2 — Technical architecture design + development plan with security/performance self-review. Trigger: "architect", "technical design", "system architecture", "tech stack".
- forge:design: P3 — Visual design system, component specs, page layouts. Trigger: "design system", "ui design", "visual design", "design spec".
- forge:develop: P4 — Multi-agent parallel development following the dev plan with checkpoints. Trigger: "start development", "implement", "build code".
- forge:test: P5 — Automated testing (unit, integration, E2E with browser screenshots). Trigger: "run tests", "test suite", "automated testing".
- forge:verify: P6 — 3-round multi-agent verification with parallel fixes. Trigger: "verify", "review", "quality check", "final review".
- forge:deploy: P7 — Automated deployment (git, database, hosting, DNS, analytics, knowledge base). Trigger: "deploy", "go live", "ship it", "launch".
- forge:resume: Resume an interrupted Forge pipeline from the last checkpoint. Trigger: "resume forge", "continue forge", "pick up where we left off".
