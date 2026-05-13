# SysForge Architecture

SysForge is an AI-Native SaaS Operating System designed to orchestrate the construction of complex, multi-tenant enterprise applications.

## High-Level Architecture
The system employs a multi-agent orchestration model driven by Claude Code.

1.  **SysForge Orchestrator (`/sysforge`):** The primary brain. It manages state, reads context, and delegates tasks to specific sub-skills based on the 10-Phase Protocol.
2.  **SysArch (`/sysarch`):** The Domain Architect. It operates strictly in the conceptual space, defining data models, security boundaries, and API contracts. It produces the `architecture.md` blueprint.
3.  **SysCore (`/syscore`):** The Infrastructure Engineer. It consumes `architecture.md` to build the rigid 6-layer foundation: DB, RLS, Auth, Middleware, Permissions, and the Base API.
4.  **SysWeb (`/sysweb`):** The Feature Developer. It iteratively builds business logic and UI components on top of the SysCore foundation, strictly adhering to the 3-files-per-feature rule.

## Data Flow
`User Request` -> `SysForge (Orchestration)` -> `SysArch (Blueprint)` -> `SysCore (Foundation)` -> `SysWeb (Features)` -> `Final Application`.

All state is persisted in `context.md` and `session-log.md` to survive context window limits and session restarts.
