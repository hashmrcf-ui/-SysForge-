# SysForge Runtime Engine

# PURPOSE
The Runtime Engine is the intelligence layer of SysForge. It orchestrates the entire SaaS lifecycle, moving control from unstructured generation into a rigid, protocol-driven architecture. 

# RESPONSIBILITIES
1. **State Machine Execution:** Consults `state-machine.md` before transitioning any phase.
2. **Memory Management:** Invocates `memory.md` to persist decisions and context into `project/context/`.
3. **Validation Orchestration:** Triggers the Dependency-Aware Validation Graph (`protocols/validate.md`) upon phase completion.
4. **Failure Recovery:** If validation fails, triggers `recovery.md` to handle partial rollbacks and targeted regeneration.
5. **Architectural Scoring:** Enforces minimum standards via `scoring.md`.

# EXECUTION LOOP
Whenever SysForge starts or resumes, the Engine executes this exact loop:

1. **READ STATE:** Parse `project/context/state.json` (or context memory) to determine current Phase and State (e.g., `PHASE 4: ACTIVE`).
2. **EVALUATE:** Check `state-machine.md` to see what transitions are legal.
3. **DELEGATE:** Pass execution to the specific Phase Protocol (e.g., `/sysarch` or a specific internal generation step).
4. **VALIDATE & SCORE:** Upon return, run `protocols/validate.md` and `scoring.md`.
5. **LOCK:** If scores pass and validation is clean, transition state to `VALIDATING`.
6. **APPROVAL GATE:** Consult `approvals.md` and wait for HITL (Human-in-the-Loop) sign-off.
7. **ADVANCE:** If approved, transition state to `COMPLETED` and move to Phase N+1 `PENDING`.

# RULES
- **No Direct Generation:** The Engine does NOT write application code. It orchestrates sub-skills.
- **Strict Halting:** If the State Machine reports an illegal transition or a Failure Recovery cannot resolve an issue automatically, the Engine halts and demands human intervention.
