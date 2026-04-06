---
name: tdd-pass
description: Write minimum code to pass the failing test. Use after tdd-test to make the current failing test pass.
---

# TDD Pass — Make the Test Pass

Change the code to make the failing test (and all previous tests) pass.

## Syntax

```
/tdd-pass [feedback: <trim feedback>]
```

**Parameters:**
- `feedback` (optional): Trim feedback from a previous attempt. When provided, rewrite the implementation addressing the specific excess identified.

## Workflow

### 1. Implement

- Look only at the failing test and write the minimum code to make it pass
- Do NOT add code that is not required by the failing test
- Do NOT add unnecessary if statements, error handling, or features
- Do NOT add comments or documentation (e.g., Javadoc, JSDoc)

### 2. Verify

- Run the entire test suite (not just the new test) to ensure all tests pass
- Run code style checks (e.g., lint, format) if available
- If tests or style checks fail, fix and retry (up to 3 attempts)
- If still failing after 3 attempts, STOP and request review

### 3. Report

- Report what was completed
- If new scenarios were discovered during implementation, report them so the caller can append them to the list
- Do NOT mark scenarios as done (`- [x]`) — the caller manages scenario progress

## Important

- Do NOT read requirements, architecture, design documents, or scenario lists — read only the failing test
- Do NOT modify existing tests
- Do NOT proceed to the next scenario automatically
