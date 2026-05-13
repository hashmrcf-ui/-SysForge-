---
name: sysforge
description: SysForge — AI-Native SaaS Operating System. Orchestrates the end-to-end building of Enterprise-Grade SaaS systems via a Runtime Intelligence Layer. Triggers on "/sysforge", "/sysforge resume", "/sysforge status", "/sysforge scope".
---

# SysForge — AI-Native SaaS Operating System

# PURPOSE
SysForge is an AI CTO Operating System. It no longer operates via unstructured generation. Instead, it acts as a **Runtime Intelligence Layer**, managing a strict State Machine, Project Memory, Validation Graphs, and Failure Recovery protocols.

# INPUTS
- User prompt describing the SaaS system (e.g., `/sysforge Accounting System`).
- Current State from `project/context/state.json`.

# REQUIRED OUTPUTS
- A fully functional, production-ready SaaS application.
- Initialized Project Memory (`project/context/`).

# RULES
## 1. Runtime Handoff
SysForge does NOT execute the 10-Phase Protocol directly. Immediately upon invocation, it MUST hand over control to the Runtime Engine.
- **Action:** Read `runtime/engine.md` and execute its `EXECUTION LOOP`.

## 2. State Machine Supremacy
- Do not attempt to bypass the State Machine (`runtime/state-machine.md`). 
- Transitions must be strictly followed (`PENDING` → `ACTIVE` → `VALIDATING` → `LOCKED` → `APPROVED` → `COMPLETED`).

## 3. Protocol Architecture
Do not generate code without architecture approval. Rely entirely on the internal protocols (`saas-architect`, `saas-core`, `next-skill`) governed by the Runtime Engine.

# VALIDATION
- Governed entirely by `runtime/engine.md` which invokes `protocols/validate.md` (Dependency Graph) and `runtime/scoring.md`.

# STOP CONDITION
- Stop and wait for user input when the State Machine reaches `LOCKED` (Approval Gate).
- Stop if the State Machine reaches a Fatal Failure in `runtime/recovery.md`.

# FAILURE HANDLING
- Delegated to `runtime/recovery.md`. The Orchestrator does not guess solutions; it isolates the failure and attempts targeted rollback/regeneration.

# NEXT PHASE
- Read `runtime/engine.md` to begin.
