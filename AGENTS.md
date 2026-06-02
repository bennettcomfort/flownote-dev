# AGENTS.md

Canonical operating manual for AI coding agents working on **Flownote**.  
Tool-specific files (`.cursor/rules`, `CLAUDE.md`, etc.) should point here unless a tool requires a minimal duplicate.

---

## Project Mission

**What it is:** Flownote is a minimal macOS scratchpad for tasks and notes—TaskPaper-style outlining, natural-language due dates, keyboard-first capture, local-first persistence, and bi-directional Apple Reminders sync.

**MVP objective:** Ship a single-user macOS app where one text-first document round-trips reliably, core outline and scheduling work offline, Reminders sync handles create/update/complete/delete/due-date flows with local-change-wins conflicts, and a launch dashboard plus focus views (Today, Upcoming, Overdue, Someday) support daily dogfooding for 3+ weeks.

**Who it is for:** Individual power users who want a faster, lower-noise, macOS-native workflow instead of juggling separate notes and task apps.

**Not in MVP:** Team collaboration, cross-platform clients, cloud-mandatory accounts, AI copilots, Kanban/Gantt, plugin ecosystems, heavy attachments/OCR, broad third-party integrations beyond Apple Reminders. See `PRODUCT.md` and `SPEC.md` out-of-scope lists.

---

## Required Reading

Read these before non-trivial work. Skim updates when returning after a break.

| File | When to consult |
|------|-----------------|
| `PRODUCT.md` | **Why** we build something—users, must-haves, nice-to-haves, success criteria, explicit non-goals. |
| `SPEC.md` | **What** MVP must do—functional/non-functional requirements, acceptance criteria, edge cases. |
| `OPEN_QUESTIONS.md` | **Unresolved decisions**—do not guess; update when decided or mark tasks **Blocked** in `TODO.md`. |
| `ARCHITECTURE.md` | **How** to structure code—stack, components, data model, folders, interfaces, security, testing. |
| `PLAN.md` | **Build order**—phases, vertical slice, dependencies, validation plan, definition of done. |
| `TODO.md` | **What to do next**—the execution queue; one task at a time. |
| `ROADMAP.md` | **Phase themes**—optional context for prioritization; `TODO.md` wins for sequencing. |
| `IDEAS.md` | **Origin context**—brainstorm history; do not treat as spec if it conflicts with `SPEC.md`. |
| `RULES.md` | **Quick checklist**—condensed rules; full detail lives in this file. |

**Precedence:** `SPEC.md` + `ARCHITECTURE.md` + `OPEN_QUESTIONS.md` (decisions captured) override `IDEAS.md` and informal notes.

---

## Operating Rules

1. **One task at a time** from `TODO.md` → section **Now**, first unchecked item, unless the user directs otherwise.
2. **`TODO.md` is the source of execution tasks**—do not invent parallel workstreams.
3. **Keep changes small**—one focused session per task; touch few files.
4. **Do not modify unrelated files**—no drive-by refactors or formatting sweeps.
5. **Do not invent features** beyond `SPEC.md` MVP scope without explicit user approval.
6. **Do not silently resolve open questions**—document in `OPEN_QUESTIONS.md` or block the task.
7. **Do not work on `Blocked` or `Later` tasks** unless the user explicitly asks.
8. **Update `TODO.md`** when a task completes (move to **Done**, check box) or when bugs/blockers are found.
9. **Preserve planning intent**—local-first, conservative parsing, Reminders behind one sync service, Core Data for MVP storage per `ARCHITECTURE.md`.

---

## Planning Rules

Before editing application code:

1. **Summarize the task** (from `TODO.md` outcome + validation).
2. **List expected files** to create or modify (match `ARCHITECTURE.md` §6 layout).
3. **Note dependencies** (`Depends on` in `TODO.md`) and confirm they are done.
4. **Flag risks**—sync, parsing, permissions, SwiftUI editor limits, scope creep.
5. If ambiguous and **unsafe to assume**, mark **Blocked** in `TODO.md` and ask the user with **3 suggested answers + custom option**.
6. If ambiguous but **safe**, make the smallest assumption and **document it** in the completion report (and `OPEN_QUESTIONS.md` if durable).

