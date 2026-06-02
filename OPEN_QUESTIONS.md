# OPEN_QUESTIONS.md

## Questions That Must Be Answered Before Building
- None currently.

## Questions That Can Wait
- When should AppKit bridging be introduced if SwiftUI editing is insufficient?
- Whether recurring task rules should sync where possible.
- Whether a menu bar quick-add entry point is needed in the first release.
- Which saved filter presets are most valuable beyond the core focus views.
- Whether markdown-style rich notes should be supported in MVP or after.
- Which theming controls matter most, if any.
- Whether a lightweight weekly review assistant should be planned for the next phase.
- Whether usage insights should be tracked at all in the early release.

## Product Risks
- Sync edge cases could create duplicates or mismatched completion states.
- Parsing errors could reduce trust in the app quickly.
- Date misinterpretation could cause missed commitments.
- Performance issues on large documents could undermine the lightweight promise.
- Keyboard-first workflows may increase onboarding friction for some users.
- Scope creep into advanced productivity features could delay the MVP.

## Technical Unknowns
- Whether SwiftUI text editing is sufficient for the required outline behavior.
- How much custom parsing is needed versus what can be handled with a simpler model.
- How to keep Reminders sync reliable without building a large conflict system.
- Whether local persistence should be SwiftData or Core Data for the MVP.
- How images should be stored locally while staying simple and durable.
- How to represent outline nodes, item types, and sync state with minimal complexity.
- Whether offline background recovery needs explicit retry UI or can stay automatic.

## Decisions Captured
- Completed item style: greyed, lower opacity, due date hidden, @DONE shown inline, and a red strikethrough across the full line using soft colors.
- Completed item lifecycle: keep visible in place by default; provide menu/app command to auto-archive all @DONE items.
- Sync conflict policy: local change wins by default.
- View model: combined list for notes and tasks, with notes visually softened.
- Default launch experience: dashboard showing due-today items with due times and quick-link chips for Inbox, Today, Scheduled, Completed, Projects, and custom views.
- Platform baseline: target recent macOS with a practical minimum of macOS 14 unless testing shows no feature compromise on lower versions.
- Export priority: TaskPaper first, then markdown and plain text.
- Document model: one document in MVP with project-based filtered views.
- Attachments: support multiple inline images per item in-editor for MVP.
- Date parsing ambiguity policy: soft confirmation. Ambiguous phrases show an inline suggested date/time that can be accepted quickly or edited.