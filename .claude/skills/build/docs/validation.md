# SysForge Validation Engine

Validation is the gatekeeper of quality in SysForge. It ensures no insecure or structurally flawed code reaches production.

## Global Validation Rules (Zero Tolerance)
-   **No Hardcoded Secrets:** Any API key, JWT secret, or database password found in source code causes an immediate failure. All secrets MUST use `.env`.
-   **No Direct DB Access in UI:** React/Next.js client components cannot contain SQL or ORM calls. They must use the API.
-   **Tenant Context Mandatory:** All backend queries must include a tenant filter or rely on RLS.

## Phase-Specific Validation Checklists

### SysArch Validation
-   [ ] Does the Domain Model cover all required entities?
-   [ ] Is the Multi-tenant strategy explicitly chosen (RLS/Schema/DB)?
-   [ ] Are Role-Based Access Control (RBAC) levels defined?

### SysCore Validation
-   [ ] Is Row-Level Security (RLS) enabled on multi-tenant tables?
-   [ ] Are auth tokens stored securely (HttpOnly cookies preferred over localStorage)?
-   [ ] Does the middleware successfully extract and apply the `tenant_id`?

### SysWeb Validation
-   [ ] Does the feature strictly use 3 core files (Model, Router, Page)?
-   [ ] Are AI endpoints rate-limited?
-   [ ] Is user input sanitized before hitting the database or AI context?
