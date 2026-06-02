# Task: [paste TODO title from TODO.md → Now]

**Branch:** [e.g. cursor/xcode-scaffold]  
**Date:** [optional]

---

## Rules (from RULES.md)

Paste the **Execution**, **Stack**, and **Safety** bullets from [RULES.md](../RULES.md), or the full file if context allows.

---

## Task spec (from TODO.md)

```markdown
- [ ] [Task name]
  - Outcome:
  - Depends on:
  - Validation:
  - Notes:
```

---

## Architecture excerpt

Paste only the sections needed for this task (from [ARCHITECTURE.md](../ARCHITECTURE.md)):

- §4 Data model — Core Data tasks
- §6 Folder structure — Xcode / scaffold tasks
- §7 Interfaces — sync, parser, services
- §8 Security — Reminders, attachments, destructive actions

---

## Spec excerpt (optional)

Paste relevant bullets from [SPEC.md](../SPEC.md) if the task touches acceptance criteria or edge cases.

---

## Relevant files

### Paths only (preferred when repo is large)

```text
[list paths the agent should read or modify]
```

### File contents (paste when small or critical)

```swift
// path/to/file.swift
[paste content]
```

---

## Instructions

- Implement **only** this task.
- Follow [AGENTS.md](../AGENTS.md) and [local-agent-workflow.md](local-agent-workflow.md).
- Do **not** work on TODO **Later** or **Blocked** items.
- Do **not** add dependencies without justification.
- When finished, provide the AGENTS.md completion report and TODO.md checkbox text.

---

## Open questions

If this task might touch an unresolved item, paste from [OPEN_QUESTIONS.md](../OPEN_QUESTIONS.md) or note “none blocking.”
