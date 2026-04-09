---
name: tdd-audit
description: Evaluate whether a test sufficiently verifies the scenario's intent. Use after tdd-test to check test quality before moving to tdd-pass.
---

# TDD Audit — Evaluate Test Effectiveness

Analyze whether the written test truly verifies the intent of its scenario.

## Syntax

```
/tdd-audit [scenario: <description>]
```

**Parameters:**
- `scenario` (optional): The scenario to audit. If not provided, find the most recent incomplete (`- [ ]`) scenario from the test list whose corresponding test method already exists.

## Workflow

### 1. Identify the Scenario and Test

- Locate the target scenario from the test list
- Find the test method corresponding to the scenario
- Read the test code

### 2. Evaluate

Apply the following criteria:

**Does the test verify the core behavior of the scenario?**
- The test must exercise the specific action described in the scenario
- The test must assert the expected outcome, not a side effect

**Is the test meaningful (not vacuously passing)?**
- The test must be able to fail when the behavior is broken
- A test that passes with an empty or default implementation is vacuous

**Are boundary conditions and exceptions covered?**
- If the scenario implies edge cases, the test should address them
- Missing error/exception handling should be flagged

**Are assertions specific enough?**
- Assertions should not be so loose that incorrect behavior passes (e.g., only checking non-null)
- Assertions should not be so coupled to implementation that any refactoring breaks them

**Does the test prefer classicist style over mockist style?**
- If the test can verify behavior using real collaborators, it should not use mocks
- Mocks are acceptable only when real collaborators are impractical (e.g., external services, non-deterministic dependencies)

**Does the test prefer stubs over spies?**
- If the test can verify behavior through return values or state, it should use stubs, not spies
- Spies are acceptable only when the scenario's intent is to verify that a specific interaction occurred

**Was exactly one test method added?**
- Check `git diff` for new test methods — only one should have been added
- If multiple test methods were added (even for different scenarios), report **Needs improvement** — the test-writer must remove all but the one for the current scenario

### 3. Report

Output one of:

- **Pass** — The test sufficiently verifies the scenario
- **Needs improvement** — with specific reasons and suggestions

## Important

- Do NOT modify the test or implementation code
- Do NOT write new tests
