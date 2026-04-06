# TDD Flow

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that orchestrates the entire TDD workflow end-to-end: scenario planning, test-first development, implementation, and refactoring — all driven by specialized agents.

## Skills

| Command | Description |
|---|---|
| `/tdd-flow:tdd-flow` | Conduct the full TDD cycle by delegating to specialized agents |
| `/tdd-flow:tdd-list` | Write a list of test scenarios for a feature |
| `/tdd-flow:tdd-test` | Turn one scenario into a failing test |
| `/tdd-flow:tdd-audit` | Evaluate whether a test sufficiently verifies its scenario |
| `/tdd-flow:tdd-pass` | Write minimum code to pass the failing test |
| `/tdd-flow:tdd-trim` | Detect excess implementation beyond what tests require |
| `/tdd-flow:tdd-refactor` | Refactor implementation while keeping all tests green |

## Agents

Sub-agents spawned by `/tdd-flow:tdd-flow` to handle each TDD step:

| Agent | Model | Role |
|---|---|---|
| `tdd-test-writer` | Sonnet | Writes one failing test per scenario via `/tdd-test` |
| `tdd-critic` | Opus | Audits test quality via `/tdd-audit` and detects excess code via `/tdd-trim` |
| `tdd-implementer` | Sonnet | Writes minimum code to pass the failing test via `/tdd-pass` |
| `tdd-refactorer` | Sonnet | Improves code structure via `/tdd-refactor` |

## Usage

### Automated full TDD cycle

The `/tdd-flow:tdd-flow` skill orchestrates the entire cycle — planning scenarios with user involvement, then autonomously looping through test → audit → implement → trim → refactor for each scenario:

```
/tdd-flow:tdd-flow feature: https://github.com/owner/repo/issues/42
```

```
/tdd-flow:tdd-flow feature: https://myteam.atlassian.net/browse/PROJ-123
```
## Installation

```bash
# Test locally without installing
claude --plugin-dir ./tdd-flow

# Or install via marketplace after publishing
```

## License

[MIT](LICENSE)
