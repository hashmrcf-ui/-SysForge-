# SysForge Protocol: Dependency-Aware Validation Graph

# PURPOSE
To serve as the central Validation Engine for SysForge. It enforces security, architecture, and dependency checks before allowing progression to the next phase. It relies on a Dependency Graph, ensuring that downstream components are not built if upstream contracts are violated.

# DEPENDENCY GRAPH RULES
- **Node:** A generated output (e.g., `schema.prisma`, `auth.ts`, `architecture.md`).
- **Edge:** A dependency constraint.
  - Example: `auth.ts` *requires* `schema.prisma` (Specifically, the `users` table).
  - Example: `Phase 7A (SysCore)` *requires* `architecture.md` (Auth Strategy section).

# VALIDATION EXECUTION (STATE: VALIDATING)
When the State Machine enters `VALIDATING`, execute the following checks based on the current Phase.

## Phase 4.5 (SysArch) Graph Check
- **Node: `architecture.md`**
  - Dependency: `context.md` (Must align with business domain).
  - Check: Does it explicitly define an RLS strategy or Schema isolation? (Fail if missing).
  - Check: Are RBAC roles defined? (Fail if missing).
  - Check: Is the AI Agent role and context boundary defined? (Fail if missing).

## Phase 7A (SysCore) Graph Check
- **Node: `schema.prisma` (or SQL migrations)**
  - Dependency: `architecture.md` (Domain Model).
  - Check: Do `tenants` and `users` tables exist? (Fail if missing).
  - Check: Is RLS explicitly enabled on multi-tenant tables? (Fail if missing).
- **Node: `auth.ts` / Security Middleware**
  - Check: ZERO hardcoded secrets? (Scan for raw strings).
  - Check: Are tokens HttpOnly? (Fail if localStorage is used for Access Tokens).
  - Check: Does middleware successfully extract and apply `tenant_id`?

## Phase 7B (SysWeb) Graph Check
- **Node: Feature Route / Component**
  - Dependency: `schema.prisma` (Must reference existing models).
  - Check: Is the 3-files-per-feature rule respected?
  - Check: Are UI components completely free of direct database calls? (Fail if true).
  - Check: Are AI endpoints rate-limited and tenant-aware? (Fail if missing).

# STOP CONDITION
Stop when the graph is fully evaluated.
Output:
1. Pass/Fail Status.
2. A quantitative score via `runtime/scoring.md`.

# FAILURE HANDLING
If FAIL:
1. Transition State Machine to `FAILED`.
2. Output exact broken nodes/edges.
3. Trigger `runtime/recovery.md` to initiate targeted rollback and regeneration.

# NEXT PHASE
If PASS: Transition State Machine to `LOCKED` and await Approval Gate.
