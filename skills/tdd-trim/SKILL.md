---
name: tdd-trim
description: Check whether implementation contains code beyond what is needed to pass the tests. Use after tdd-pass to detect over-engineering.
---

# TDD Trim — Detect Excess Implementation

Check whether the implementation contains code that is not required by the current tests.

## Syntax

```
/tdd-trim
```

## Workflow

### 1. Identify Scope

- Run `git diff` to find all implementation changes from the most recent `tdd-pass` step. If there are no unstaged changes, use `git diff HEAD~1` to check the last commit.
- Read the current test suite to understand what is being tested

### 2. Evaluate

Apply the following criteria:

**Is there code not exercised by any test?**
- Logic branches that no test reaches
- Methods or parameters that no test calls or verifies

**Is there premature generalization?**
- Abstractions introduced before a second use case exists
- Configurability or extensibility not demanded by any test

**Is there speculative error handling?**
- Validation or exception handling for cases no test covers
- Defensive checks against conditions no test produces

**Is there unnecessary structure?**
- Helper methods, constants, or classes that could be inlined without harming readability
- Patterns (strategy, factory, etc.) applied before complexity warrants them

### 3. Report

Output one of:

- **Pass** — The implementation contains only what the tests require
- **Needs trimming** — with specific locations and reasons

## Important

- Do NOT modify the test or implementation code
- Do NOT write new tests
