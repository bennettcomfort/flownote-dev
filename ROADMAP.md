# ROADMAP.md

## Phase 0: Planning / Setup
- Confirm the final planning set: PRODUCT.md, SPEC.md, OPEN_QUESTIONS.md, and ARCHITECTURE.md stay aligned.
- Set the MVP platform target to macOS 14 unless early validation supports lower without feature compromise.
- Validate the single-document MVP model and project-based filtered views as the initial information architecture.
- Define the first parser rules for task, project, note, @DONE, tags, and due-date metadata.
- Define the Core Data schema for Document, Item, DueDate, ReminderLink, Attachment, and SyncState.
- Validate Apple Reminders permission flow and core API surface before deep UI work.
- Decide the minimum visual design tokens for soft note styling, completed-item styling, and dashboard chips.
- Set up manual validation checklists for editor behavior, due-date parsing, offline persistence, and Reminders sync.

## Phase 1: MVP
- Build the main app shell with a default dashboard and navigation to Inbox, Today, Scheduled, Completed, Projects, and custom views.
- Implement the single-document local store and autosave behavior.
- Build the TaskPaper-style editor with create, edit, indent, outdent, reorder, and fold operations.
- Implement parser/classifier round-trip behavior so task, project, and note lines preserve formatting.
- Add combined task-and-note rendering with softer note styling.
- Implement due-date parsing for common phrases and soft confirmation for ambiguous phrases.
- Build focus views for Today, Upcoming, Overdue, and Someday.
- Implement completed-item rendering: muted gray, reduced opacity, @DONE inline, due date hidden, soft red strikethrough.
- Add command/menu action to auto-archive all @DONE items with confirmation and undo behavior.
- Implement Apple Reminders sync for create, update, complete, delete, and due-date changes.
- Apply local-change-wins as the default conflict resolution policy.
- Add visible sync status states for syncing, success, error, and conflict recovery.
- Implement offline-safe local editing and sync retry behavior when Reminders becomes available again.
- Support inline image paste and drag-drop with multiple in-editor image previews per item.
- Validate keyboard-first workflows for navigation, task actions, due-date entry, and archive command.

## Phase 2: Polish / Usability
- Improve dashboard hierarchy, due-today presentation, and quick-link chip interactions.
- Tune editor performance for larger documents and frequent filtering.
- Improve empty states for no tasks, no due items, no permissions, and no search/filter results.
- Refine soft styling for notes and completed items to ensure readability and low visual noise.
- Improve sync feedback, error recovery messaging, and retry affordances.
- Add import/export for the first portability format, prioritizing TaskPaper, then markdown/plain text if time allows.
- Add settings for small but useful controls such as default launch behavior and archive preferences.
- Expand manual QA with repeatable dogfooding scenarios and regression checklists.

## Phase 3: Advanced Features
- Add recurring task support where Apple Reminders allows reliable mapping.
- Add saved filters and context-tag views beyond the default focus views.
- Add menu bar quick add and optional global hotkey capture.
- Add basic markdown-style rich notes within items if it does not compromise editor speed.
- Add limited theming controls such as font size, line spacing, and translucency intensity.
- Explore AppKit bridging only if SwiftUI editor limitations block polish or correctness.

## Later / Maybe
- Weekly review assistant for inbox cleanup and deferred task triage.
- Lightweight usage insights such as completed tasks and overdue trends.
- Expanded portability beyond the first export format.
- More advanced project-specific views if the one-document model proves limiting.
- Optional per-action sync conflict override on top of local-change-wins default.
- Richer attachment handling beyond inline image previews.

## Roadmap Notes
- The MVP should optimize for trust, speed, and low cognitive overhead rather than feature breadth.
- If editor correctness and sync reliability conflict with polish work, prioritize correctness first.
- Do not expand into collaboration, AI copilots, cross-platform clients, or heavyweight planning views before the core daily workflow is proven.
- Weekly milestones should be measured against working flows, not just component completion.
