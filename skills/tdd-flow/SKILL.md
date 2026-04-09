---
name: tdd-flow
description: Conduct the full TDD cycle by delegating to specialized agents. Drives planning with user involvement, then autonomously loops through test, audit, implementation, trim, and refactor for each scenario.
---

# TDD Flow — Full TDD Cycle

Conduct the full TDD cycle by delegating each step to a specialized agent.

## Syntax

```
/tdd-flow [feature: <description or url>] [model: <sonnet|opus|haiku>]
```

**Parameters:**
- `feature` (optional): Description of the feature or URL of the issue. If not provided, use conversation context or ask the user.
- `model` (optional): Model to use for all sub-agents (`sonnet`, `opus`, or `haiku`). When specified, pass this as the `model` parameter of every `Agent` tool call during Setup — this overrides each agent definition's default model. If omitted, each agent uses its own default model from its frontmatter.

## Role

You are the conductor of a TDD session. You do NOT write tests or code yourself. You delegate every step to the appropriate agent and evaluate the outcome before moving on.

## How to Delegate

### Spawn once, reuse via SendMessage

Each sub-agent is spawned **once** with the `Agent` tool and then **reused** for all subsequent tasks via `SendMessage`. This preserves the agent's accumulated context (project structure, file paths, test framework, prior results) across the entire TDD session.

- **First contact**: Use the `Agent` tool with `subagent_type` to spawn the agent. Assign a descriptive `description` so you can identify it later.
- **Subsequent contacts**: Use `SendMessage` with the agent's name or ID as the `to` field. Include only the new task — the agent already has prior context.

### Agent reference

| Agent | subagent_type | When spawned | What to include in messages |
|-------|--------------|-------------|----------------------------|
| test-writer | `tdd-flow:tdd-test-writer` | Setup | Scenario, audit feedback (if retry) |
| critic | `tdd-flow:tdd-critic` | Setup | Scenario + "run audit" or "run trim check" |
| implementer | `tdd-flow:tdd-implementer` | Setup | Failing test location, trim feedback (if retry) |
| refactorer | `tdd-flow:tdd-refactorer` | Setup | What to improve |

### Skills invoked directly

| Skill | When | Why |
|-------|------|-----|
| `/tdd-list` | Planning | Requires user interaction (review, approval) and external persistence — cannot run inside a sub-agent |

## Agents

```
/tdd-flow (you — running in main conversation)
├── /tdd-list (skill, invoked directly) → scenario list
├── tdd-flow:tdd-test-writer    → one failing test via /tdd-test
├── tdd-flow:tdd-critic         → test audit via /tdd-audit, excess check via /tdd-trim
├── tdd-flow:tdd-implementer    → minimum code via /tdd-pass
└── tdd-flow:tdd-refactorer     → design improvement via /tdd-refactor
```

## Workflow

### Planning (user-involved)

1. Invoke `/tdd-list` with the user's feature description.
2. `/tdd-list` will understand the feature, write scenarios, collect user approval, and persist the list.
3. Once the scenario list is persisted, proceed to agent initialization.

### Setup

Spawn the four cycle agents **in parallel** using the `Agent` tool. Each agent receives an initial prompt that includes the scenario list (from `/tdd-list` output) and its role. If the user specified a `model` parameter, pass it as the `model` field in every `Agent` tool call.

```
Spawn all four in a single message (parallel Agent tool calls):

1. tdd-flow:tdd-test-writer
   "You are the test writer for this TDD session.
    Scenarios:
    <paste the scenario list here>
    Acknowledge and stand by."

2. tdd-flow:tdd-critic
   "You are the critic for this TDD session.
    Scenarios:
    <paste the scenario list here>
    Acknowledge and stand by."

3. tdd-flow:tdd-implementer
   "You are the implementer for this TDD session.
    Acknowledge and stand by."

4. tdd-flow:tdd-refactorer
   "You are the refactorer for this TDD session.
    Acknowledge and stand by."
```

Do NOT write the scenario list to a file. Pass it inline in the agent prompts.

After all four agents acknowledge, proceed to the Cycle phase.

### Cycle (autonomous)

Process every incomplete scenario (`- [ ]`) in order, **without waiting for user input** between scenarios.

Use `SendMessage` to communicate with the pre-spawned agents. Do NOT spawn new agents.

For each scenario:

#### Step 1: Write a Failing Test

- Send to **test-writer**: the current scenario description.
- If the test **passes unexpectedly**, escalate to user.

#### Step 2: Audit the Test

- Send to **critic**: the current scenario + "run audit".
- If the audit reports **Pass**, proceed to Step 3.
- If the audit reports **Needs improvement**:
  - Send to **test-writer**: the audit feedback (retry 1).
  - Send to **critic**: "re-audit" (retry 1).
  - If still failing, retry once more (retry 2).
  - If still failing after 2 retries, escalate to user.

#### Step 3: Make the Test Pass

- Send to **implementer**: the failing test information.
- The implementer retries internally up to 2 times.
- If it still fails after retries, escalate to user.

#### Step 4: Check for Excess Code

- Send to **critic**: "run trim check".
- If the report says **Pass**, proceed to Step 5.
- If the report says **Needs trimming**:
  - Send to **implementer**: the trim findings to rewrite the code (retry 1).
  - Send to **critic**: "re-check trim" (retry 1).
  - If still needs trimming, retry once more (retry 2).
  - If still failing after 2 retries, escalate to user.

#### Step 5: Refactor

- Send to **refactorer**: "review and refactor if needed".
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
3. Present an **agent event log** — a table of every agent interaction during the session:

| Scenario | Step | Agent | Spawn/Reuse | Result |
|----------|------|-------|-------------|--------|
| 1 | 1 — Write test | test-writer | Spawn | Failing test written |
| 1 | 2 — Audit | critic | Spawn | Needs improvement — assertion too broad |
| 1 | 2 — Retry | test-writer | Reuse | Fixed assertion |
| 1 | 2 — Re-audit | critic | Reuse | Pass |
| ... | ... | ... | ... | ... |

   Record each row as the cycle progresses — do NOT reconstruct from memory at the end. After the table, include a one-line stat: total spawns, total reuses, total retries.

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

- **Never write code or tests yourself.** Delegate all work to sub-agents.
- **Only invoke `/tdd-list` directly.** All other skills (`/tdd-test`, `/tdd-pass`, `/tdd-audit`, `/tdd-trim`, `/tdd-refactor`) are for the sub-agents to invoke internally.
- **Spawn each agent exactly once, then reuse via SendMessage.** Use the `Agent` tool only during Setup to create agents. During the Cycle, always use `SendMessage` to communicate with existing agents.
- **Only spawn tdd-flow: prefixed sub-agents.** The ONLY valid `subagent_type` values are: `tdd-flow:tdd-test-writer`, `tdd-flow:tdd-critic`, `tdd-flow:tdd-implementer`, `tdd-flow:tdd-refactorer`. Do NOT use `Explore`, `general-purpose`, `Plan`, or any other agent type.
- **Never wait for user input** during the autonomous cycle unless escalating.
- **Max 2 retries** for all steps.
- **Progress reporting:** After every 3 scenarios or when escalating, briefly summarize progress (e.g., "Completed 5/12 scenarios, no issues so far").
