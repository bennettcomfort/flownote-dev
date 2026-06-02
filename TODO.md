# TODO.md

## Rules

- Complete **one task at a time**; check it off in **Done** before starting the next.
- Keep changes small and focused (prefer touching a small set of files per task).
- Do **not** work on **Later** tasks unless explicitly asked.
- Do **not** resolve open questions silently; update `OPEN_QUESTIONS.md` if a decision is made.
- If a task depends on a blocking open question, move it to **Blocked** with the exact blocker.
- Update this file when tasks are completed, discovered, or blocked.
- Local MLX + OpenCode: see [`docs/local-agent-workflow.md`](docs/local-agent-workflow.md).

---

## Now

_Phase 0 foundation + Phase 1 smallest vertical slice (`PLAN.md`)._

- [ ] Create Xcode macOS app project (SwiftUI shell, macOS 14 deployment target)
  - Outcome: Empty app builds, launches, and shows a placeholder main window.
  - Depends on: None
  - Validation: Xcode build succeeds; app runs locally.
  - Notes: Align target name with product (Flownote). Configure signing for local dev.

- [ ] Scaffold folder structure per `ARCHITECTURE.md` §6
  - Outcome: `App/`, `Features/`, `Domain/`, `Persistence/`, `Services/`, `Tests/` directories exist with placeholder groups in Xcode.
  - Depends on: Create Xcode macOS app project
  - Validation: Project navigator matches planned layout; build still succeeds.

- [ ] Add XCTest targets (unit + integration) with passing placeholder tests
  - Outcome: `Tests/Unit` and `Tests/Integration` run from Xcode and report green.
  - Depends on: Scaffold folder structure
  - Validation: `Cmd+U` passes with at least one test per target.

- [ ] Define Core Data model for MVP entities
  - Outcome: `Document`, `Item`, `DueDate`, `ReminderLink`, `Attachment`, `SyncState` entities with fields/relationships from `ARCHITECTURE.md` §4.
  - Depends on: Scaffold folder structure
  - Validation: Model compiles; lightweight migration loads in app; relationships match plan.

- [ ] Implement `CoreDataStack` and document repository skeleton
  - Outcome: App initializes persistent container; `DocumentRepository` can create/fetch the single MVP document.
  - Depends on: Define Core Data model
  - Validation: Launch app without crash; fetch returns empty or default document.

- [ ] Document parser line rules in `docs/parser-rules.md`
  - Outcome: Written rules for task, project, note lines, `@DONE`, tags, and due-date metadata.
  - Depends on: None (can parallelize after repo exists)
  - Validation: Rules reviewed against `SPEC.md` functional requirements.

- [ ] Add `docs/dev-setup.md` (build, run, Reminders entitlement notes)
  - Outcome: New contributor can build app and understand Reminders capability requirements.
  - Depends on: Create Xcode macOS app project
  - Validation: Steps match actual project; no secrets committed.

- [ ] Seed manual validation checklist in `docs/manual-qa.md`
  - Outcome: Checklist covers editor, due dates, sync, offline, permissions, images from `ARCHITECTURE.md` §10.
  - Depends on: None
  - Validation: Each MVP acceptance area has at least one manual step.

- [ ] Run Apple Reminders API spike (permission + CRUD smoke test)
  - Outcome: Documented results for request permission, create, read, update, complete, delete test reminder; failure modes noted.
  - Depends on: Create Xcode macOS app project
  - Validation: Spike code or test runs on dev machine; findings in `docs/reminders-spike.md`.

- [ ] Define minimum design tokens (note, completed, dashboard chip)
  - Outcome: SwiftUI `Color`/`Font` constants or asset catalog entries for soft note, completed line, and chip styles.
  - Depends on: Scaffold folder structure
  - Validation: Preview or snapshot documents intended opacity, gray, strikethrough behavior.

- [ ] Implement domain `Item` types and parser classification (unit tests only)
  - Outcome: `Parser` classifies lines as task, project, or note from fixtures.
  - Depends on: Document parser line rules
  - Validation: XCTest covers happy paths and ambiguous/mixed lines.

- [ ] Implement parser text round-trip (structured items → canonical text)
  - Outcome: Parsed items serialize back to stable text without losing type or content.
  - Depends on: Implement domain `Item` types and parser classification
  - Validation: Round-trip unit tests pass for multi-line fixtures.

- [ ] Implement `ItemRepository` save/load for one document
  - Outcome: Items persist to Core Data and reload into domain models.
  - Depends on: Implement `CoreDataStack` and document repository skeleton
  - Validation: Integration test saves and fetches item graph.

