---
name: sysarch
description: SysForge Architect — SysForge Protocol Phase 4.5. Designs Enterprise-Grade Core Architecture before any code is written. Triggers on "/sysarch", "/saas-architect".
---

# SysArch — Enterprise Architecture Design

# PURPOSE
To define the complete architectural blueprint before a single line of code is written. This mitigates 80% of project failures by forcing decisions on Domain Models, Multi-tenancy, Auth, and AI Integration upfront.

# INPUTS
- `context.md` (System type, scale, AI role).
- Selected stack option (A, B, or C).

# REQUIRED OUTPUTS
- `architecture.md` containing the finalized architectural decisions.

# RULES
1. **No Code Rule:** Absolutely no code generation in this phase.
2. **Mandatory Tenant Strategy:** Never assume a multi-tenant strategy. Explicitly evaluate RLS vs. Schema vs. Database isolation based on scaling needs.
3. **AI Native:** The AI Agent is a first-class citizen. Its role and tools must be defined in the architecture.

# VALIDATION
Use `build/protocols/validate.md` to ensure:
- Domain model covers all required entities.
- Multi-tenant strategy is explicitly documented.
- Role-based permissions (Super Admin, Tenant Admin, User) are defined.
- API Contract covers basic CRUD and AI interactions.

# STOP CONDITION
Stop when `architecture.md` is generated and fully populated.

# FAILURE HANDLING
If the user's requirements are contradictory (e.g., extreme data isolation but low budget), explain the tradeoff and ask for a definitive choice before generating the architecture.

# NEXT PHASE
Return control to SysForge. Execution proceeds to Phase 5 (DEVIL).

---

## THE SYSARCH PROCESS

Execute these steps sequentially.

### STEP 1: Domain Modeling
1. Read `context.md`.
2. Propose the core entities and their relationships (e.g., Tenant (1) -> (*) Users).
3. Wait for user approval.

### STEP 2: Multi-Tenant Strategy
1. Propose isolation levels:
   - **Level 1 (RLS):** Row-Level Security (Default for most SaaS).
   - **Level 2 (Schema):** Schema-per-tenant (High isolation).
   - **Level 3 (Database):** Database-per-tenant (Enterprise compliance).
2. Recommend the best fit based on `context.md`. Wait for approval.

### STEP 3: Authentication & Permissions
1. Define the auth flow (e.g., JWT with HttpOnly cookies).
2. Define the exact Role Hierarchy (e.g., Super Admin -> Tenant Admin -> User).
3. Wait for approval.

### STEP 4: AI Agent Architecture
1. Define the AI's capability (Assistant, Analyst, or Autonomous Agent).
2. Define the exact tools the AI will have access to.
3. Define how the AI respects Tenant Isolation (Context Injection).
4. Wait for approval.

### STEP 5: API Contract
1. Outline the essential endpoints needed to support the Domain Model and AI integrations.
2. Wait for approval.

### STEP 6: Generate Document
1. Compile all approved decisions into `architecture.md` using the standard template.
2. Final Validate.
