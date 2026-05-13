# 🏗️ SysForge — AI-Native SaaS Operating System

> **The AI CTO for Agentic Coding Systems**  
> SysForge is not a coding assistant. It is a protocol-driven Runtime Engine that orchestrates the construction of Enterprise-Grade SaaS systems.

---

## What is SysForge?

SysForge shifts the paradigm from **"instruction-based generation"** (spaghetti code) to **"protocol-driven system orchestration"** (enterprise architecture). 

It enforces a rigid 10-Phase Protocol governed by a **Runtime Intelligence Layer**. It tracks state, scores architecture, manages project memory, and automatically handles failure recovery. 

```bash
/sysforge [System Name]  # Initializes the Runtime Engine
```

---

## 🧠 The Runtime Intelligence Layer (`/runtime`)

SysForge operates via a strict internal engine (`build/runtime/engine.md`) that wraps all execution in a deterministic state machine.

### Core Runtime Capabilities
1. **State Machine (`state-machine.md`):** Execution flows strictly through `PENDING` → `ACTIVE` → `VALIDATING` → `LOCKED` → `APPROVED` → `COMPLETED`. No skipping allowed.
2. **Project Memory (`memory.md`):** Automatically persists architectural decisions, rejected alternatives, and context in `project/context/`.
3. **Dependency-Aware Validation (`validate.md`):** A validation graph that blocks downstream generation if upstream contracts (e.g., Auth, RLS) are violated.
4. **Failure Recovery (`recovery.md`):** If generation fails, SysForge isolates the failure, preserves valid outputs, and regenerates *only* the broken node. It never restarts the pipeline blindly.
5. **Architectural Scoring (`scoring.md`):** Every phase is scored on Security, Scalability, Modularity, and AI-Readiness. Sub-standard generation triggers a halt.

---

## 🧭 Architecture Workflow

```ascii
                      +-------------------+
                      |   USER REQUEST    |
                      +-------------------+
                               |
                      [ SysForge Runtime ]
                 (State Machine & Memory Engine)
                               |
        +----------------------+----------------------+
        |                      |                      |
[ SysArch ]               [ SysCore ]            [ SysWeb ]
(Architecture)         (Infrastructure)      (Feature Modules)
- Domain Models        - DB Foundation       - Business Logic
- Multi-tenant         - Strict RLS          - UI Components
- Security/Auth        - Auth Middleware     - AI Integration
```

### The Engine Loop
`READ STATE` → `EVALUATE` → `DELEGATE` → `VALIDATE & SCORE` → `LOCK` → `APPROVAL GATE` → `ADVANCE`

---

## 🚀 The 10-Phase Master Protocol

```text
PHASE 0   IDENTIFY    Identify core business domain.
PHASE 1   ALIGN       Resolve ambiguity via 5 critical scale/budget questions.
PHASE 2   COORDINATE  Initialize memory (context.md, state.json).
PHASE 3   SCOUT       Evaluate Open Source alternatives vs. Custom Build.
PHASE 4   BURST       Select technology stack (A/B/C).

PHASE 4.5 ARCHITECT   Delegated to SysArch. Output: architecture.md
          ├── Domain Model & Multi-tenant Strategy
          └── Auth, Permissions & AI Constraints

PHASE 5   DEVIL       Red Team attack on architecture.
PHASE 6   PROTOTYPE   HTML-only wireframes for visual sign-off.

PHASE 7A  CORE ENGINE Delegated to SysCore. Build Foundation.
          ├── Strict Row-Level Security (RLS) & DB Foundation
          └── Auth System, Middleware, Permission Guard

PHASE 7B  FEATURES    Delegated to SysWeb. Feature Loop.
          └── Enforce 3-files-per-feature rule (Model, Router, Page).

PHASE 8   VERIFY      Final security and architecture graph audit.
PHASE 9   SHIP        Deploy and update intelligence templates.
```

---

## 🛡️ Enterprise SaaS Philosophy

1. **Validation Engine:** Progression between phases is locked behind the Validation Graph. Zero tolerance for hardcoded secrets.
2. **AI-Native Constraints:** AI endpoints must be tenant-aware, rate-limited, and auditable.
3. **Anti-Spaghetti:** No direct DB calls in UI components. Max 3 files per feature.
4. **Human-in-the-Loop (HITL):** The system locks at critical gates (e.g., Architecture generation) requiring explicit human approval before continuing.

---

## 📚 Documentation

Refer to the `build/runtime/` and `docs/` folders for in-depth protocol specifications.

---

## ⚙️ Installation

### Global Install (Recommended for Claude Code)

```bash
git clone https://github.com/hashmrcf-ui/-SysForge-.git
cp -r ./-SysForge-/.claude/skills/* ~/.claude/skills/
```

**Windows (PowerShell):**
```powershell
git clone https://github.com/hashmrcf-ui/-SysForge-.git
Copy-Item -Recurse .\-SysForge-\.claude\skills\* "$env:USERPROFILE\.claude\skills\"
```

---

## 🎯 Usage

```bash
# Start the Runtime Engine
/sysforge Enterprise HR Management System
```

---

<div align="center">

**SysForge — Build Systems that Last.**

</div>