---

## Coding Standards

Follow `ARCHITECTURE.md` unless the user overrides.

| Area | Standard |
|------|----------|
| **Platform** | macOS only for MVP; deployment target **macOS 14** unless validation documents otherwise. |
| **UI** | SwiftUI for app shell, navigation, dashboard, focus views. |
| **Persistence** | **Core Data** for documents, items, sync metadata; file-backed images in app container. |
| **Structure** | `App/`, `Features/`, `Domain/`, `Persistence/`, `Services/`, `Tests/`—keep feature UI separate from domain logic. |
| **Interfaces** | `OutlineService`, `Parser`, `DueDateParser`, `RemindersSyncService`, `AttachmentService`, `FocusQueryService`, `DashboardService`, `SyncQueue`—keep boundaries as in `ARCHITECTURE.md` §7. |
| **Style** | Prefer boring, maintainable Swift; clear names; focused files; no premature abstractions. |
| **Patterns** | Match existing code in the repo once present; extend rather than reimplement. |
| **Refactors** | Avoid broad refactors unless the task requires them. |
| **Sync** | Local store is source of truth; Reminders is external; default conflict policy **local-change-wins**. |
| **Parsing** | Conservative classification; soft confirmation for ambiguous due dates. |

---

## Dependency Policy

- **Do not add Swift packages** unless necessary and approved in the task or by the user.
- **Prefer** Apple frameworks (SwiftUI, Core Data, EventKit/Reminders APIs as applicable).
- **Before adding a dependency:** state why it is needed, alternatives considered, and maintenance risk.
- **Never** add abandoned, duplicate, or “convenience” libraries for one-liners.
- Lockfiles and Xcode project files: change only when the task requires.

---

## Security and Privacy Rules

From `ARCHITECTURE.md` §8 and `SPEC.md`:

- **Never commit secrets** (API keys, tokens, `.env`, credentials).
- **Never log** task/note body content or Reminders identifiers in production-style logs unless debugging and user-approved.
- **No weakening** of permission prompts, validation, or destructive-action safeguards without explicit instruction.
- **User data stays local** by default; no mandatory cloud account for MVP.
- **Attachments** only in the app container unless the user explicitly imports a file.
- **Reminders permission:** handle denied/revoked gracefully; clear UX, no crashes.
- **Destructive actions** (delete, bulk auto-archive `@DONE`): require confirmation and undo where specified.
- **Validate input** for file drops/pastes and date strings before persisting.

---

## Testing and Validation

A task is **not done** until its `TODO.md` **Validation** is satisfied or failures are reported.

### Required checks (when applicable)

| Check | Command / action |
|-------|------------------|
| **Build** | Xcode build for macOS target (⌘B) |
| **Unit tests** | XCTest unit target (⌘U, unit scheme) |
| **Integration tests** | XCTest integration target |
| **Manual** | Steps in `docs/manual-qa.md` for the affected area |

### By layer (`ARCHITECTURE.md` §10, `PLAN.md` Validation Plan)

- **Parser:** classification + round-trip tests.
- **Outline:** indent, outdent, reorder, fold tests.
- **Due dates:** common phrase fixtures; ambiguous → soft confirmation (manual when UI involved).
- **Persistence:** Core Data round-trip integration tests.
- **Sync:** queue + mocked Reminders; conflict path; manual bi-directional when wired.
- **Attachments:** store + preview integration test.
- **MVP sign-off:** all `SPEC.md` acceptance criteria.

If a check cannot run (no Mac, no Reminders permission, CI missing), **state why** in the completion report.

---

## Documentation Rules