- [ ] Wire minimal `OutlineService` (insert line, update line text)
  - Outcome: Domain API to add/update items without indent/outdent/reorder yet.
  - Depends on: Implement `ItemRepository` save/load
  - Validation: Unit tests for insert/update operations.

- [ ] Build minimal editor UI (single `TextEditor` bound to document text)
  - Outcome: User can type TaskPaper-like content in the main window.
  - Depends on: Scaffold folder structure
  - Validation: Manual typing works; window resizes correctly.

- [ ] Connect editor to parser + persistence with debounced autosave
  - Outcome: Edits parse to items, save locally, and reload on next launch with same structure.
  - Depends on: Parser round-trip, `ItemRepository`, minimal editor UI
  - Validation: Manual test—mixed task/project/note lines survive quit/relaunch; unit tests still green.

- [ ] Phase 1 acceptance pass (save/reload fidelity)
  - Outcome: Confirmed against `SPEC.md` “preserves content faithfully across save and reload.”
  - Depends on: Connect editor to parser + persistence
  - Validation: Manual QA + parser/integration tests; record result in `docs/manual-qa.md`.

---

## Next

_Phase 2 core MVP + Phase 3 hardening (`PLAN.md`)._

### Editor and outline

- [ ] Implement indent and outdent in `OutlineService`
  - Outcome: Parent/child relationships and `orderIndex` update correctly.
  - Depends on: Phase 1 acceptance pass
  - Validation: Unit tests for nested structure; manual indent in editor.

- [ ] Implement reorder (move up / move down) in `OutlineService`
  - Outcome: Sibling order changes persist.
  - Depends on: Implement indent and outdent
  - Validation: Unit tests; manual reorder.

- [ ] Implement fold and unfold per item
  - Outcome: `folded` flag hides/shows children in UI.
  - Depends on: Implement indent and outdent
  - Validation: Unit + manual fold test.

- [ ] Apply softer visual styling for note lines in editor
  - Outcome: Notes distinguishable from tasks per design tokens.
  - Depends on: Define minimum design tokens
  - Validation: Visual check against `OPEN_QUESTIONS.md` completed-style spec for notes.

- [ ] Implement completed-item rendering (opacity, gray, `@DONE`, hide due, strikethrough)
  - Outcome: Completed tasks match `SPEC.md` / `OPEN_QUESTIONS.md` styling.
  - Depends on: Define minimum design tokens
  - Validation: Manual toggle complete; visual matches spec.

- [ ] Add auto-archive all `@DONE` command with confirmation and undo
  - Outcome: Menu/command removes or archives completed items with safeguard.
  - Depends on: Implement completed-item rendering
  - Validation: Manual test with zero vs many `@DONE` items; undo works.

### Due dates

- [ ] Implement `DueDateParser` for common English phrases
  - Outcome: Parses `tomorrow 3pm`, `next Monday`, `in two weeks` to normalized dates.
  - Depends on: Phase 1 acceptance pass
  - Validation: Unit tests per phrase family in `SPEC.md`.

- [ ] Persist `DueDate` entities linked to items
  - Outcome: Parsed dates stored in Core Data and survive reload.
  - Depends on: Implement `DueDateParser`
  - Validation: Integration test round-trip.

- [ ] Add soft-confirmation UI for ambiguous date phrases
  - Outcome: Inline suggestion with accept/edit before finalizing.
  - Depends on: Implement `DueDateParser`
  - Validation: Manual ambiguous input test.

### Focus views and dashboard

- [ ] Implement `FocusQueryService` (Today, Upcoming, Overdue, Someday)
  - Outcome: Query API returns filtered item lists from local data.
  - Depends on: Persist `DueDate` entities
  - Validation: Unit tests with fixture dates.

- [ ] Build focus view UI screens
  - Outcome: User can navigate to each focus view and see correct items.
  - Depends on: Implement `FocusQueryService`
  - Validation: Manual test as dates change.

- [ ] Implement `DashboardService` (due-today + quick-link chip models)
  - Outcome: Dashboard data layer ready for launch surface.
  - Depends on: Implement `FocusQueryService`
  - Validation: Unit tests for due-today ordering and chip list.

- [ ] Build launch dashboard UI (default root on app open)
  - Outcome: App opens to dashboard with due-today times and chips (Inbox, Today, Scheduled, Completed, Projects, custom).
  - Depends on: Implement `DashboardService`
  - Validation: Manual launch test; matches `SPEC.md` dashboard requirements.

