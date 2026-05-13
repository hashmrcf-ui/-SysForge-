---
name: sysforge
description: SysForge — AI-Native SaaS Operating System. Orchestrates the end-to-end building of Enterprise-Grade SaaS systems through a strict 10-phase protocol. Triggers on "/sysforge", "/sysforge resume", "/sysforge status", "/sysforge scope".
---

# SysForge — AI-Native SaaS Operating System

# PURPOSE
SysForge is the master orchestrator for building Enterprise-Grade SaaS systems. It enforces a strict 10-phase protocol to guarantee architectural integrity, security, and scalability. It prevents spaghetti code by mandating design before implementation and enforcing validation gates between phases.

# INPUTS
- User prompt describing the SaaS system (e.g., `/sysforge Accounting System`).
- Architecture blueprints (`systems/*.md`).
- Context from previous sessions (`templates/context.md`).
- Output from sub-skills (`/sysarch`, `/syscore`, `/sysweb`).

# REQUIRED OUTPUTS
- A fully functional, production-ready SaaS application.
- Comprehensive architectural documentation (`architecture.md`).
- System state tracking (`context.md`).
- Audit logs and session history (`session-log.md`).

# RULES
## 1. Anti-Spaghetti Rules
- **No business logic in UI:** Components only consume data and emit events.
- **No direct DB calls in components:** All data access must go through the API layer.
- **No shared mutable state:** Use proper state management patterns (e.g., Zustand).
- **No hidden dependencies:** All dependencies must be explicitly defined in architectural docs.
- **No random utility dumping:** Group utilities by domain (e.g., `utils/auth.ts`, `utils/date.ts`).

## 2. Phase Locking
- You MUST NOT proceed to the next phase until the `VALIDATE` step is 100% complete and verified.
- You MUST wait for explicit user approval before crossing a phase boundary.
- If a critical error occurs, execution halts immediately.

## 3. Protocol Architecture Execution
Every phase MUST follow this execution flow strictly:
`INPUT` → `DECISION` → `GENERATE` → `VALIDATE` → `STOP` → `WAIT FOR APPROVAL`

# VALIDATION
- A global Validation Engine acts as a gatekeeper.
- Execute the checklist specified in `protocols/validate.md` at the end of each phase.
- Missing files, security flaws (e.g., hardcoded secrets), or architecture violations block progression.

# STOP CONDITION
SysForge stops when:
1. A validation gate fails.
2. User approval is required to proceed.
3. The 10-phase protocol completes successfully.

# FAILURE HANDLING
- If a phase fails validation, rollback any destructive changes.
- Clearly present the error to the user with actionable steps.
- Do not attempt to "guess" fixes for architectural violations; ask the user for clarification.

# NEXT PHASE
The orchestrator dictates the flow through the following 10 phases. Consult the specific protocol file for details on execution.

---

## THE 10-PHASE PROTOCOL

### PHASE 0: IDENTIFY
- **INPUT:** User request.
- **GENERATE:** Analyze the request. If ambiguous, ask clarifying questions.
- **VALIDATE:** Is the core business domain clear?
- **STOP/WAIT:** Ask user to confirm the identified domain.

### PHASE 1: ALIGN
- **INPUT:** Identified domain.
- **GENERATE:** Present 5 critical questions regarding scale, users, AI agent role, integration, and budget.
- **VALIDATE:** Are all 5 questions answered?
- **STOP/WAIT:** Wait for user responses.

### PHASE 2: COORDINATE
- **INPUT:** User answers from Phase 1.
- **GENERATE:** Read `protocols/coordinate.md`. Initialize `context.md` and `session-log.md` using templates.
- **VALIDATE:** Are the context files created and populated?
- **STOP/WAIT:** Ask user to confirm project initialization.

### PHASE 3: SCOUT
- **INPUT:** `context.md`.
- **GENERATE:** Read `sources/open-source.md`. Propose existing open-source solutions to avoid reinventing the wheel.
- **VALIDATE:** Did we check for existing solutions?
- **STOP/WAIT:** Ask user if they want to use an open-source base or build custom (A/B/C).

### PHASE 4: BURST (Stack Selection)
- **INPUT:** User decision from Phase 3.
- **GENERATE:** Present architecture options A, B, and C (from `stacks/option-*.md`).
- **VALIDATE:** Is a valid stack selected?
- **STOP/WAIT:** Wait for stack selection.

### PHASE 4.5: ARCHITECT
- **INPUT:** Selected stack.
- **GENERATE:** Trigger `/sysarch` (or read `../saas-architect/SKILL.md`).
- **VALIDATE:** Is `architecture.md` generated and complete? Does it include RLS, Auth, Domain Model, and AI design?
- **STOP/WAIT:** Wait for user sign-off on `architecture.md`.

### PHASE 5: DEVIL (Red Teaming)
- **INPUT:** `architecture.md` and `context.md`.
- **GENERATE:** Read `protocols/devil.md`. Attack the architecture. Identify single points of failure, scalability bottlenecks, and AI integration risks.
- **VALIDATE:** Has the red-team report been generated?
- **STOP/WAIT:** Present risks. Ask user if they want to adjust the architecture or proceed with accepted risks.

### PHASE 6: PROTOTYPE
- **INPUT:** Approved `architecture.md`.
- **GENERATE:** Build 5 HTML-only wireframes (no React/complex JS).
- **VALIDATE:** Do the wireframes cover the core user flows?
- **STOP/WAIT:** Ask user to approve the visual layout.

### PHASE 7A: CORE ENGINE
- **INPUT:** Approved wireframes and `architecture.md`.
- **GENERATE:** Trigger `/syscore` (or read `../saas-core/SKILL.md`). Build the 6 foundational layers (DB+RLS, Auth, Multi-tenant, Permission Guard, Base API, Frontend Shell).
- **VALIDATE:** Read `protocols/validate.md` and run the security and core engine checklist. NO hardcoded secrets allowed.
- **STOP/WAIT:** Ask user to verify the Core Engine deployment.

### PHASE 7B: FEATURE LOOP
- **INPUT:** Deployed Core Engine.
- **GENERATE:** Trigger `/sysweb` (or read `../next-skill/SKILL.md`). Iterate through features based on `architecture.md`. Max 3 files per feature.
- **VALIDATE:** Does the feature work? Is it secure?
- **STOP/WAIT:** Wait for user approval after each feature module.

### PHASE 8: VERIFY
- **INPUT:** Completed application.
- **GENERATE:** Run a final security and architecture audit.
- **VALIDATE:** Does the system meet all requirements in `architecture.md`?
- **STOP/WAIT:** Present final audit report.

### PHASE 9: SHIP
- **INPUT:** Audited application.
- **GENERATE:** Provide deployment instructions. Read `protocols/calibrate.md` to update templates based on lessons learned.
- **VALIDATE:** Is deployment successful?
- **STOP/WAIT:** Project complete.
