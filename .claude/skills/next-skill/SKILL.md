---
name: sysweb
description: SysForge Web — SysForge Protocol Phase 7B. Builds feature modules and AI integrations using Next.js and FastAPI. Triggers on "/sysweb", "/next-skill".
---

# SysWeb — Feature & AI Module Builder

# PURPOSE
To iteratively build business features and AI capabilities on top of the SysCore foundation. It enforces a strict "3-files-per-feature" modular architecture and ensures AI integrations are secure, performant, and tenant-aware.

# INPUTS
- `architecture.md` (Data models and API contracts).
- Target feature description.

# REQUIRED OUTPUTS
- Database Model updates.
- API Endpoints (CRUD + AI).
- Frontend Components and Pages.

# RULES
1. **State Machine Awareness:** You are executing within the `ACTIVE` state of Phase 7B.
2. **Anti-Spaghetti (Strict):**
   - Max 3 core files per feature (Model, Router, Page).
   - UI components MUST NOT contain direct database queries or complex business logic.
3. **AI-Native Constraints:**
   - **Tenant-Aware AI:** AI requests must be scoped strictly to the current `tenant_id`.
   - **Rate Limiting:** AI endpoints must implement rate limiting to prevent abuse.
   - **Streaming:** Heavy AI responses must stream to the client.
   - **Cost Monitoring & Audit:** All AI interactions must log usage metrics for auditing.

# VALIDATION
Upon completion of a feature module, transition to `VALIDATING` state. The Runtime Engine will use `build/protocols/validate.md` to ensure:
- The new feature respects existing RLS policies.
- AI endpoints do not leak cross-tenant data.
- UI components are pure and use proper state management.
- Fallback models are defined for AI endpoints in case of primary model failure.

# STOP CONDITION
Stop when the specific feature module is fully implemented. Signal the Runtime Engine to transition to `VALIDATING`.

# FAILURE HANDLING
If an AI integration fails due to context limits or timeouts, transition to `FAILED`. Trigger `runtime/recovery.md` to implement chunking or fallback models without destroying the valid UI components.

# NEXT PHASE
Return control to the SysForge Runtime Engine. The system will enter the `LOCKED` Approval Gate for the completed feature before looping back.

---

## THE SYSWEB PROCESS

For every feature module requested, execute the following:

### Step 1: Model & Migrations
1. Define the SQLAlchemy/Prisma model.
2. Ensure `tenant_id` is included and indexed.
3. Generate and apply migration.

### Step 2: Secure API Endpoints
1. Implement the Router.
2. Apply the Tenant Middleware and Permission Guards established in SysCore.
3. Implement pagination for list endpoints to prevent N+1 and memory issues.

### Step 3: AI Integration (If Applicable)
1. Implement the AI logic.
2. Enforce Tenant Isolation in the context window.
3. Implement Streaming for real-time feedback.
4. Log the interaction to the Audit table.

### Step 4: Frontend UI
1. Build the Page, Table, and Form components.
2. Ensure proper RTL support (`dir="rtl"`, `text-right`).
3. Connect to the API using the established API Client.

### Step 5: Verification
1. Run local tests to ensure the feature operates correctly within the multi-tenant context.
