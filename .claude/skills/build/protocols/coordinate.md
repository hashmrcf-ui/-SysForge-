# SysForge Protocol: Coordinate

# PURPOSE
To establish the project memory, tracking, and contextual foundation before any code or design begins. This replaces the external `/coordinate` ghost command.

# INPUTS
- User's answers from Phase 1 (ALIGN).
- Identified system type (e.g., Accounting, HR).

# REQUIRED OUTPUTS
- `context.md` (Project Context and State).
- `session-log.md` (Audit and Session History).
- Base project directory.

# RULES
1. **Never overwrite without asking:** If `context.md` already exists, prompt for resume or overwrite.
2. **Immutable history:** `session-log.md` is append-only.
3. **Structured data:** Use templates strictly. Do not invent new structures for these files.

# VALIDATION
- Check if project directory is created.
- Verify `context.md` exists and contains Phase 1 answers.
- Verify `session-log.md` exists and has an initialization entry.

# STOP CONDITION
Stop when both files are successfully created and populated.

# FAILURE HANDLING
If directory creation fails (e.g., permissions), alert the user and ask for an alternative path.

# NEXT PHASE
Return control to SysForge for PHASE 3 (SCOUT).
