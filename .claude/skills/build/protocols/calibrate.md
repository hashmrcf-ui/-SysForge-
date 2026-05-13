# SysForge Protocol: Calibrate

# PURPOSE
To act as the post-project improvement engine. It analyzes the development journey, identifies friction points, and updates templates to prevent repeating mistakes. Replaces the external `/calibrate` ghost command.

# INPUTS
- `session-log.md` (The full history of the build).
- Final application state.

# REQUIRED OUTPUTS
- Recommended updates to templates (e.g., `context.md`, `architecture.md`).
- Post-Mortem summary.

# RULES
1. **Focus on Systemic Issues:** Look for errors that required multiple prompts to fix.
2. **Template Evolution:** Suggest concrete changes to templates, not abstract advice.

# VALIDATION
Ensure the Post-Mortem covers:
- Time spent vs. expected.
- Most frequent error types.
- AI hallucination rate.

# STOP CONDITION
Stop when the Post-Mortem is presented and template updates are proposed.

# FAILURE HANDLING
If `session-log.md` is empty or missing, skip calibration.

# NEXT PHASE
End of SysForge process.
