---
name: tdd-list
description: Write a list of test scenarios for TDD. Use when starting TDD for a new feature or adding scenarios to an existing list.
---

# TDD List — Write Test Scenarios

Prepare a list of test scenarios to cover a feature.

## Syntax

```
/tdd-list [feature: <description or url>]
```

**Parameters:**
- `feature` (optional): Description of the feature or URL of the issue (GitHub, Jira, Notion, etc.). If not provided, use conversation context or ask the user.

## Workflow

### 1. Understand the Feature

- Read the feature description, issue, or relevant code
- Explore the existing codebase for relevant context
- Identify the system under test (sut)
- Identify the behaviors to verify
- Resolve ambiguities by asking the user
- If the feature description already contains a test scenario list, ask the user whether to **skip** (use the existing list as-is) or **augment** (review and add missing scenarios)

### 2. Write Scenarios

Write test scenarios following these rules:

- Write one scenario per line as a checklist item (`- [ ]`)
- Write in English, present tense
- Start with lowercase (usable as test method name)
- Do not use special characters that prevent conversion to an identifier by replacing spaces with underscores
- Refer to the system under test as 'sut'
- Write as concisely as possible while preserving meaning
- Omit articles (a, an, the), filler words, and obvious context
- Prefer short verb phrases (e.g., "sut rejects empty name" over "sut should reject when the name is empty")
- Order from most important to least important
- Start with the simplest, most fundamental behavior

### 3. Present for Review

Show the scenario list to the user and ask for feedback. Do NOT write the list to any external system until the user approves.

### 4. Persist the List

After user approval, append the scenario list to the body of the feature source (e.g., GitHub issue, Jira work item, Notion page) using available tools. If no source was provided, ask the user where to persist the list.

## Important

- Do NOT write tests or code. Only produce the scenario list.
- Do NOT proceed to implementation. Stop after persisting the list.
