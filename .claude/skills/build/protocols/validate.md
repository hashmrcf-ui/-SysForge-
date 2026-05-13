# SysForge Protocol: Validate

# PURPOSE
To serve as the central Validation Engine for SysForge. It enforces security, architecture, and dependency checks before allowing progression to the next phase.

# INPUTS
- Current Phase ID.
- Generated artifacts/code from the current Phase.
- `architecture.md` (if applicable).

# REQUIRED OUTPUTS
- Validation Report (Pass/Fail).
- List of missing files or security vulnerabilities.

# RULES
1. **Zero Tolerance for Secrets:** Scan for hardcoded API keys, JWT secrets, or DB passwords. If found, FAIL immediately.
2. **Architecture Compliance:** Ensure generated code matches the Domain Model and API Contract in `architecture.md`.
3. **Dependency Check:** Ensure no hidden libraries or conflicting versions are introduced.
4. **Phase Locking:** Do not issue a "PASS" unless all checks clear.

# VALIDATION
Execute checks based on Phase:
- **Phase 4.5 (SysArch):** Check for RLS strategy, Tenant isolation plan, and defined roles.
- **Phase 7A (SysCore):** Check for `.env` usage, HttpOnly cookies, and correct middleware implementation.
- **Phase 7B (SysWeb):** Check for AI rate limiting, audit logs, and component purity (no direct DB calls).

# STOP CONDITION
Stop when the checklist is fully evaluated and a Pass/Fail status is generated.

# FAILURE HANDLING
If FAIL, output the exact issues and propose fixes. Wait for user to authorize fixes before re-running validation.

# NEXT PHASE
Return control to SysForge to prompt the user for approval to proceed.
