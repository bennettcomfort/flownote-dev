# PLAN.md

## Implementation Summary

### What will be built
A macOS-native, local-first TaskPaper-style scratchpad that:
- Parses one text-first document into tasks, projects, and notes with outline operations (indent, outdent, reorder, fold).
- Persists structure and metadata offline via Core Data, with file-backed image attachments in the app container.
- Parses natural-language due dates with soft confirmation for ambiguous phrases.
- Syncs supported item lifecycle changes with Apple Reminders through an isolated sync service (local-change-wins by default).
- Surfaces a launch dashboard (due-today + quick-link chips) and focus views (Today, Upcoming, Overdue, Someday).
- Renders completed items in the defined subdued style and supports bulk auto-archive of `@DONE` items.

### MVP goal
Deliver a single-user daily workflow where capture, edit, schedule, complete, and Reminders sync are reliable enough for 3+ weeks of dogfooding, with keyboard-viable core actions and no mandatory cloud account.

### Guiding implementation principles
- **Smallest vertical slice first:** prove editor round-trip and local persistence before Reminders sync and attachments.
- **Local-first:** the editor store is source of truth; Reminders is an external sync target.
- **Separate concerns:** parser, outline operations, persistence, due-date parsing, and Reminders sync stay behind clear interfaces (`ARCHITECTURE.md` §7).
- **Conservative parsing:** prefer predictable classification and soft date confirmation over aggressive inference.
- **Solo-builder scope:** one document, one window, shallow folder structure; defer import/export, theming, menu bar, and AppKit unless SwiftUI blocks correctness.
- **Do not resolve open questions silently:** defer recurring sync, saved filters, markdown notes, and AppKit bridging unless explicitly pulled into scope.

---

## Build Strategy

### Recommended order of implementation
1. **Foundation** — Xcode project, Core Data schema, folder layout, baseline tests, manual checklists.
2. **Vertical slice** — single document load/save, parser round-trip, minimal editor shell (type + persist + reload).
3. **Core MVP** — outline ops, due dates, focus views, dashboard, completed styling, Reminders sync, images, sync status, keyboard commands.
4. **Hardening** — performance, empty states, error UX, security safeguards, documentation cleanup.

### Why this order is safest
- Parser and persistence bugs are cheaper to fix before sync multiplies state.
- Reminders permission and API behavior should be validated early (spike in Phase 0) but wired after local truth exists.
- Dashboard and focus views depend on stable item/due-date models.
- Attachments and bulk archive are valuable but not required to prove the core capture → edit → complete loop.

### What should be built first
- Core Data entities and repositories for `Document`, `Item`, `DueDate`, `ReminderLink`, `Attachment`, `SyncState`.
- Parser/classifier with round-trip tests.
- Minimal editor surface that reads/writes the document model.

### What should be delayed
- Import/export (TaskPaper first per `OPEN_QUESTIONS.md` — post-MVP per `ARCHITECTURE.md`).
- Preferences beyond launch-critical defaults.
- Menu bar quick-add, recurring sync, saved filters, markdown notes, theming.
- AppKit editor bridge until SwiftUI limits are proven.
- Optional per-action conflict override (`ARCHITECTURE.md` §12).

---

## Phase 0: Project Foundation

| Task | Details |
|------|---------|
| Repository / project setup | Create Xcode macOS app target (SwiftUI shell), deployment target **macOS 14**, bundle ID and signing for local dev. Align repo layout with `ARCHITECTURE.md` §6 (`App/`, `Features/`, `Domain/`, `Persistence/`, `Services/`, `Tests/`). |
| Tooling | Swift Package Manager only when a dependency is justified; XCTest targets for unit + integration. Enable build/type checks in CI when ready (optional in Phase 0). |
| Environment | No cloud secrets for MVP. Document Reminders capability and sandbox/entitlements needs in README or `docs/`. |
| Core Data schema | Model `Document`, `Item`, `DueDate`, `ReminderLink`, `Attachment`, `SyncState` with fields/relationships from `ARCHITECTURE.md` §4. Start with **one primary attachment per item** in schema if it simplifies MVP; allow multiple in UI when storage path is stable. |
| Reminders spike | Small prototype or integration test hook: request permission, create/read/update/complete/delete a test reminder, document failure modes. |
| Parser rules draft | Document line rules for task / project / note, `@DONE`, tags, due-date metadata (per `ROADMAP.md` Phase 0). |
| Design tokens (minimum) | Soft note style, completed-item style (opacity, gray, red strikethrough, hide due date, show `@DONE`), dashboard chip styles. |
| Baseline tests | Empty XCTest targets; placeholder tests that build. |
| Documentation | Keep `PRODUCT.md`, `SPEC.md`, `ARCHITECTURE.md` as source of truth; add short `docs/dev-setup.md` for build/run and Reminders testing. |
| Manual validation checklist | Seed checklist from `ARCHITECTURE.md` §10 (editor, dates, sync, offline, permissions, images). |

