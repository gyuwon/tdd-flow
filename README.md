# TDD Flow

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that orchestrates the entire TDD workflow end-to-end: scenario planning, test-first development, implementation, and refactoring — all driven by an agent team.

## Skills

| Command | Description |
|---|---|
| `/tdd-flow:tdd-flow` | Conduct the full TDD cycle by delegating to specialized agent teammates |
| `/tdd-flow:tdd-list` | Write a list of test scenarios for TDD |

## Agents

Persistent teammates spawned by `/tdd-flow:tdd-flow` as an agent team. Each agent has its workflow built in — no skill invocation needed. Teammates retain full context across all messages within a TDD session.

| Agent | Model | Role |
|---|---|---|
| `tdd-test-writer` (×2) | Sonnet | Two writers compete in parallel worktrees; each writes one failing test and commits in the worktree |
| `tdd-critic` | Opus | Compares competing tests and selects the best one; provides concrete trim instructions for excess implementation code |
| `tdd-implementer` | Sonnet | Writes minimum code to pass the failing test; executes trim deletions when instructed |
| `tdd-refactorer` | Sonnet | Improves code structure while keeping tests green |

## Usage

### Prerequisites

Agent teams must be enabled. Add to your Claude Code settings:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### Automated full TDD cycle

The `/tdd-flow:tdd-flow` skill orchestrates the entire cycle — planning scenarios with user involvement, then autonomously looping through write (parallel) → select → apply → implement → trim → refactor for each scenario:

```
/tdd-flow:tdd-flow feature: https://github.com/owner/repo/issues/42
```

```
/tdd-flow:tdd-flow feature: https://myteam.atlassian.net/browse/PROJ-123
```

Override the model for all teammates:

```
/tdd-flow:tdd-flow feature: https://github.com/owner/repo/issues/42 model: opus
```

### Architecture

```
/tdd-flow (team lead)
├── /tdd-list (skill, invoked directly) → scenario list
├── test-writer-a (teammate, worktree) ─┐
├── test-writer-b (teammate, worktree) ─┤→ critic selects best test
├── critic (teammate)                    → compare and select, trim
├── implementer (teammate)               → minimum code to pass, execute trim
└── refactorer (teammate)                → improve code structure
```

### Cycle per scenario

```
1. Write   test-writer-a + test-writer-b in parallel worktrees
2. Select  critic compares, audits, picks the best (or rejects both)
3. Apply   team lead applies winning test to main tree, verifies failure
4. Pass    implementer writes minimum code
5. Trim    critic evaluates, implementer executes deletions
6. Refactor refactorer improves structure
   ↓
   Transition: temporary commit ([tdd-flow-wip]), mark scenario done
```

On completion, all temporary commits are reset (`git reset --soft`) to leave changes staged for the user.

## Installation

```bash
# Test locally without installing
claude --plugin-dir ./tdd-flow

# Or install via marketplace after publishing
```

## License

[MIT](LICENSE)
