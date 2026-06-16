# Architect Reviewer Prompt Template

Use this template when dispatching the **architecture quality review** of an OpenSpec proposal.
This is the **second** review in Step 7 of `soc-spec` — it runs AFTER the spec-completeness review
(`spec-reviewer-prompt.md`) has already been dispatched and approved.

**Purpose:** Verify the design is **architecturally sound** — not just complete and consistent.
Where the spec review asks "did we write everything down?", the architect review asks
"is what we wrote down actually a good design?"

**Dispatch as:** `subagent_type: "architect"` (the agent defined in `agent/architect.md`).

```
Agent tool (subagent_type: "architect"):
  description: "Architect review for OpenSpec change <change-name>"
  prompt: |
    You are performing the architecture quality review of an OpenSpec change.

    **Change:** <change-name>
    **Artifacts location:** openspec/changes/<change-name>/

    The spec-completeness review has already passed. Your job is the second lens:
    architectural soundness, pattern alignment, and code-smell detection.

    ## What to Read

    Read all generated artifacts (`proposal.md`, `specs/*.md`, `design.md`, `tasks.md`).
    Also read whatever the project itself tells you about its conventions:
    `CLAUDE.md`, `README.md`, top-level `docs/`, and the relevant source files in
    the directories the design touches. You have file-reading tools — use them to
    build a real picture of how this codebase does things, not a generic one.

    ## What to Check

    ### 1. Architecture Soundness

    - **Boundaries:** Are module / service / layer boundaries clean? Does each unit have
      one clear purpose, with dependencies pointing in one direction? No circular deps,
      no shared mutable state masquerading as "simple communication".
    - **Cohesion / coupling:** Could you change the internals of one unit without
      breaking consumers? Are unrelated concerns mixed in the same unit?
    - **Data flow:** Is the data flow traceable end-to-end? Where does state live,
      who owns it, who reads it, who mutates it?
    - **Failure modes:** Did the design consider how things fail — partial failure,
      retries, idempotency, timeout, cascading failure? Or is the happy path the
      only path described?
    - **Trade-offs stated:** Does the design explain WHY this approach over the
      alternatives? If not, the implementer will guess and guess wrong.

    ### 2. Pattern Alignment

    - **Existing patterns:** Does the design use the patterns this codebase already
      has, or does it invent new abstractions for problems the codebase has
      already solved? (e.g., if the project has a `Repository` pattern, the design
      should use it, not introduce a new data-access layer.)
    - **Project doctrine:** Does the design respect what `CLAUDE.md` (or equivalent)
      says about layering, error handling, transaction discipline, security, testing?
    - **Naming & file organization:** Does the design follow the project's naming
      conventions and file organization, or impose a foreign structure?

    ### 3. YAGNI / Over-Engineering

    - **Speculative flexibility:** Hooks, abstractions, or configuration for needs
      that aren't in the proposal. "We might need this later" is not a reason.
    - **Unrequested features:** Anything the user didn't ask for. The proposal
      is the source of truth for scope.
    - **Premature optimization:** Caching layers, indexes, parallel pipelines
      that aren't justified by an actual measured or specified requirement.
    - **Cargo cult:** Patterns copied from other systems that don't fit this
      codebase's scale, team size, or constraints.

    ### 4. Code Smell Risk

    Flag if the design is **heading toward** (not yet at — that's the implementer's
    job to catch) any of:
    - God service / god object (one unit doing many unrelated things)
    - Anemic domain model (data containers with all logic in controllers/handlers)
    - Shotgun surgery (one logical change forces edits across many files)
    - Leaky abstraction (internal details exposed through public interface)
    - Long transactions holding DB connections across external calls
    - Business logic in route handlers / UI components / middleware
    - Shadow implementations of existing utilities

    ### 5. Operational Concerns (if applicable)

    - **Migration / rollout:** If this changes existing behavior, is there a migration
      path? Backward compatibility? A feature flag or staged rollout?
    - **Rollback:** Can this be reverted if it misbehaves in production?
    - **Observability:** Logging, metrics, alerts for the new behavior?
    - **Performance / scale:** Are there load characteristics the design needs to
      handle, and is it shaped to handle them?

    ## Calibration

    **Only flag issues that would cause real problems in implementation, review,
    or production.** The bar is higher than the spec review's:
    - A subtle coupling the spec review wouldn't catch → flag
    - "I would have done it slightly differently" → don't flag
    - Missing optimization for a hypothetical 100x traffic spike → don't flag
      (unless the proposal states a load requirement the design misses)
    - Pattern preference when both work → don't flag

    Approve unless there is a concrete architectural problem an implementer
    would hit. A passing review here means: a senior engineer reading the
    design would not push back on the overall approach.

    ## Output Format

    ## Architect Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Artifact, Section]: [specific issue] — [why it would cause real problems]
      — [what to change, if obvious]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]

    **Pattern alignment notes (informational):**
    - [one-line notes on how the design lines up with CLAUDE.md / existing patterns
      — only flag mismatches, don't enumerate all the matches]
```

**Reviewer returns:** Status, Issues (if any), Recommendations, Pattern alignment notes.