**Phase 0 done when:** App builds and launches; Core Data stack migrates; Reminders spike results documented; parser rules and schema reviewed against `SPEC.md`.

---

## Phase 1: Smallest Vertical Slice

**Goal:** One end-to-end path: open app → edit TaskPaper-like text → autosave → quit → relaunch → identical structure and line types.

### UI
- Single main window with a text-first editing surface (SwiftUI `TextEditor` or equivalent).
- No dashboard yet; optional placeholder navigation.

### Backend / domain
- `Parser`: raw text ↔ structured `Item` graph (task, project, note).
- `OutlineService` (minimal): insert line, update text (indent/outdent/reorder deferred to Phase 2).
- `Local document store`: one `Document` record, autosave on change debounce.

### Storage / data
- Core Data repositories for document + items.
- Round-trip: save loaded items back to canonical text without losing classification.

### Validation
- Unit tests: parser classification, round-trip preservation (`ARCHITECTURE.md` §10).
- Manual: create mixed task/project/note lines, reload app, confirm fidelity.

**Phase 1 done when:** Acceptance criterion “preserves content faithfully across save and reload” holds for line types defined in `SPEC.md`; no Reminders dependency yet.

---

## Phase 2: Core MVP Completion

### Editor and outline
- Full outline operations: indent, outdent, reorder, fold/unfold.
- Combined task + note list with softer note styling.
- Completed-item rendering per `SPEC.md` / `OPEN_QUESTIONS.md` (muted, `@DONE`, hide due, strikethrough).
- Command/menu: **Auto-archive all `@DONE`** with confirmation + undo window (`ARCHITECTURE.md` §8).

### Due dates
- `DueDateParser` for common phrases (`tomorrow 3pm`, `next Monday`, `in two weeks`).
- Soft confirmation UI for ambiguous phrases (inline suggestion, accept/edit).

### Focus views and dashboard
- `FocusQueryService`: Today, Upcoming, Overdue, Someday.
- `DashboardService`: due-today list with times, quick-link chips (Inbox, Today, Scheduled, Completed, Projects, custom views).
- **Default launch:** dashboard (`SPEC.md`, `OPEN_QUESTIONS.md`).

### Apple Reminders sync
- `RemindersSyncService` + `SyncQueue`: create, update, complete, delete, due-date changes; pull remote changes.
- `ReminderLink` mapping; per-item `SyncState`.
- **Conflict policy:** local-change-wins default; visible syncing / success / error / conflict states.
- Graceful degradation when permission denied or revoked.

### Attachments
- `AttachmentService`: paste/drop images to app container, thumbnails, link to item (multiple per item per `SPEC.md`; implement storage accordingly).

### Offline and recovery
- Offline editing always writes locally first.
- Background retry on app lifecycle / sync-now; no large custom job system.

### Keyboard-first
- `AppCommands`: new item/task, toggle complete, indent/outdent, move up/down, fold, set due date, sync now, focus view shortcuts, archive command.
- Command palette entry points in app shell.

### Integration tests
- Persistence round-trip, sync queue with mocked Reminders, conflict path, attachment storage.

**Phase 2 done when:** `SPEC.md` acceptance criteria and `ARCHITECTURE.md` §10 “must pass” list are satisfied for MVP scope (excluding post-MVP polish).

---

## Phase 3: Hardening and Polish

| Area | Work |
|------|------|
| Security / privacy | Confirm attachments stay in app container; destructive actions (delete, bulk archive) have confirmation/undo; Reminders permission copy is clear. |
| UX | Empty states (no items, no permission, no focus results); sync error recovery messaging; dashboard hierarchy tuning. |
| Edge cases | Large document performance (filtering/editing); broken image paste; bulk archive with zero `@DONE` items; offline backlog on reopen. |
| Performance | Profile editor + focus queries; reduce main-thread work on large outlines. |
| Documentation | Update manual QA checklist; document known Reminders limitations. |
| Tests | Expand integration coverage; keyboard navigation smoke tests where automatable. |

**Phase 3 done when:** Dogfooding-ready: stable sessions, acceptable performance on realistic note size, clear failure UX.

---

## Later / Post-MVP

- Import/export: **TaskPaper first**, then markdown/plain text (`OPEN_QUESTIONS.md`).
- Recurring task sync where Reminders allows.
- Menu bar quick-add and global hotkey.
- Saved filters / context tags beyond core focus views.
- Markdown-style rich notes, theming controls.
- Weekly review assistant, usage insights.
- AppKit editor bridge if SwiftUI insufficient.
- Optional per-action sync conflict override.
- SwiftData migration evaluation (only if Core Data cost proves high — not a Phase 2 goal).

---

## Dependencies and Sequencing

```text
Phase 0 (schema, spike, rules)
    ↓
Phase 1 (parser + persistence + minimal editor)
    ↓
Phase 2a (outline ops + due dates + completed UI)
    ↓
Phase 2b (focus views + dashboard)     Phase 2c (Reminders sync) — can overlap after 2a model stable
    ↓
Phase 2d (attachments + keyboard commands + offline retry)
    ↓
Phase 3 (hardening)
```

