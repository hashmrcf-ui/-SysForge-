# SysForge Project Memory System

# PURPOSE
To track project evolution, context, and architectural decisions permanently. This ensures SysForge operates as an "AI CTO" that understands *why* decisions were made, not just *what* the current code looks like.

# STORAGE LOCATION
All memory is stored in the user's workspace at `project/context/`.

# MEMORY DOMAINS

## 1. Architectural Log (`architecture_decisions.md`)
Logs every major decision, including rejected alternatives.
**Format:**
- **Decision:** [e.g., Use RLS for Multi-tenancy]
- **Context:** [Why was this needed?]
- **Considered Alternatives:** [e.g., Schema-per-tenant]
- **Why Rejected:** [e.g., High migration complexity for our current scale]

## 2. Risk & Assumption Register (`risks.md`)
Tracks unmitigated risks and scaling assumptions.
**Format:**
- **Risk:** [e.g., AI Rate Limits hitting quota]
- **Impact:** [High/Med/Low]
- **Mitigation Plan:** [e.g., Fallback to Haiku model if Opus rate-limits]

## 3. Workflow State (`state.json`)
Tracks the current phase and exact state of the State Machine.
**Format:**
```json
{
  "current_phase": 4.5,
  "phase_name": "ARCHITECT",
  "state": "LOCKED",
  "last_updated": "2026-05-13T10:00:00Z"
}
```

# RULES
1. **Append, Don't Erase:** When decisions change, append an update. Never delete historical context. SysForge needs to know the evolution path.
2. **Context Evolution:** Before generating code in SysWeb, the Agent must read the `architecture_decisions.md` to ensure code aligns with historical decisions.