### Apple Reminders sync

- [ ] Define `RemindersSyncService` protocol and `SyncQueue` model
  - Outcome: Interfaces for enqueue, retry, and status match `ARCHITECTURE.md` §7.
  - Depends on: Run Apple Reminders API spike
  - Validation: Compiles; documented in code comments.

- [ ] Map local items to Reminders (create + `ReminderLink`)
  - Outcome: New local tasks can create remote reminders and store link ids.
  - Depends on: Define `RemindersSyncService` protocol; Phase 1 persistence
  - Validation: Manual create sync test.

- [ ] Implement update, complete, delete, and due-date push to Reminders
  - Outcome: Local lifecycle changes propagate remotely.
  - Depends on: Map local items to Reminders
  - Validation: Manual bi-directional test checklist.

- [ ] Implement pull remote changes into local store
  - Outcome: Remote edits update local items and `SyncState`.
  - Depends on: Map local items to Reminders
  - Validation: Edit in Reminders app; verify local update.

- [ ] Apply local-change-wins conflict policy with visible sync status
  - Outcome: Conflicts resolve per default; UI shows syncing/success/error/conflict.
  - Depends on: Implement pull remote changes
  - Validation: Simulated conflict test; status visible in UI.

- [ ] Handle Reminders permission denied/revoked gracefully
  - Outcome: Local-only mode with clear messaging; no crashes.
  - Depends on: Define `RemindersSyncService` protocol
  - Validation: Manual deny permission test.

- [ ] Implement offline-first write + background sync retry
  - Outcome: Edits always save locally; queue retries on lifecycle/sync-now.
  - Depends on: Define `SyncQueue`
  - Validation: Airplane-mode edit test; sync resumes when available.

### Attachments

- [ ] Implement `AttachmentService` (store file in app container, thumbnail)
  - Outcome: Image saved under app container with metadata on `Attachment` entity.
  - Depends on: Define Core Data model
  - Validation: Integration test write/read file.

- [ ] Wire paste and drag-drop to attach images to current item
  - Outcome: Multiple images per item supported per `SPEC.md`.
  - Depends on: Implement `AttachmentService`; minimal editor UI
  - Validation: Manual paste test; unsupported type handled.

- [ ] Show inline image previews in editor
  - Outcome: Lightweight previews linked to items.
  - Depends on: Wire paste and drag-drop
  - Validation: Manual multi-image test.

### Keyboard and commands

- [ ] Register `AppCommands` for core actions (new, complete, indent, move, fold, due date, sync, focus, archive)
  - Outcome: Menu commands and shortcuts invoke domain services.
  - Depends on: Outline ops; due dates; sync service stubs as needed per command
  - Validation: Keyboard-only walkthrough of main flows.

- [ ] Add command palette entry point in app shell
  - Outcome: User can discover and trigger commands from palette.
  - Depends on: Register `AppCommands`
  - Validation: Manual palette search test.

### Integration and MVP sign-off

- [ ] Add integration tests: persistence round-trip
  - Outcome: Automated coverage for document + items save/load.
  - Depends on: Phase 1 acceptance pass
  - Validation: CI/local test green.

- [ ] Add integration tests: sync queue with mocked Reminders
  - Outcome: Queue enqueue/retry tested without live Reminders.
  - Depends on: Define `SyncQueue`
  - Validation: Tests pass without network.

- [ ] Add integration tests: conflict path (local-change-wins)
  - Outcome: Conflict scenario produces expected local state and flag.
  - Depends on: Apply local-change-wins conflict policy
  - Validation: Test documents expected behavior.

- [ ] MVP acceptance audit against `SPEC.md`
  - Outcome: All acceptance criteria checked off or filed as Bugs.
  - Depends on: Core Phase 2 feature tasks above
  - Validation: `docs/manual-qa.md` completed for MVP.

### Phase 3 hardening (after MVP feature-complete)

- [ ] Add empty states (no items, no permission, empty focus view)
  - Outcome: UI handles empty cases without blank confusion.
  - Depends on: MVP acceptance audit
  - Validation: Manual empty-state pass.

- [ ] Profile and improve large-document editor performance
  - Outcome: Acceptable responsiveness on realistic outline size.
  - Depends on: MVP acceptance audit
  - Validation: Manual test with large fixture doc.

- [ ] Harden sync error recovery messaging and retry affordances
  - Outcome: User understands failures and can retry.
  - Depends on: Implement offline-first write + background sync retry
  - Validation: Manual failure injection test.

