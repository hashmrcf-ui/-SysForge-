# SysForge Approval Gates

# PURPOSE
To enforce Human-in-the-Loop (HITL) checkpoints. SysForge is an AI Operating System, but it does not run unchecked. Approval Gates prevent massive cascading failures caused by early architectural hallucinations.

# GATING MECHANISM
When the State Machine hits `LOCKED`, SysForge must halt all autonomous action and present a summary to the user.

# MANDATORY GATES

## Gate 1: Phase 0/1 Alignment
- **Requires:** User confirms the system boundaries, AI role, and budget constraints.
- **If Rejected:** Redo Q&A.

## Gate 2: Stack Selection (Phase 4)
- **Requires:** User selects Stack A, B, or C.
- **If Rejected:** Present alternatives or custom stack.

## Gate 3: Architecture Approval (Phase 4.5)
- **THE MOST CRITICAL GATE.**
- **Requires:** User must explicitly type "Approve" after reviewing Domain Model, Tenant Strategy, and AI Contract.
- **If Rejected:** Trigger Recovery; rewrite `architecture.md` based on feedback.

## Gate 4: Prototype Approval (Phase 6)
- **Requires:** Visual sign-off on HTML wireframes. Prevents massive React refactoring later.
- **If Rejected:** Adjust UI layout.

## Gate 5: Feature Loop (Phase 7B)
- **Requires:** Approval after EVERY discrete feature module is built (e.g., "Invoices module done. Approve?").
- **If Rejected:** Trigger Recovery to fix the specific module.

# RULES FOR PRESENTING GATES
1. Be concise. Do not dump code.
2. Present the Architectural Scorecard.
3. List 1-2 key risks from the Devil protocol.
4. End with a clear Call to Action: "Type 'Approve' to proceed to [Next Phase], or state adjustments."
