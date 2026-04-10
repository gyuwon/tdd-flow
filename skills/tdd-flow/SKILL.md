---
name: tdd-flow
description: Conduct the full TDD cycle by delegating to specialized agents. Two test writers compete in parallel, a critic selects the best test, then implementation, trim, and refactor follow.
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

Create an agent team and spawn **5 teammates**, each based on an agent definition. Teammates are persistent — they retain full context across all messages within the session.

- **First message** to a teammate: include the full task context (scenario description, relevant file paths).
- **Subsequent messages**: include only the new instruction or feedback — the teammate already has context from prior interactions.

### Teammate reference

| Name | Agent type | Role |
|------|-----------|------|
| test-writer-a | `tdd-flow:tdd-test-writer` | Write one failing test in a worktree |
| test-writer-b | `tdd-flow:tdd-test-writer` | Write one failing test in a worktree |
| critic | `tdd-flow:tdd-critic` | Compare and select best test, check excess |
| implementer | `tdd-flow:tdd-implementer` | Write minimum code to pass the failing test |
| refactorer | `tdd-flow:tdd-refactorer` | Improve code structure |

Both test writers use the same agent type but are spawned as separate teammates with different names. They work independently in isolated worktrees.

### Skills invoked directly

| Skill | When | Why |
|-------|------|-----|
| `/tdd-list` | Planning | Requires user interaction (review, approval) and external persistence — cannot run inside a teammate |

## Architecture

```
/tdd-flow (you — team lead)
├── /tdd-list (skill, invoked directly) → scenario list
├── test-writer-a (teammate, worktree) ─┐
├── test-writer-b (teammate, worktree) ─┤→ critic selects best test
├── critic (teammate)                    → compare and select, trim
├── implementer (teammate)               → minimum code to pass
└── refactorer (teammate)                → improve code structure
```

## Workflow

### Planning (user-involved)

1. Invoke `/tdd-list` with the user's feature description.
2. `/tdd-list` will understand the feature, write scenarios, collect user approval, and persist the list.
3. Once the scenario list is confirmed, proceed to Setup.

### Setup

Create an agent team with `TeamCreate`, then spawn all five teammates in parallel using the `Agent` tool. Each `Agent` call must include:

- `description`: a short label (e.g., "TDD test writer A")
- `name`: the teammate name from the reference table (e.g., `test-writer-a`)
- `subagent_type`: the agent type (e.g., `tdd-flow:tdd-test-writer`)
- `team_name`: the team name from `TeamCreate`
- `prompt`: project context only (repo path, issue number, SUT location)
- `model` (if the user specified one): override model for all teammates

**Do NOT include the scenario list in spawn prompts.** Scenarios are provided one at a time during the Cycle phase.

After all five teammates are ready, proceed to the Cycle phase.

### Cycle (autonomous)

Process every incomplete scenario (`- [ ]`) in order, **without waiting for user input** between scenarios.

Use `SendMessage` to communicate with teammates. Teammates retain context — do NOT repeat information they already have.

For each scenario, execute **all 6 steps in order**. Do NOT skip any step.

#### Step 1: Write Competing Tests (parallel)

1. Create two git worktrees for this scenario (use unique branch names to avoid collisions, e.g., `tdd-wt-a-{scenario-index}` and `tdd-wt-b-{scenario-index}`).
2. Send the **current** scenario description to **both** test-writer-a and test-writer-b **simultaneously**. Each message must include:
   - The assigned worktree path
   - This exact instruction: **"After writing the test, you MUST run `git add -A && git commit -m 'test: <scenario>'` in the worktree before reporting back. Without a commit, your work will be lost."**
   - Never include other scenarios or the full scenario list.
3. Wait for **both** writers to complete before proceeding.

#### Step 2: Select the Best Test

- Message **critic**: provide the scenario description and both candidate tests (including their worktree paths). Ask it to compare and select the better test.
- The critic evaluates both tests for scenario fidelity, precision, readability, and robustness, then reports one of:
  - **Accept**: one test is selected as the winner, with the worktree path. Proceed to Step 3.
  - **Reject**: neither test is good enough. The critic provides specific feedback.
- If rejected:
  - Forward the feedback to **both** test-writer-a and test-writer-b (they still have their worktree context). They revise in their existing worktrees.
  - Message **critic** again with the revised tests.
  - If still rejected after 2 retries, escalate to user.

#### Step 3: Apply and Verify

- Apply the winning test from the selected worktree to the main working tree. Use `git checkout <branch> -- <path>` to pull changed files from the winning worktree's branch. Do NOT use `git diff` + `git apply` — patches break when the main tree has staged changes from prior scenarios.
- Remove both worktrees and their branches.
- Verify the test **fails** as expected. If it **passes unexpectedly**, escalate to user.

#### Step 4: Make the Test Pass

- Message **implementer** with the **test method name and file path** of the failing test that was applied in Step 3. Do NOT send scenario descriptions, future scenarios, or requirements — only the existing failing test location.
- The implementer retries internally up to 2 times.
- If it still fails after retries, escalate to user.

#### Step 5: Trim

- Message **critic**: "run trim check" (it already has the scenario context from the compare step).
- If the critic reports **Pass**, proceed to Step 6.
- If the critic reports **Trim** with deletion instructions:
  - Forward the deletion instructions to **implementer**. The implementer executes the deletions, verifies all tests pass, and reports what was removed.
  - If tests fail after deletion, escalate to user.

#### Step 6: Refactor

- Message **refactorer**: the current scenario + "review and refactor if needed".
- The refactorer decides whether improvements are warranted and applies them, or reports that no changes are needed.
- Wait for the refactorer to complete before proceeding to Transition.

#### Transition

- Mark the scenario as done (`- [x]`) in the feature source where the list was persisted (e.g., update the GitHub issue body).
- Stage and create a temporary commit for this scenario (`git add -A && git commit -m "[tdd-flow-wip] <scenario>"`). This ensures the next scenario's worktrees include this scenario's work. These temporary commits are reset during Completion.
- Proceed immediately to the next incomplete scenario.

### Completion

When all scenarios are marked `- [x]`:

1. Run the full test suite one final time.
2. Reset the temporary commits: find the oldest `[tdd-flow-wip]` commit and run `git reset --soft <commit-before-it>`. This leaves all scenario changes staged but uncommitted, giving the user full control over the final commit.
3. Present a summary to the user:
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
- **Only invoke `/tdd-list` directly.** Teammates have their workflows built in — they do not invoke skills.
- **Communicate via SendMessage.** After setup, always use `SendMessage` to interact with teammates. Do NOT spawn new subagents with the `Agent` tool during the cycle.
- **Teammates retain context.** On subsequent messages, include only new instructions or feedback — do NOT repeat the scenario description or prior results.
- **Use `tdd-flow:` agent types for teammates.** The ONLY valid agent types are: `tdd-flow:tdd-test-writer`, `tdd-flow:tdd-critic`, `tdd-flow:tdd-implementer`, `tdd-flow:tdd-refactorer`.
- **Send to both test writers simultaneously.** Always message test-writer-a and test-writer-b in parallel for each scenario.
- **Never wait for user input** during the autonomous cycle unless escalating.
- **Execute all 6 steps for every scenario.** Never skip select, apply, implement, trim, or refactor.
- **Max 2 retries** for all steps.
- **Progress reporting:** After every 3 scenarios or when escalating, briefly summarize progress (e.g., "Completed 5/12 scenarios, no issues so far").
