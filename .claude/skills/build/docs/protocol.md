# SysForge Execution Protocol

Every phase within SysForge, and every interaction within sub-skills, MUST adhere to the following rigid protocol to prevent hallucinations and spaghetti code.

## The 6-Step Loop

1.  **INPUT:** The agent receives the trigger command, user request, or output from a previous phase. It reads relevant `context.md` or `architecture.md` files.
2.  **DECISION:** The agent analyzes the input against the phase requirements. It plans the execution steps internally.
3.  **GENERATE:** The agent performs the required action (e.g., asking a question, generating a schema, writing a component).
4.  **VALIDATE:** The agent self-evaluates the output against the `build/protocols/validate.md` checklist.
    *   *If validation fails:* Discard output, adjust approach, retry.
    *   *If validation passes:* Proceed.
5.  **STOP:** The agent explicitly halts execution to prevent runaway generation.
6.  **WAIT FOR APPROVAL:** The agent presents the validated output to the user and requires explicit confirmation before transitioning to the next phase or state.

## Phase Locking
Transitioning between the 10 core phases is locked. The orchestrator cannot move from Phase N to Phase N+1 without completing the `WAIT FOR APPROVAL` step.
