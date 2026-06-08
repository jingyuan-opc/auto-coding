---
name: soc-build
description: "Use when an OpenSpec change has a tasks.md ready for implementation. Dispatches subagent per task with two-stage review, updates task status automatically."
---

# BSS Build — Subagent-Driven Implementation

Execute OpenSpec tasks by dispatching a fresh subagent per task, with two-stage review after each: spec compliance review first, then code quality review.

**Why subagents:** Each subagent gets isolated context with precisely crafted instructions. They never inherit session history — you construct exactly what they need. This preserves your own context for coordination.

**Core principle:** Fresh subagent per task + two-stage review (spec then quality) + acceptance criteria verification = high quality, fast iteration. A task is NOT done until it passes all acceptance criteria defined in tasks.md.

**Continuous execution:** Do not pause between tasks. Execute all tasks without stopping. The only reasons to stop are: BLOCKED status you cannot resolve, ambiguity that genuinely prevents progress, 3 consecutive task failures, or all tasks complete.

## When to Use

When an OpenSpec change exists at `openspec/changes/<name>/` with a `tasks.md` containing incomplete tasks.

## Trigger

```
/soc-build              # Auto-detects the active change
/soc-build <name>       # Specifies which change to build
```

## Process

1. **Resolve change** — If no name given, auto-detect active change in `openspec/changes/` (excluding `archive/`). If ambiguous, use AskUserQuestion to select.
2. **Parse tasks.md** — Read `openspec/changes/<name>/tasks.md`, extract incomplete tasks (`[ ]`), sort by ID
3. **Read context** — Load proposal.md and design.md for subagent context
4. **For each task:**
   a. Mark task `🔄 in_progress` in tasks.md
   b. Spawn Agent (general-purpose) with context from proposal.md, design.md, and current task description (use `./implementer-prompt.md` template)
   c. Run spec compliance review (use `./spec-reviewer-prompt.md` template)
   d. If spec review fails → implementer fixes, max 2 retries
   e. Run code quality review (use `./code-quality-reviewer-prompt.md` template)
   f. If quality review fails → implementer fixes, max 2 retries
   g. Verify acceptance criteria — confirm implementation satisfies all acceptance criteria defined in the task. If any criterion is not met → implementer fixes, max 2 retries
   h. All reviews pass AND acceptance criteria met → mark task `[x] ✅` in tasks.md
   i. If task fails after retries → mark `❌`
5. **Handle failures:**
   - Single task failure: mark ❌, continue to next
   - 3 consecutive failures: pause entire flow, notify user
6. **After all tasks** → output summary report, then **directly invoke `/soc-archive`** to archive the change

## Subagent Context

Each implementer subagent receives:
- Change name
- proposal.md content (why we're doing this)
- design.md content (how to build it)
- Current task description (what to implement)
- Project root directory path

## Status Markers in tasks.md

- `[ ]` — pending
- `[ ] 🔄 in_progress` — being worked on
- `[x] ✅` — done
- `[ ] ❌` — failed

## Model Selection

- **Simple tasks** (1-2 files, clear spec): fast model
- **Integration tasks** (multi-file, patterns): standard model
- **Review tasks** (architecture, judgment): most capable model

## Handling Implementer Status

- **DONE:** Proceed to spec compliance review
- **DONE_WITH_CONCERNS:** Read concerns, proceed if observations only
- **NEEDS_CONTEXT:** Provide missing context, re-dispatch
- **BLOCKED:** Assess blocker, escalate if needed

## Failure Handling

| Scenario | Action |
|----------|--------|
| Subagent fails | Retry once, then mark ❌ |
| Review not passed | Subagent fixes, max 2 retries |
| 3 consecutive failures | Pause entire flow, notify user |
| User interrupts | Progress saved in tasks.md, resumable |

## Report Format

After all tasks, output:
```
BSS Build Complete: <name>
Total: N tasks | ✅ Passed: X | ❌ Failed: Y
Tasks:
  ✅ 1.1 Create theme context
  ✅ 1.2 Create toggle component
  ❌ 2.1 Add CSS variables (failed after 3 attempts)
```

Then directly invoke `/soc-archive` to archive the change. Do NOT wait for user instruction.
