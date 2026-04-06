---
name: tdd-test
description: Turn one test scenario into a failing test. Use after tdd-list to write the next failing test in the TDD cycle.
---

# TDD Test — Write a Failing Test

Turn exactly one scenario from the list into a concrete, runnable, failing test.

## Syntax

```
/tdd-test [scenario: <description>] [feedback: <audit feedback>]
```

**Parameters:**
- `scenario` (optional): The scenario to implement. If not provided, pick the next incomplete scenario from the list.
- `feedback` (optional): Audit feedback from a previous attempt. When provided, rewrite the test addressing the specific issues raised.

## Workflow

### 1. Select Scenario

Identify the next incomplete (`- [ ]`) scenario from the test list.

### 2. Define Signatures (if needed)

If the minimum method signatures required for this scenario do not exist yet:

- Add only the methods necessary for this specific scenario
- Place new signatures in the type referenced by the test; if the type does not exist, create it
- Follow the existing code convention for ordering (e.g., alphabetical, by access modifier); when unclear, append at the end of the type
- Use empty or default/zero-value implementations
- Do NOT implement any logic
- Do NOT add methods for other scenarios
- Do NOT include comments
- Verify compilation succeeds

**Interactive mode:** Request user review and STOP. Do NOT write the test in this step.

**Orchestrated mode:** Report the signatures added, then proceed to Step 3 immediately.

### 3. Write the Test

- Convert the scenario into one executable test method
- Use snake_case for the test method name matching the scenario description
- Write only one test — never multiple tests at once
- Use existing test fixtures when possible; create new ones only when necessary

### 4. Verify Failure

Run the test and confirm it fails.

- If the test PASSES unexpectedly, STOP and request user review immediately
- If the test FAILS as expected, request user review

## Important

- Do NOT write implementation code to make the test pass
- Do NOT modify existing tests
