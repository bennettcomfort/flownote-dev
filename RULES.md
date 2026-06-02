# RULES.md

Condensed rules for Flownote. **Full authority:** [`AGENTS.md`](AGENTS.md).

---

## Mission (one line)

macOS local-first TaskPaper-style scratchpad with Reminders sync—MVP for solo power-user daily capture, schedule, and sync.

---

## Read first

| File | Use |
|------|-----|
| `TODO.md` | What to do **now** |
| `SPEC.md` | What “done” means |
| `ARCHITECTURE.md` | How to structure code |
| `OPEN_QUESTIONS.md` | What not to guess |
| `PLAN.md` | Build order |
| `AGENTS.md` | Full agent manual |

---

## Execution

1. One task at a time → first unchecked item under **`TODO.md` → Now**.
2. Small diffs; few files; no unrelated edits.
3. No **Later** / **Blocked** tasks unless user asks.
4. No new features outside `SPEC.md` MVP.
5. Mark `TODO.md` done when finished; log bugs in **Bugs**.

---

## Stack (MVP)

- **macOS 14+**, SwiftUI shell
- **Core Data** + app-container image files
- **No** cloud auth, **no** casual SPM deps
- Reminders only via **`RemindersSyncService`**
- **Local-change-wins** sync default

---

## Code layout

```
App/  Features/  Domain/  Persistence/  Services/  Tests/
```

Keep parser, outline, persistence, sync, and UI separated per `ARCHITECTURE.md` §6–7.

---

## Safety

- Never commit secrets or log sensitive content.
- Attachments stay in app container.
- Confirm destructive actions (delete, bulk `@DONE` archive).
- Graceful Reminders permission denial.

---

## Before you finish a task

- [ ] `TODO.md` outcome + validation met
- [ ] Xcode build (+ tests if applicable)
- [ ] Manual steps from task validation when UI/sync involved
- [ ] `TODO.md` updated
- [ ] Completion report per `AGENTS.md`

---

## Do not touch (unless task says so)

- `logs/`, `.cursor/hooks/state/`, `.DS_Store`
- Planning specs (`PRODUCT.md`, `SPEC.md`, `ARCHITECTURE.md`) without user request
- Post-MVP: import/export, menu bar, recurring sync, theming, AppKit bridge

---

## Open questions

If unsure: check `OPEN_QUESTIONS.md` → if blocking, move task to **Blocked** and ask user (3 options + custom). Do not silently decide.

---

## After planning stages

Next workflow step: **`ai-workflow-stage7-code`** — implement `TODO.md` tasks using this repo’s `AGENTS.md`.
