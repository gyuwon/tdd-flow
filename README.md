# TDD Flow

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that orchestrates the entire TDD workflow end-to-end: scenario planning, test-first development, implementation, and refactoring — all driven by an agent team.

## Skills

| Command | Description |
|---|---|
| `/tdd-flow:tdd-flow` | Conduct the full TDD cycle by delegating to specialized agent teammates |
| `/tdd-flow:tdd-list` | Write a list of test scenarios for TDD |
| `/tdd-flow:tdd-test` | Turn one scenario into a failing test |
| `/tdd-flow:tdd-audit` | Evaluate whether a test sufficiently verifies its scenario |
| `/tdd-flow:tdd-pass` | Write minimum code to pass the failing test |
| `/tdd-flow:tdd-trim` | Detect excess implementation beyond what tests require |
| `/tdd-flow:tdd-refactor` | Refactor implementation while keeping all tests green |

## Agents

Persistent teammates spawned by `/tdd-flow:tdd-flow` as an agent team. Each teammate retains full context across all messages within a TDD session.

| Agent | Model | Role |
|---|---|---|
| `tdd-test-writer` | Sonnet | Writes one failing test per scenario via `/tdd-test` |
| `tdd-critic` | Opus | Audits test quality via `/tdd-audit` and detects excess code via `/tdd-trim` |
| `tdd-implementer` | Sonnet | Writes minimum code to pass the failing test via `/tdd-pass` |
| `tdd-refactorer` | Sonnet | Improves code structure via `/tdd-refactor` |

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

The `/tdd-flow:tdd-flow` skill orchestrates the entire cycle — planning scenarios with user involvement, then autonomously looping through test → audit → implement → trim → refactor for each scenario:

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
├── test-writer (teammate)  → one failing test via /tdd-test
├── critic (teammate)       → test audit via /tdd-audit, excess check via /tdd-trim
├── implementer (teammate)  → minimum code via /tdd-pass
└── refactorer (teammate)   → design improvement via /tdd-refactor
```

## Installation

```bash
# Test locally without installing
claude --plugin-dir ./tdd-flow

# Or install via marketplace after publishing
```

## License

[MIT](LICENSE)
