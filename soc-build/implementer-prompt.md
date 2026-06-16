# Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent.

**No independent reviewer:** This subagent is the only implementer/reviewer. It must self-audit before reporting DONE. Treat the Phase 0 and Phase Z checklists as load-bearing — skimping on them is what fails downstream.

```
Agent tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    ## Task Identifier

    - **Task ID:** [task-id, e.g. "1.2"]
    - **Change name:** <change-name>
    - **Project root:** [project-root]
    - **Spec dir:** openspec/changes/<change-name>

    ## Gate Profile

    Run ONLY these quality gates when verifying your work:
    [comma-separated gate list, e.g. "test, lint, type-check" — or "all detected gates" if no `[gates: ...]` annotation was specified on the task line in tasks.md]

    Valid gate names: `test`, `lint`, `build`, `type-check`

    Detect available gates using the same logic as `soc-archive` step 2a (auto-detect from `package.json` / `tsconfig.json` / `pyproject.toml`, or read from `.soc-quality-gate.json`).

    **Test scope:** When running the `test` gate, run ONLY the scoped set — tests in files you create/modify plus tests covering the modules you touched. Do NOT run the full project suite; that's the E2E phase's job.

    ## What to Read (in this order)

    1. **proposal.md** — read FULLY. It's the "why" behind this change; usually <5KB. Internalize the goal and motivation.
    2. **tasks.md** — find your task by ID (Task N). Quote each acceptance criterion verbatim and number them (AC-1, AC-2, ...) in your Phase 0 plan. This list is your final checklist — every item must be checkable in code or test output.
    3. **design.md** — read ONLY the section relevant to your task. If the design uses task-numbered sections (e.g. `## Task 1.2`), read that. Otherwise, `grep` for keywords from your task title and AC and read just the matching slice. Do NOT dump the whole file.
    4. **Source code** — explore only what your Phase 0 plan needs. Use `Grep` / `Glob` to find candidate files; `Read` for specifics.

    In your report, reference files by `path:line` or section header. Do NOT paste large blocks from the spec or source.

    ## Phase 0 — Pre-Implementation Plan (REQUIRED, do not skip)

    Before writing any code, work through this checklist and write the plan into your report's
    "Pre-implementation plan" section. Skipping this phase is the #1 cause of rework.

    1. **Re-read the task.** Quote each acceptance criterion verbatim and number them (AC-1, AC-2, …).
       This list is your final checklist at the end — every item must be checkable in code or test output.
    2. **Re-read design.md.** Identify the specific constraints, interfaces, and layering rules that
       apply to THIS task (don't dump all of design.md — just the relevant slice). Note any
       file paths / class names / function signatures the design dictates.
    3. **List files to create/modify.** For each file, one sentence on what it does and why it
       needs to exist. This is your blast-radius check — if a file would do two unrelated things,
       split it.
    4. **Identify security/error paths.** If this task touches user input, external APIs, file I/O,
       auth, or persistence: enumerate the specific failure modes and how you'll handle them.
       If it doesn't touch any of these, say so explicitly.
    5. **Identify test strategy.** What behavior do the tests need to assert? Where do tests live?
       What's the minimum number of test cases to cover the acceptance criteria + edge cases?
       **Identify your scoped test set** — the unit tests you will actually run before reporting
       DONE. The scope is:
       - Tests in test files you newly create
       - Tests in test files co-located with or importing the source files you modify
         (e.g. for Python: tests in the same `tests/<module>/`; for JS/TS: files matching
         the same `<name>.test.ts` / `<name>.spec.ts` convention as your changed source)
       - Any existing tests that directly exercise functions/classes you changed

       Do NOT plan to run the full project test suite — that's the E2E phase's job. If your
       scoped set is unclear, list candidate test files in your plan and pick by name pattern.
    6. **State the plan in 1-2 sentences.** "I will <verb> by <approach>." If the plan reveals
       something the design didn't cover, escalate `NEEDS_CONTEXT` — don't silently improvise.

    ## Implementation

    Implement per your Phase 0 plan. Follow the project's existing patterns and layering — do
    not invent new abstractions. Run the configured quality gates as you go; don't write the
    whole feature then debug five failing tests at the end.

    ## Phase Z — Post-Implementation Self-Audit (REQUIRED before reporting DONE)

    Before reporting DONE, walk through this checklist. Every item must be true, or you must
    fix it. Paste the checklist into your report under "Self-audit" with ✅ / ❌ + evidence for
    each item.

    **Acceptance criteria (mandatory, numbered):**
    - AC-1: <quote criterion> — ✅ / ❌ — Evidence: <test name + file:line, or file:line showing the code>
    - AC-2: ...
    - Every AC from the task description must appear here. If you can't verify one, you are not done.

    **Security (skip section entirely if task has no security surface):**
    - No hardcoded secrets, API keys, or credentials
    - User input is validated at system boundaries
    - No injection vectors (SQL, command, XSS, path traversal)
    - Sensitive data not logged or leaked in error messages

    **Error handling:**
    - Errors are caught, re-thrown, or logged — never silently swallowed
    - Edge cases identified in Phase 0 are handled
    - Failure paths return useful messages to the caller / user

    **Code clarity & organization:**
    - Each new/modified function does one thing
    - No function > 80 lines (if longer, justify in report)
    - No new file > 400 lines (if longer, justify in report)
    - Naming matches what things do
    - Follows existing codebase patterns; no invented abstractions

    **Tests:**
    - Tests assert behavior, not implementation details
    - Edge cases identified in Phase 0 are tested
    - For UI interactions: Playwright e2e script covers the key user flow
    - All configured quality gates pass
    - **Scoped test set only** — you ran the unit tests covering the modules you touched
      (plus any new tests you wrote), NOT the full project suite. Full suite is the E2E phase's job.
    - Lint / type-check / build still run project-wide (they're cheap and catch cross-cutting issues)

    **Anti-patterns check (from project CLAUDE.md):**
    - No shadow implementations of existing utilities
    - No business logic in route handlers / UI components / middleware
    - No bypassing the project's existing Repository / service abstractions
    - No long DB transactions holding connections across external API calls or heavy computation

    If any item is ❌, fix it and re-run the gates before reporting DONE. If a check is
    genuinely not applicable to this task, mark it `N/A` with a one-line reason — do not omit it.

    ## Report Format

    When done, report:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - **Pre-implementation plan:** The Phase 0 output (numbered AC list, design constraints, file plan, security/error paths, test strategy, one-line plan)
    - **Plan summary:** Files touched and what each does (brief)
    - **What you implemented** (or what you attempted, if blocked)
    - **Quality gates run:** For each gate (test/lint/type-check/build) — command, exit code, last 20 lines of output, duration. For `test`: list which test files/dirs you ran (the scoped set, not full suite)
    - **Coverage:** Skip at task level — full-suite coverage is checked at E2E / archive time
    - **Self-audit:** The Phase Z checklist with ✅/❌/N/A + evidence for each item
    - **Acceptance criteria verification:** Each AC with status and evidence (also covered in self-audit, but call out explicitly here)
    - **Files changed**
    - **Residual concerns:** Anything you're unsure about — flag for the E2E phase to verify
    - **Concerns or issues** (if any)

    Use DONE_WITH_CONCERNS if you completed the work but have doubts the E2E phase should look at.
    Use BLOCKED if you cannot complete the task.
    Use NEEDS_CONTEXT if you need information that wasn't provided.
    Never silently produce work you're unsure about.

    Work from: [project-root]
```
