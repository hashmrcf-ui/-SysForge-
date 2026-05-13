# SysForge State Machine

# PURPOSE
To enforce a rigid lifecycle for every project phase. This eliminates phase-skipping, uncontrolled generation, and hidden failures.

# STATES
Every Phase in the 10-Phase Protocol exists in exactly one of these states at any time:

1. **PENDING:** Phase is queued but execution has not begun.
2. **ACTIVE:** The phase is currently generating output or gathering information.
3. **VALIDATING:** Output is generated and currently being processed by the Validation Graph and Scoring Engine.
4. **FAILED:** Validation or Scoring failed. System is halted or executing Recovery.
5. **LOCKED:** Validation passed. State is frozen awaiting Human-in-the-Loop (HITL) approval.
6. **APPROVED:** User has explicitly signed off.
7. **COMPLETED:** State is finalized. Post-phase memory updates are written. Ready for N+1.

# LEGAL TRANSITIONS
Transitions are strictly enforced. Deviations are considered fatal errors.

- `PENDING` → `ACTIVE`
- `ACTIVE` → `VALIDATING`
- `VALIDATING` → `FAILED` (If validation/scoring < threshold)
- `VALIDATING` → `LOCKED` (If validation/scoring pass)
- `FAILED` → `ACTIVE` (Triggered by Recovery Protocol, regenerates specific section)
- `LOCKED` → `APPROVED` (User types "Yes/Approve")
- `LOCKED` → `FAILED` (User rejects the output)
- `APPROVED` → `COMPLETED`
- `COMPLETED` → Phase N+1 `PENDING`

# PERSISTENCE
The current state of the machine must be updated in `project/context/state.json` (or the equivalent context file) at every state transition.
