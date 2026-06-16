# Implementer Retry Subagent Prompt Template

Use this template when re-dispatching an implementer to fix a failed quality gate or unmet acceptance criteria. **Do NOT use this for the initial implementation** — that's `implementer-prompt.md`.

**Purpose:** Force the retry to address the specific failure rather than re-trying the same approach and failing the same way.

```
Agent tool (general-purpose):
  description: "Fix Task N (retry R of 2): [task name]"
  prompt: |
    You are retrying Task N: [task name] — attempt R of max 2 retries.

    ## Why You're Here (READ THIS FIRST)

    The previous attempt failed a quality gate or its self-verified acceptance criteria didn't hold
    up under the orchestrator's check. You are NOT starting from scratch — your job is to diagnose
    and fix a SPECIFIC failure. Re-implementing from scratch wastes the previous attempt's
    learnings and is likely to fail the same way.

    ## Task Identifier

    - **Task ID:** [task-id, e.g. "1.2"]
    - **Change name:** <change-name>
    - **Project root:** [project-root]
    - **Spec dir:** openspec/changes/<change-name>

    ## Gate Profile

    For this task, run ONLY these quality gates when verifying your fix:
    [comma-separated gate list, e.g. "test, lint, type-check" — or "all detected gates" if no `[gates: ...]` annotation was specified on the task line in tasks.md]

    Valid gate names: `test`, `lint`, `build`, `type-check`

    Detect available gates using the same logic as `soc-archive` step 2a (auto-detect from `package.json` / `tsconfig.json` / `pyproject.toml`, or read from `.soc-quality-gate.json`).

    **Test scope:** When running the `test` gate, run ONLY the scoped set —
    tests in files you create/modify plus tests covering the modules you touched. Do NOT
    re-run the full project suite here; the E2E phase and `soc-archive` handle the full run.

    ## What to Read

    Same reading strategy as the initial implementer prompt:

    1. **proposal.md** — read fully if you need the broader "why" (usually only on first read).
    2. **tasks.md** — find your task by ID and re-read the acceptance criteria carefully.
    3. **design.md** — read only the section relevant to your task.
    4. **Source code** — focus on the previously changed files and the failure report's referenced files:line.

    In your report, reference files by `path:line` or section header. Do NOT paste large blocks.

    ## Failure Context

    **Failed at:** <which step>
    One of: `acceptance criteria check` | `gate: <name>` | `E2E regression`
    **Attempt:** R of max 2
    **Files previously changed:** [list from previous attempt]

    ### What was reported done in the previous attempt
    [previous implementer report summary — files touched, what was implemented, gate results claimed, residual concerns flagged]

    ### What specifically failed
    [full failure report from the orchestrator / E2E / gate output — paste it verbatim, do not summarize]

    For gate failures, also include:
    - **Failed gate:** <gate name>
    - **Command:** <full command>
    - **Exit code:** <n>
    - **Output (last 30 lines):**
      ```
      <output>
      ```

    ## Phase 1: Diagnose (DO NOT SKIP)

    Before changing any code:
    1. Read the actual code/files in the previous attempt (do NOT trust the report — the report is why we're here)
    2. Read the failure report in full
    3. Identify the ROOT CAUSE — choose ONE category and state it explicitly:
       - **Code bug:** typo, wrong API call, missing import, off-by-one, etc. → fix it
       - **Design misunderstanding:** re-read design.md carefully, then fix the implementation
       - **Spec ambiguity:** after careful reading you genuinely cannot determine what was wanted → escalate as `NEEDS_CONTEXT` (NOT `BLOCKED`)
       - **Test itself is wrong:** test contradicts the task's acceptance criteria → fix the test, but ONLY if the test is clearly incorrect (not just inconvenient)
    4. State the root cause in your report before making changes

    **Do NOT just retry the same approach.** That is what failed.

    ## Phase 2: Fix

    Apply the MINIMAL change that addresses the root cause:
    - Don't refactor unrelated code
    - Don't add features not requested
    - Don't change the architecture
    - Match existing patterns in the codebase
    - If the previous attempt had a working approach and only one part failed, keep the working part — don't rewrite everything

    ## Phase 3: Verify

    Run the gate profile specified above. Confirm:
    - The specific failure from the failure report is fixed (re-read the report, check each point)
    - No NEW failures introduced
    - All acceptance criteria from the task description are still met

    If a gate fails again, do NOT report DONE — fix and re-run.

    ## Phase 4: Report

    When done, report:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - **Root cause:** What was actually wrong (be specific, e.g. "test relied on `await` in a non-async function — flaked under load; real fix is to make the helper async and assert on settled state")
    - **Fix applied:** Files changed and what each change does
    - **Quality gates:** Exit code + last 20 lines of output for each gate run
    - **Acceptance criteria:** Each AC from the task description and its current status with evidence
    - **Verification:** How you confirmed the specific failure is fixed (e.g., "reran the failing scoped test in isolation 3× — all green; re-ran the rest of the scoped set — no new failures")
    - **Residual concerns:** Anything still suspicious (will be passed to E2E if you report DONE_WITH_CONCERNS)

    Work from: [project-root]
```