- Update **`docs/`** when setup, parser rules, Reminders behavior, or QA steps change.
- Update **`TODO.md`** on task completion, new bugs, or blockers.
- Add discoveries to **`TODO.md` → Bugs**.
- Add unresolved product/tech choices to **`OPEN_QUESTIONS.md`** (do not silently decide).
- Durable decisions may later go in **`DECISIONS.md`** if that file is added; until then use `OPEN_QUESTIONS.md` **Decisions Captured** section.
- Do not edit **`PRODUCT.md` / `SPEC.md` / `ARCHITECTURE.md`** unless the user asks for a planning change.

---

## File Boundaries

### Avoid unless the task explicitly requires

| Path / area | Reason |
|-------------|--------|
| `.git/` | Git metadata |
| `logs/` | Local runtime logs (gitignored) |
| `.cursor/hooks/state/` | Ephemeral IDE state (gitignored) |
| `.DS_Store` | OS metadata |
| User home, paths outside repo | Not part of project |
| `DerivedData/`, `build/` | Generated |
| Planning docs | Unless directed—product/architecture changes are human-led |
| `IDEAS.md` | Historical; not execution spec |

### Prefer to touch for feature work

- `App/`, `Features/`, `Domain/`, `Persistence/`, `Services/`, `Tests/`, `Resources/`
- `docs/` for dev-setup, parser rules, manual QA, spikes
- `TODO.md` for progress

### Generated / careful

- `*.xcodeproj`, `project.pbxproj`—only for Xcode/project tasks
- Core Data model versioning—coordinate with schema tasks
- Entitlements & capabilities—Reminders-related tasks only

---

## Task Completion Criteria

- [ ] Task **outcome** from `TODO.md` is met.
- [ ] Task **validation** passed or failures documented.
- [ ] **No unrelated** file changes.
- [ ] **No secrets** added.
- [ ] **Docs** updated if behavior/setup/commands changed.
- [ ] **`TODO.md`** updated (checkbox, **Done**, **Bugs**, or **Blocked**).
- [ ] **Open questions** not silently resolved.
- [ ] MVP boundaries respected (`ARCHITECTURE.md` §13).

---

## Completion Report Format

End every coding task with:

```markdown
## Completion report

**Task:** <TODO.md task name>

**Summary:** <1–3 sentences>

**Files modified:** <list>

**Commands run:** <build/test commands and results>

**Test/check results:** <pass/fail/skipped + reason>

**Issues / skipped checks:** <if any>

**TODO updates:** <moved to Done, new Bugs, etc.>

**Recommended next task:** <first unchecked Now item or dependency-unblocked item>
```

---

## Tool-Specific Notes

| Tool | Guidance |
|------|----------|
| **Cursor / Claude Code / Codex / Warp** | Follow this `AGENTS.md`; use `RULES.md` as a short reminder. |
| **Commits** | Only when the user asks; concise messages focused on *why*. |
| **PRs** | Only when the user asks; include test plan from `SPEC.md` where relevant. |
| **Xcode** | Primary IDE for build/run/test; document new schemes or targets in `docs/dev-setup.md`. |

---

## Current Execution State

- **Implementation:** Not started (planning docs complete; see `TODO.md` **Done**).
- **First coding task:** Create Xcode macOS app project (SwiftUI shell, macOS 14).
- **Storage decision:** Core Data (not SwiftData) for MVP per `ARCHITECTURE.md`.
- **Blocking open questions:** None for MVP start per `OPEN_QUESTIONS.md`.

---

## Quick Reference: MVP Architecture Boundaries

**In scope:** One macOS app, one document, TaskPaper editor + outline ops, parser, due dates with soft confirmation, dashboard + focus views, Reminders sync + offline retry, inline images, keyboard commands, completed styling + bulk archive.

**Out of scope for agents unless asked:** Import/export, menu bar quick-add, recurring sync, saved filters UI, markdown notes, theming, AppKit bridge (until SwiftUI proven insufficient), cloud accounts, collaboration, AI features.

See `ARCHITECTURE.md` §13 and `SPEC.md` out-of-scope for full lists.
