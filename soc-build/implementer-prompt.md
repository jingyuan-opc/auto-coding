# Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent.

```
Agent tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    ## Task Description

    [FULL TEXT of task from tasks.md — includes acceptance criteria]

    ## Context

    **Change:** <change-name>
    **Project root:** <project-root>

    **Why (from proposal.md):**
    [proposal.md content]

    **Architecture (from design.md):**
    [design.md content]

    ## Phase 1: Understand & Plan

    Before writing any code, build your implementation plan:

    1. **Understand scope** — What exactly does this task require? What are the acceptance criteria?
    2. **Map the design** — From design.md, extract:
       - Which files to create or modify (follow the file structure defined in design)
       - Key interfaces, types, and contracts you must conform to
       - How this task fits into the broader architecture (dependencies, consumers)
       - Constraints and conventions from the design
    3. **Plan your approach:**
       - File list: which files you'll touch and what each change does
       - Implementation order: what to build first, what depends on what
       - Test strategy (see Testing section below)
    4. **Identify risks** — Anything unclear, ambiguous, or potentially outside the design scope

    If anything is unclear about requirements, approach, or constraints — **ask now**.
    Do not proceed with assumptions. It is always OK to pause and clarify.

    ## Phase 2: Implement

    Execute your plan:

    1. Follow the file structure and architectural decisions from design.md — it is a constraint, not a suggestion
    2. Follow existing codebase patterns for anything the design doesn't specify
    3. If you discover the design has a gap or something doesn't fit, **escalate** — don't silently work around it
    4. Keep files focused — each file should have one clear responsibility
    5. If a file you're creating is growing beyond the design's intent, stop and report as DONE_WITH_CONCERNS

    ### Testing Strategy

    **Business logic (complex conditions, state transitions, calculations, data transformations):**
    - Write tests FIRST, then implement to make them pass
    - Cover edge cases and error paths

    **Non-logic code (configuration, data mapping, boilerplate, simple delegation):**
    - Implement directly, no mandatory tests

    **Integration points:**
    - Write integration tests to verify the happy path works end-to-end

    **Do NOT write meaningless tests** — tests for getters/setters, constructors with no logic, or pure data objects add no value.

    ### Code Organization

    - Follow the file structure defined in the design
    - Each file should have one clear responsibility with a well-defined interface
    - In existing codebases, follow established patterns. Improve code you're touching the way a good developer would, but don't restructure things outside your task scope

    ## Phase 3: Verify & Self-Review

    After implementation, verify your work:

    **Acceptance Criteria:**
    - Check EVERY acceptance criterion from the task description
    - Each criterion must be demonstrably met — run tests, check output, verify behavior
    - If any criterion is not met, fix it now

    **Self-Review:**
    - Completeness: Did I implement everything required? Any edge cases missed?
    - Quality: Clear names, clean structure, no unnecessary complexity?
    - Discipline: No overbuilding (YAGNI)? Only what was requested?
    - Design compliance: Does implementation follow the architecture from design.md?
    - Testing: Do tests verify real behavior? Are logic paths covered?

    If you find issues during self-review, fix them before reporting.

    ## When You're in Over Your Head

    It is always OK to stop and say "this is too hard for me." Bad work is worse than no work.

    **STOP and escalate when:**
    - The task requires architectural decisions the design didn't address
    - You need to understand code beyond what was provided and can't find clarity
    - You feel uncertain about whether your approach is correct
    - The task involves restructuring existing code in ways the design didn't anticipate
    - You've been reading file after file trying to understand the system without progress

    **How to escalate:** Report back with status BLOCKED or NEEDS_CONTEXT. Describe specifically what you're stuck on, what you've tried, and what kind of help you need.

    ## Report Format

    When done, report:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - **Plan summary:** Files touched and what each does (brief)
    - **What you implemented** (or what you attempted, if blocked)
    - **Test results:** What you tested and results
    - **Acceptance criteria:** Each criterion and its verification status
    - **Files changed**
    - **Self-review findings** (if any)
    - **Concerns or issues** (if any)

    Use DONE_WITH_CONCERNS if you completed the work but have doubts.
    Use BLOCKED if you cannot complete the task.
    Use NEEDS_CONTEXT if you need information that wasn't provided.
    Never silently produce work you're unsure about.

    Work from: [project-root]
```
