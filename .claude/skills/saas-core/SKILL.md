---
name: syscore
description: SysForge Core — SysForge Protocol Phase 7A. Builds the secure, 6-layer foundational Core Engine. Triggers on "/syscore", "/saas-core".
---

# SysCore — Secure Core Engine Builder

# PURPOSE
To build the foundational engine upon which all future features rely. This engine handles the complexities of Multi-tenancy, Authentication, Authorization, and Base Routing so that subsequent feature modules remain clean and focused solely on business logic.

# INPUTS
- `architecture.md` (Approved architecture).
- Selected stack option (e.g., Next.js + FastAPI).

# REQUIRED OUTPUTS
- Configured Database with Base Tables (`tenants`, `users`).
- Row-Level Security (RLS) implementation.
- Authentication endpoints and Middleware.
- Next.js Protected Layout Shell.

# RULES
1. **Security First:**
   - **NO hardcoded secrets.** All keys must use `.env`.
   - **NO localStorage for sensitive tokens.** Use HttpOnly cookies or secure token passing mechanisms.
   - **Refresh Token Rotation:** Implement proper rotation.
2. **Tenant Isolation:** Every query must automatically filter by `tenant_id` via middleware.
3. **Role Enforcement:** All protected routes must enforce Role-Based Access Control (RBAC).

# VALIDATION
Use `build/protocols/validate.md` to ensure:
- RLS policies are active on the `users` table.
- Auth middleware correctly intercepts and parses tokens.
- Frontend login flow successfully establishes a session.
- No sensitive data is exposed in the frontend shell.

# STOP CONDITION
Stop when all 6 layers of the Core Engine are implemented and verified via a local build check.

# FAILURE HANDLING
If database migrations fail, rollback the migration, explain the SQL error to the user, and propose a fix. Do not proceed until migrations apply cleanly.

# NEXT PHASE
Return control to SysForge. Execution proceeds to Phase 7B (FEATURE LOOP).

---

## THE SYSCORE PROCESS (6 LAYERS)

Implement these layers sequentially.

### Layer 1: Secure DB Foundation
1. Create `tenants` and `users` tables.
2. Implement strict RLS. Example (PostgreSQL):
```sql
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation_users ON users
  USING (tenant_id = current_setting('app.tenant_id')::UUID);
```

### Layer 2: Secure Auth System
1. Implement JWT creation and validation.
2. Ensure secrets are loaded via environment variables ONLY.
3. Implement `login`, `refresh`, and `me` endpoints.

### Layer 3: Multi-Tenant Middleware
1. Implement middleware that extracts the tenant ID from the authenticated session.
2. Inject the tenant ID into the database connection context for RLS.

### Layer 4: Permission Guard
1. Implement RBAC decorators/hooks (e.g., `@require_role('tenant_admin')`).

### Layer 5: Base API Structure
1. Setup FastAPI (or Next.js API) with CORS, global error handling, and a standard response format.

### Layer 6: Frontend Shell
1. Create the base Layout with Sidebar and Header.
2. Implement the API client with automatic token attachment and refresh logic.
