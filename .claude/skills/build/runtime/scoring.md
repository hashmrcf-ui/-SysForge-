# SysForge Architectural Scoring

# PURPOSE
To quantitatively evaluate phase outputs against Enterprise SaaS standards. If the score falls below a critical threshold, the phase fails and triggers Recovery.

# SCORING DIMENSIONS (0-10)

## 1. Security Score
- **10/10:** Zero hardcoded secrets, explicit HttpOnly JWTs, active RLS on all tenant data.
- **< 7 (FAIL):** Missing `.env` variables, weak auth implementations.

## 2. Scalability Score
- **10/10:** Uses connection pooling, pagination implemented on list endpoints, stateless API.
- **< 6 (FAIL):** N+1 queries detected in design, no pagination.

## 3. Modularity Score
- **10/10:** Strict 3-files-per-feature rule followed. Zero business logic in UI components.
- **< 8 (FAIL):** Direct DB queries in React components (Spaghetti Code).

## 4. AI-Readiness Score
- **10/10:** AI requests are tenant-scoped, rate-limited, and auditable.
- **< 7 (FAIL):** Unbounded AI context windows, cross-tenant data leakage risk in prompts.

## 5. Maintainability Score
- **10/10:** Comprehensive inline documentation, standard directory structures.

# THRESHOLDS
- **Passing Phase:** Average score must be >= 8.5/10.
- **Critical Fail:** Any single dimension scoring < 7/10 immediately halts the phase.

# EXECUTION
Scores are calculated internally during the `VALIDATING` state. The final scorecard is presented to the user during the `LOCKED` (Approval) state.