| Dependency | Blocks |
|------------|--------|
| Core Data schema | All persistence, sync metadata, focus queries |
| Parser round-trip | Editor, sync mapping, export later |
| Item + DueDate model | Focus views, dashboard due-today, date parser UI |
| Local persistence | Reminders sync (queue needs local truth) |
| Reminders spike | Full sync implementation confidence |
| Outline operations | Keyboard move/indent commands |
| Sync service | Sync status UI, conflict flags |

**Independent after Phase 1:** Due-date parser unit tests; focus query logic tests against fixture data; dashboard chip models.

**Risky sequencing:** Building dashboard before item/due-date queries exist; wiring Reminders before parser/reminder link mapping is stable; AppKit bridge before SwiftUI editor is evaluated.

---

## Validation Plan

| Check | When |
|-------|------|
| Build | Every commit; Xcode build for macOS 14+ |
| Type check | Swift compiler strictness as configured in project |
| Lint | SwiftLint or equivalent if adopted in Phase 0 |
| Unit tests | Parser, round-trip, due-date phrases, focus filters, outline ops |
| Integration tests | Core Data round-trip, sync queue + mock Reminders, conflicts, attachments |
| Manual | `ARCHITECTURE.md` §10 checklist + permission denied path |
| Acceptance | Map to `SPEC.md` § Acceptance Criteria |

**MVP acceptance highlights (from `SPEC.md`):**
- Create/update tasks, projects, notes in one editor.
- Save/reload fidelity.
- Bi-directional Reminders for supported operations; local-change-wins.
- NL due dates + soft confirmation for ambiguous input.
- Focus views and dashboard accurate on launch.
- Offline safe + resume sync.
- Keyboard-viable main workflow.
- Completed styling + bulk archive command.

---

## Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| **Product:** Sync duplicates / mismatched completion | Single sync service, explicit `ReminderLink`, queue retries, visible status; dogfood with small doc first |
| **Product:** Parsing mistrust | Conservative rules, tests for mixed lines, manual fixtures |
| **Product:** Wrong due dates | Soft confirmation for ambiguity; conservative parser; hide due on complete |
| **Product:** Scope creep | Defer ROADMAP Phase 3+ items; gate features against `SPEC.md` out-of-scope |
| **Technical:** SwiftUI editor limits | Time-box SwiftUI attempt in Phase 1–2; AppKit only if blocking (open question) |
| **Technical:** Large doc lag | Debounced saves, indexed queries for focus views, avoid full reparse on every keystroke where possible |
| **Security/Privacy:** Reminders permission denied | Clear onboarding, read-only local mode, no silent failures |
| **Security/Privacy:** Data loss on archive/delete | Confirmation + undo for bulk archive; reversible deletes where feasible |
| **Scope:** Solo timeline | Weekly milestones on *working flows* (`ROADMAP.md`); cut attachments or dashboard polish before cutting parser/sync |

---

## Open Questions Impact

| Question | Blocks MVP? | Plan handling |
|----------|-------------|---------------|
| AppKit bridging timing | No | Start SwiftUI; revisit only if editor/outline blocked |
| Recurring task sync | No | Post-MVP |
| Menu bar quick-add | No | Post-MVP |
| Saved filter presets | No | Core focus views only in MVP |
| Markdown rich notes | No | Post-MVP |
| Theming controls | No | Post-MVP |
| Weekly review / usage insights | No | Later |
| SwiftData vs Core Data | No (decided) | **Core Data** per `ARCHITECTURE.md` |
| Optional per-action conflict override | No | Default local-change-wins only |
| Multiple images per item | No | In MVP scope; simplify schema to one-then-many if needed |
| Tags as first-class vs later | Partial | Support tags in parser if low cost; defer saved-filter UI |

**Blocking:** None (`OPEN_QUESTIONS.md`, `ARCHITECTURE.md` §12).

---

## Definition of Done

### MVP complete (release to dogfood)
- All `SPEC.md` acceptance criteria met.
- All `ARCHITECTURE.md` §10 “must pass” items met.
- `PRODUCT.md` success signals achievable: sub-5s capture, keyboard-heavy workflow, 3-week dogfood target started.
- No mandatory cloud account; local offline path verified.
- Known open questions either implemented as deferred or documented in `OPEN_QUESTIONS.md` without silent assumptions.

### Phase 0 complete
- Buildable macOS app, Core Data schema, Reminders spike documented, parser rules + manual checklist ready.

### Phase 1 complete
- Single-document editor round-trip without sync.

### Phase 2 complete
- Full MVP feature set in `SPEC.md` MVP Scope including dashboard, focus views, Reminders sync, images, keyboard commands, completed style, archive command.

### Phase 3 complete
- Hardened for daily use: performance, empty/error states, security safeguards, expanded QA docs.
