# SysForge Failure Recovery Protocol

# PURPOSE
To gracefully handle generation failures, validation errors, or architectural conflicts without restarting the entire pipeline. 

# TRIGGER
Automatically invoked by the Runtime Engine when a phase transitions to the `FAILED` state.

# RECOVERY WORKFLOW

## Step 1: Isolate the Failure
Determine exactly which node in the Validation Graph failed.
*Example: "Phase 7A Validation Failed: RLS policy is missing from the users table."*

## Step 2: Preserve Valid State
Do not delete the entire phase output. If the Auth endpoints generated correctly but the RLS policy failed, keep the Auth code.

## Step 3: Targeted Rollback
Rollback only the specific files or configurations that caused the failure. Remove invalid code blocks.

## Step 4: Regenerate & Patch
Instruct the generation engine to re-attempt ONLY the failed component, providing the failure reason as explicit context.
*Prompt Injection:* "Previous generation failed because [Reason]. Fix this specific issue without modifying [Preserved Components]."

## Step 5: Return to Active
Transition the State Machine from `FAILED` back to `ACTIVE` and re-run the Validation phase.

# FATAL FAILURES
If a failure loops 3 times (Failure → Active → Failure), it is deemed a **Fatal Failure**.
- Halt automation.
- Ask the user to intervene manually.
- Do NOT attempt to guess complex architectural resolutions.
