# SysForge Security Guidelines

SysForge applications are designed to be Enterprise-Grade. Security is not an afterthought; it is baked into SysCore.

## 1. Multi-Tenant Data Isolation
-   **Default Strategy:** PostgreSQL Row-Level Security (RLS).
-   **Implementation:** Every table containing tenant data must have `tenant_id`. RLS policies must enforce that users can only select/insert/update/delete rows where `tenant_id` matches their authenticated context.
-   **Middleware:** The application middleware extracts the `tenant_id` from the secure session and sets it in the database connection context (e.g., `set_config('app.tenant_id', ...)`).

## 2. Authentication & Session Management
-   **JWT Handling:** JWTs must not be stored in `localStorage` due to XSS vulnerability. Store access and refresh tokens in `HttpOnly`, `Secure`, `SameSite=Strict` cookies.
-   **Rotation:** Implement refresh token rotation. When a refresh token is used, issue a new one and invalidate the old one.

## 3. Authorization (RBAC)
-   Enforce permissions at the API route level using decorators or middleware (e.g., `@require_role("tenant_admin")`). Do not rely solely on UI hiding.

## 4. API Security
-   Implement Rate Limiting on all public and AI endpoints.
-   Validate all incoming requests using strict schema validation (e.g., Zod, Pydantic).
