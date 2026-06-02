# SPEC.md

## Goal
Build a small, reliable macOS app that lets a single user capture, organize, and sync tasks and notes in one lightweight TaskPaper-style editor.

## MVP Scope
- A single, text-first editor for tasks, projects, and notes.
- One-document MVP model with project-based filtered views.
- Intent-aware parsing of lines into item types.
- Core outline operations: create, edit, indent, outdent, reorder, and fold.
- Apple Reminders bi-directional sync for core item lifecycle changes.
- Natural language due-date parsing for common phrases.
- Focus views for Today, Upcoming, Overdue, and Someday.
- A launch dashboard with due-today items and quick-link chips.
- Keyboard-first navigation and actions.
- Local-first persistence with offline editing and background sync recovery.
- Inline image paste and simple previews, including multiple images per item.

## Functional Requirements
- Users can enter content in a single editor using TaskPaper-like syntax.
- The app distinguishes task, project, and note lines and preserves formatting after save and reload.
- Users can indent, outdent, reorder, and fold outline items.
- New entries are auto-classified from the capture stream.
- Users can set due dates using natural language such as tomorrow 3pm, next Monday, and in two weeks.
- Ambiguous date phrases use soft confirmation with an inline suggested date/time that users can accept or adjust.
- The app syncs creates, updates, completions, deletions, and due-date changes with Apple Reminders.
- Sync status is visible to the user and conflicts default to local-change-wins.
- Users can browse filtered focus views for Today, Upcoming, Overdue, and Someday.
- Users see a dashboard on launch with due-today items and due times, plus quick-link chips for Inbox, Today, Scheduled, Completed, Projects, and custom views.
- Users can complete core flows using the keyboard.
- The app stores data locally and continues working offline.
- Images pasted or dropped into the app render lightweight previews and remain attached to the related item, with multiple images allowed.
- Completed items remain visible by default and are displayed in a softened style: reduced opacity, muted gray tone, @DONE inline tag, due date hidden, and full-line red strikethrough.
- The app provides a command or menu action to auto-archive all @DONE items.

## Non-Functional Requirements
- Fast enough for large notes without visible lag during normal editing and filtering.
- Stable enough for daily dogfooding with minimal crashes.
- Offline edits must not be lost.
- Sync operations should recover automatically after temporary failures.
- UI should remain minimal and low-noise.
- Completed-state styling should use soft colors and remain readable.
- Behavior should be understandable without requiring a cloud account.

## Inputs
- User-typed editor content.
- Keyboard commands and command palette actions.
- Natural-language date strings.
- Apple Reminders data and permission state.
- Pasted or dragged image files.
- Local persisted documents and item metadata.

## Outputs
- Updated local task and note documents.
- Synced Apple Reminders items.
- Focus views and filtered item lists.
- Visible sync state, conflict state, and parsing results.
- Inline image previews.
- Dashboard cards/chips and due-today summary.

## Edge Cases
- Reminders permission is denied or revoked.
- Sync conflicts occur after edits in both systems.
- Date parsing is ambiguous or incomplete.
- Large documents cause slow filtering or editing.
- Mixed lines contain both note-like and task-like content.
- Offline edits accumulate while the app is closed and reconnect later.
- Image paste fails or unsupported file types are dropped.
- Bulk auto-archive command is triggered with no completed items present.

## Acceptance Criteria
- A user can create and update tasks, projects, and notes without leaving the editor.
- The app preserves content faithfully across save and reload.
- Core Reminders sync flows work in both directions for supported item changes.
- Conflict resolution follows local-change-wins in default mode.
- A user can set a due date from natural language without needing a separate date picker for common phrases.
- For ambiguous date phrases, the app shows a soft confirmation suggestion before finalizing.
- Focus views reflect item changes in real time or near real time.
- The app works offline and safely resumes sync afterward.
- Keyboard-only use is viable for the main workflow.
- Completed items render with the specified subdued style and can be bulk archived via command/menu action.

## Out-of-Scope Items
- Team collaboration, shared workspaces, and real-time co-editing.
- Cross-platform support outside macOS.
- Calendar-style scheduling and event management.
- Deep file attachment management.
- OCR, annotation, or computer-vision automation.
- AI copilots or autonomous planning.
- Heavy project-management views such as Kanban or Gantt.
- Plugin ecosystem, enterprise auth, or mandatory cloud accounts.

## Assumptions
- The MVP can be delivered on macOS only, with a practical minimum target of macOS 14 unless validation supports lower.
- Date parsing ambiguity policy uses soft confirmation for ambiguous phrases.
- Local-first storage is the right default for trust and simplicity.
- AppKit bridging may be needed only if SwiftUI text editing limits appear.