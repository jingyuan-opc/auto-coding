# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes.**

```
Agent tool (general-purpose):
  description: "Review code quality for Task N"
  prompt: |
    You are reviewing code quality for a recently implemented task.

    ## Task Summary

    [From implementer's report]

    ## Design Reference

    Task N from openspec/changes/<name>/design.md:
    [Relevant design section]

    ## Your Job

    Review the code changes for quality. Focus on:

    **Security:**
    - Any injection vulnerabilities (SQL, command, XSS)?
    - Proper input validation at system boundaries?
    - Sensitive data handled correctly?

    **Error handling:**
    - Errors caught and handled appropriately?
    - Edge cases covered?
    - No silent failures?

    **Code clarity:**
    - Clear naming (names match what things do)?
    - No unnecessary complexity?
    - Follows existing codebase patterns?

    **File organization:**
    - Does each file have one clear responsibility?
    - Are units decomposed for independent understanding and testing?
    - Did this change create new files that are already large, or significantly grow existing files?

    **Testing:**
    - Do tests verify actual behavior?
    - Are edge cases tested?
    - Test coverage adequate for the change?

    Report:
    - **Strengths:** What's done well
    - **Issues:** Critical / Important / Minor (with file:line references)
    - **Assessment:** ✅ Pass or ❌ Needs fixes (with specific instructions)
```
