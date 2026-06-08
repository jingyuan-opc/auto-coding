---
name: soc-archive
description: "Use when an OpenSpec change is complete and ready to be archived. Syncs specs and archives via OpenSpec CLI automatically. Triggered directly by /soc-build after completion."
---

# BSS Archive — Auto Archive Change

Archive a completed change. Fully automated — no user interaction required.

**Input**: Optionally specify a change name after `/soc-archive` (e.g., `/soc-archive add-auth`). If omitted, auto-detect from conversation context. If ambiguous, pick the most recently active change.

## Steps

1. **Resolve change name**

   If not provided, auto-detect from conversation context. If called after `/soc-build`, use the same change name. If truly ambiguous, use `openspec list --json` and pick the only active (non-archived) change. If multiple active changes exist, pick the one most recently referenced.

2. **Check artifact completion status**

   Run `openspec status --change "<name>" --json` to check artifact completion.

   Parse the JSON to understand:
   - `schemaName`: The workflow being used
   - `planningHome`, `changeRoot`, `artifactPaths`, and `actionContext`: path and scope context
   - `artifacts`: List of artifacts with their status (`done` or other)

   If status reports `actionContext.mode: "workspace-planning"`, report error and STOP.

   If any artifacts are not `done`: log a warning but proceed without stopping.

3. **Check task completion status**

   Read the tasks file (typically `tasks.md`) to check for incomplete tasks.

   Count tasks marked with `- [ ]` (incomplete) vs `- [x]` (complete).

   If incomplete tasks found: log a warning but proceed without stopping.

   If no tasks file exists: proceed normally.

4. **Sync delta specs automatically**

   Use `artifactPaths.specs.existingOutputPaths` from status JSON to check for delta specs.

   If delta specs exist, invoke `openspec-sync-specs` via Skill tool to sync them. Proceed to archive regardless of sync outcome.

   If no delta specs exist, skip sync.

5. **Perform the archive**

   Create an `archive` directory under `planningHome.changesDir` if it doesn't exist:
   ```bash
   mkdir -p "<planningHome.changesDir>/archive"
   ```

   Generate target name using current date: `YYYY-MM-DD-<change-name>`

   If target already exists: report error with options (rename existing / delete duplicate / retry later) and STOP.

   Otherwise:
   ```bash
   mv "<changeRoot>" "<planningHome.changesDir>/archive/YYYY-MM-DD-<name>"
   ```

6. **Display summary**

   Report final result:
   ```
   ## Archive Complete

   **Change:** <change-name>
   **Schema:** <schema-name>
   **Archived to:** <archive-path>
   **Specs:** ✓ Synced (or "No delta specs" or "Sync skipped")
   **Warnings:** <any warnings, or "None">
   ```

## Guardrails

- Auto-resolve change name from context, no user prompts
- Proceed on warnings (incomplete artifacts/tasks) without stopping
- Always sync delta specs if they exist, no confirmation needed
- Preserve .openspec.yaml when moving to archive
- Show clear final summary of what happened
