# 🏗️ SysForge — AI-Native SaaS Operating System

> **Production-Grade Systems Engineering for Agentic Coding Systems**  
> From concept to deployment through a strict 10-phase enterprise protocol.

---

## What is SysForge?

**SysForge** is not just a boilerplate; it is an **AI-Native SaaS Operating System** designed for Claude Code, Cursor, and Agentic Coding frameworks. It enforces a rigid architectural discipline to build secure, scalable, multi-tenant enterprise applications (Accounting, HR, Healthcare, POS).

By enforcing "Architecture before Implementation" and strict "Phase Locking", SysForge prevents the spaghetti code and architectural drift commonly produced by unstructured AI coding sessions.

```bash
/sysforge [System Name]  # Initiates the 10-phase master protocol
```

---

## 🧭 Architecture & Workflow

SysForge operates as a multi-agent orchestration model.

```ascii
                      +-------------------+
                      |   USER REQUEST    |
                      +-------------------+
                               |
                      [ SysForge Orchestrator ]
                      (10-Phase Protocol Manager)
                               |
        +----------------------+----------------------+
        |                      |                      |
[ SysArch ]               [ SysCore ]            [ SysWeb ]
(Architecture)         (Infrastructure)      (Feature Modules)
- Domain Models        - DB Foundation       - Business Logic
- Multi-tenant         - Strict RLS          - UI Components
- Security/Auth        - Auth Middleware     - AI Integration
```

### The Phase Protocol Loop
Every phase executes a strict internal protocol before moving forward:
`INPUT` → `DECISION` → `GENERATE` → `VALIDATE` → `STOP` → `WAIT FOR APPROVAL`

---

## 🛠️ The 4 Core Skills

| Skill | Role | When It Runs |
|---|---|---|
| `/sysforge` | **Master Orchestrator.** Manages context, session logging, and phase progression. | Start of project |
| `/sysarch` | **Domain Architect.** Designs the Domain Model, Tenant Strategy, and AI interactions. | Phase 4.5 |
| `/syscore` | **Infrastructure Engineer.** Builds the rigid 6-layer foundation (DB, RLS, Auth). | Phase 7A |
| `/sysweb` | **Feature Developer.** Iteratively builds modular features (max 3 files per feature). | Phase 7B |

---

## 🚀 The 10-Phase Master Protocol

```text
PHASE 0   IDENTIFY    Identify core business domain.
PHASE 1   ALIGN       Resolve ambiguity via 5 critical scale/budget questions.
PHASE 2   COORDINATE  Initialize memory (context.md, session-log.md).
PHASE 3   SCOUT       Evaluate Open Source alternatives vs. Custom Build.
PHASE 4   BURST       Select technology stack (A/B/C).

PHASE 4.5 ARCHITECT   Trigger /sysarch. Output: architecture.md
          ├── Domain Model
          ├── Multi-tenant Strategy (RLS, Schema, DB)
          ├── Auth & Permission Flow (RBAC)
          └── AI Agent Capabilities

PHASE 5   DEVIL       Internal Red Team attack on architecture.
PHASE 6   PROTOTYPE   HTML-only wireframes for visual sign-off.

PHASE 7A  CORE ENGINE Trigger /syscore. Build Foundation.
          ├── DB Foundation (tenants, users)
          ├── Strict Row-Level Security (RLS)
          ├── Auth System (HttpOnly, Rotation)
          ├── Multi-tenant Middleware
          ├── Permission Guard (RBAC)
          └── Base API Structure

PHASE 7B  FEATURES    Trigger /sysweb. Loop through features.
          └── Enforce 3-files-per-feature rule (Model, Router, Page).

PHASE 8   VERIFY      Final security and architecture audit.
PHASE 9   SHIP        Deploy and execute /calibrate for template updates.
```

---

## 🛡️ Enterprise SaaS Philosophy

SysForge enforces strict rules to ensure production readiness:

1. **Validation Engine:** Progression between phases is locked behind a strict Validation Engine. Zero tolerance for hardcoded secrets.
2. **AI-Native Constraints:** AI endpoints must be tenant-aware, rate-limited, and auditable.
3. **Anti-Spaghetti:** No direct DB calls in UI components. No shared mutable state. Max 3 files per feature.
4. **Security First:** RLS is mandatory for multi-tenant data. JWTs belong in HttpOnly cookies, not localStorage.

---

## 📚 Documentation

Refer to the `docs/` folder for in-depth protocol specifications:
- `docs/architecture.md` - Core system design.
- `docs/protocol.md` - The Execution Loop.
- `docs/validation.md` - Phase-locking checklists.
- `docs/security.md` - Guidelines for RLS and Auth.
- `docs/ai-system.md` - Tenant-aware AI integration.
- `docs/multi-tenant.md` - Strategies (RLS vs Schema vs DB).

---

## ⚙️ Installation

### Global Install (Recommended for Claude Code)

```bash
# Clone the repository
git clone https://github.com/hashmrcf-ui/-SysForge-.git

# Install globally to your Claude environment
cp -r ./-SysForge-/.claude/skills/* ~/.claude/skills/
```

**Windows (PowerShell):**
```powershell
git clone https://github.com/hashmrcf-ui/-SysForge-.git
Copy-Item -Recurse .\-SysForge-\.claude\skills\* "$env:USERPROFILE\.claude\skills\"
```

### Local Project Install
Copy the `.claude` folder directly into your project root to run locally.

---

## 🎯 Usage

To start a new project or resume an existing one:

```bash
# Start the Master Orchestrator
/sysforge [System Description]

# Example:
/sysforge Enterprise HR Management System
```

Other commands:
- `/sysforge resume` - Resume from `context.md`
- `/sysforge status` - Print current phase and validation state
- `/sysforge scope` - Adjust project scope

---

<div align="center">

**SysForge — Build Systems that Last.**

</div>