- [ ] Security pass: attachment container-only, destructive action safeguards
  - Outcome: Matches `ARCHITECTURE.md` §8.
  - Depends on: Attachments + auto-archive command
  - Validation: Checklist in `docs/manual-qa.md`.

- [ ] Update `docs/manual-qa.md` and known Reminders limitations
  - Outcome: Dogfooding-ready documentation.
  - Depends on: MVP acceptance audit
  - Validation: Reviewed before 3-week dogfood start.

---

## Later

_Post-MVP (`PLAN.md` Later / `OPEN_QUESTIONS.md` can-wait items). Do not start unless asked._

- [ ] TaskPaper import/export (first portability format)
  - Outcome: User can export/import TaskPaper-compatible text.
  - Depends on: MVP acceptance audit
  - Validation: Round-trip sample file test.

- [ ] Markdown and plain-text export
  - Outcome: Secondary export formats available.
  - Depends on: TaskPaper import/export
  - Validation: Sample export files open correctly.

- [ ] Recurring task sync (where Reminders supports)
  - Outcome: Recurrence rules map when possible.
  - Depends on: MVP sync stable
  - Validation: Manual recurrence cases.

- [ ] Menu bar quick-add and global hotkey
  - Outcome: Capture from anywhere on macOS.
  - Depends on: MVP acceptance audit
  - Validation: Manual hotkey test.

- [ ] Saved filters / context tag views beyond core focus views
  - Outcome: User-defined filters (e.g. `@work`).
  - Depends on: Parser tag support
  - Validation: Filter UI tests.

- [ ] Markdown-style rich notes within items
  - Outcome: Lightweight rich text without hurting editor speed.
  - Depends on: MVP editor stable
  - Validation: Performance check.

- [ ] Theming controls (font, spacing, translucency)
  - Outcome: User preferences for appearance.
  - Depends on: MVP UI stable
  - Validation: Settings persist.

- [ ] Weekly review assistant
  - Outcome: Guided inbox/deferred triage flow.
  - Depends on: Focus views + dashboard
  - Validation: Manual review session.

- [ ] Lightweight usage insights
  - Outcome: Optional metrics (completed, overdue trend).
  - Depends on: MVP dogfood data policy decision
  - Validation: Privacy review.

- [ ] AppKit editor bridge (only if SwiftUI blocks outline correctness)
  - Outcome: AppKit-backed editor replaces or augments SwiftUI surface.
  - Depends on: Documented SwiftUI limitation
  - Validation: Outline ops regression tests pass.

- [ ] Optional per-action sync conflict override
  - Outcome: User can override default local-change-wins per action.
  - Depends on: Stable default conflict policy
  - Validation: Manual override scenarios.

- [ ] Evaluate SwiftData migration (only if Core Data cost is high)
  - Outcome: Written decision; migration plan or explicit reject.
  - Depends on: MVP shipped
  - Validation: ADR or `OPEN_QUESTIONS.md` update.

- [ ] CI pipeline (build + test on push)
  - Outcome: Automated build/test for macOS target.
  - Depends on: XCTest targets stable
  - Validation: Green CI on main branch.

---

## Bugs

_Known issues from planning; add discoveries here._

- (none yet)

---

## Blocked

_Tasks waiting on unresolved decisions. None blocking MVP per `OPEN_QUESTIONS.md`._

- [ ] AppKit editor bridge implementation
  - Blocked by: Whether SwiftUI editor is insufficient (`OPEN_QUESTIONS.md` — can wait).
  - Unblock when: SwiftUI prototype fails outline/fold/keyboard requirements with documented repro.

- [ ] Optional per-action sync conflict override UI
  - Blocked by: Product decision on override UX (`ARCHITECTURE.md` §12).
  - Unblock when: Decision recorded in `OPEN_QUESTIONS.md`.

---

## Done

_Planning workflow (pre-implementation)._

- [x] `IDEAS.md` brainstorm captured
- [x] `PRODUCT.md` defined
- [x] `SPEC.md` defined
- [x] `OPEN_QUESTIONS.md` captured (no blocking questions)
- [x] `ARCHITECTURE.md` defined
- [x] `ROADMAP.md` drafted
- [x] `PLAN.md` created (Stage 4)
- [x] `TODO.md` created (Stage 5)
- [x] `AGENTS.md` created (Stage 6)
- [x] `RULES.md` created (Stage 6 quick reference)
- [x] Git repository initialized and pushed to GitHub
