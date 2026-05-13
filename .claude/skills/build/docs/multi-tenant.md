# SysForge Multi-Tenant Strategies

SysForge supports three levels of multi-tenancy. Phase 4.5 (`/sysarch`) must explicitly select one.

## Level 1: Row-Level Security (RLS) - Recommended
- **Architecture:** All tenants share the same database and schemas.
- **Isolation:** Enforced via PostgreSQL RLS policies. Every table has a `tenant_id`.
- **Pros:** Lowest infrastructure cost, simplest migrations.
- **Cons:** Risk of misconfigured RLS leaking data.

## Level 2: Schema-per-Tenant
- **Architecture:** Single database, but each tenant gets a dedicated schema (e.g., `tenant_acme.users`).
- **Isolation:** Enforced by setting the search path (e.g., `SET search_path TO tenant_acme`).
- **Pros:** Better isolation, easier to backup a single tenant.
- **Cons:** Migrations become complex (must run N times).

## Level 3: Database-per-Tenant
- **Architecture:** Dedicated database instance per tenant.
- **Isolation:** Absolute physical isolation.
- **Pros:** Highest security, required for certain compliance standards (HIPAA, SOC2).
- **Cons:** Highest cost, extremely complex infrastructure management.
