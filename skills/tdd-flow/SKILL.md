---
name: tdd-flow
description: Conduct the full TDD cycle by delegating to specialized agents. Drives planning with user involvement, then autonomously loops through test, audit, implementation, trim, and refactor for each scenario.
---

# TDD Flow — Full TDD Cycle

Conduct the full TDD cycle by delegating each step to a persistent teammate in an agent team.

## Syntax

```
/tdd-flow [feature: <description or url>] [model: <sonnet|opus|haiku>]
```

**Parameters:**
- `feature` (optional): Description of the feature or URL of the issue. If not provided, use conversation context or ask the user.
- `model` (optional): Model to use for all teammates (`sonnet`, `opus`, or `haiku`). When specified, set this model for every teammate at spawn time. If omitted, each teammate uses its default model from its agent definition.

## Prerequisites

Agent teams must be enabled. If not already configured, add to settings:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Role

You are the **team lead** of a TDD session. You do NOT write tests or code yourself. You create an agent team, spawn teammates, delegate each step via `SendMessage`, and evaluate outcomes before moving on.

## How to Delegate

Create an agent team and spawn **4 teammates**, each based on an agent definition. Teammates are persistent — they retain full context across all messages within the session.

- **First message** to a teammate: include the full task context (scenario description, relevant file paths).
- **Subsequent messages**: include only the new instruction or feedback — the teammate already has context from prior interactions.

### Teammate reference

| Name | Agent type | Role |
|------|-----------|------|
| test-writer | `tdd-flow:tdd-test-writer` | Write one failing test via `/tdd-test` |
| critic | `tdd-flow:tdd-critic` | Audit tests via `/tdd-audit`, check excess via `/tdd-trim` |
| implementer | `tdd-flow:tdd-implementer` | Write minimum code via `/tdd-pass` |
| refactorer | `tdd-flow:tdd-refactorer` | Improve design via `/tdd-refactor` |

### Skills invoked directly

| Skill | When | Why |
|-------|------|-----|
| `/tdd-list` | Planning | Requires user interaction (review, approval) and external persistence — cannot run inside a teammate |

## Architecture

```
/tdd-flow (you — team lead)
├── /tdd-list (skill, invoked directly) → scenario list
├── test-writer (teammate)  → one failing test via /tdd-test
├── critic (teammate)       → test audit via /tdd-audit, excess check via /tdd-trim
├── implementer (teammate)  → minimum code via /tdd-pass
└── refactorer (teammate)   → design improvement via /tdd-refactor
```

## Workflow

### Planning (user-involved)

1. Invoke `/tdd-list` with the user's feature description.
2. `/tdd-list` will understand the feature, write scenarios, collect user approval, and persist the list.
3. Once the scenario list is persisted, proceed to Setup.

### Setup

Create an agent team and spawn all four teammates. Reference each agent type by name. If the user specified a `model` parameter, apply it to every teammate.

After all four teammates are ready, proceed to the Cycle phase.

### Cycle (autonomous)

Process every incomplete scenario (`- [ ]`) in order, **without waiting for user input** between scenarios.

Use `SendMessage` to communicate with teammates. Teammates retain context — do NOT repeat information they already have.

For each scenario, execute **all 5 steps**. Do NOT skip any step.

#### Step 1: Write a Failing Test

- Message **test-writer**: the current scenario description.
- If the test **passes unexpectedly**, escalate to user.

#### Step 2: Audit the Test

- Message **critic**: the current scenario + "run audit".
- If the audit reports **Pass**, proceed to Step 3.
- If the audit reports **Needs improvement**:
  - Message **test-writer**: the audit feedback only (it already has the scenario).
  - Message **critic**: "re-audit" (it already has the scenario).
  - If still failing, retry once more (retry 2).
  - If still failing after 2 retries, escalate to user.

#### Step 3: Make the Test Pass

- Message **implementer**: the failing test information.
- The implementer retries internally up to 2 times.
- If it still fails after retries, escalate to user.

#### Step 4: Check for Excess Code

- Message **critic**: "run trim check" (it already has the scenario context from the audit).
- If the report says **Pass**, proceed to Step 5.
- If the report says **Needs trimming**:
  - Message **implementer**: the trim findings (it already has context of the implementation).
  - Message **critic**: "re-check trim".
  - If still needs trimming, retry once more (retry 2).
  - If still failing after 2 retries, escalate to user.

#### Step 5: Refactor

- Message **refactorer**: the current scenario + "review and refactor if needed".
- The refactorer decides whether improvements are warranted and applies them, or reports that no changes are needed.

#### Transition

- Mark the scenario as done (`- [x]`) in the feature source where the list was persisted (e.g., update the GitHub issue body).
- Stage all changes with `git add`. This keeps `git diff` scoped to the current scenario for subsequent trim checks.
- Proceed immediately to the next incomplete scenario.

### Completion

When all scenarios are marked `- [x]`:

1. Run the full test suite one final time.
2. Present a summary to the user:
   - Total scenarios completed
   - Any issues encountered and how they were resolved
   - Any deferred items that were escalated
   - Suggested follow-up refactorings (if any)
3. Present the **scenario list** with completion status (e.g., `- [x] scenario A`, `- [ ] scenario B`).
4. Present a **resource usage report**:
   - **Cycle duration**: wall-clock time from the start of the first scenario to the completion of the last scenario (excluding Planning and Setup).
   - **Token usage**: total tokens consumed across all teammates during the cycle. Collect from each teammate's usage stats.
   - **Scenarios**: total completed, total retries across all steps.

5. Clean up the team.

## Escalation Procedure

When a step fails beyond retry limits:

1. STOP the autonomous cycle.
2. Report to the user:
   - Which scenario and step failed
   - What was attempted (including retry history)
   - The specific error or feedback
3. Wait for the user's guidance.
4. After the user resolves the issue, resume the cycle from the current step.

## Rules

- **Never write code or tests yourself.** Delegate all work to teammates.
- **Only invoke `/tdd-list` directly.** All other skills (`/tdd-test`, `/tdd-pass`, `/tdd-audit`, `/tdd-trim`, `/tdd-refactor`) are for the teammates to invoke internally.
- **Communicate via SendMessage.** After setup, always use `SendMessage` to interact with teammates. Do NOT spawn new subagents with the `Agent` tool during the cycle.
- **Teammates retain context.** On subsequent messages, include only new instructions or feedback — do NOT repeat the scenario description or prior results.
- **Use `tdd-flow:` agent types for teammates.** The ONLY valid agent types are: `tdd-flow:tdd-test-writer`, `tdd-flow:tdd-critic`, `tdd-flow:tdd-implementer`, `tdd-flow:tdd-refactorer`.
- **Never wait for user input** during the autonomous cycle unless escalating.
- **Execute all 5 steps for every scenario.** Never skip audit, trim, or refactor.
- **Max 2 retries** for all steps.
- **Progress reporting:** After every 3 scenarios or when escalating, briefly summarize progress (e.g., "Completed 5/12 scenarios, no issues so far").
