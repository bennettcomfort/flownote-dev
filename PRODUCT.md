# PRODUCT.md

## One-liner
Flownote is a minimal macOS scratchpad for tasks and notes, combining TaskPaper-style outlining with Apple Reminders sync and fast keyboard-first capture.

## Target User
Individual power users who manage tasks and notes across multiple apps and want a faster, lower-noise, macOS-native workflow.

## Core Problem
Task notes and reminders are fragmented across separate apps, which creates switching costs, duplicate entry, and a weaker sense of flow when moving between capture, planning, and follow-up.

## Primary Use Case
A user opens Flownote, quickly types tasks or notes into one editor, adds due dates when needed, and syncs selected items with Apple Reminders without leaving the app.

## Must-Have Features
- TaskPaper-style editor for tasks, projects, and notes.
- Intent-aware capture that classifies new entries from one input stream.
- Bi-directional Apple Reminders sync for create, update, complete, delete, and due date changes.
- Natural language due-date parsing for common phrases.
- Focus views for Today, Upcoming, Overdue, and Someday.
- Keyboard-first navigation and task actions.
- Local-first offline editing with background sync recovery.
- Inline image paste and lightweight previews.

## Nice-to-Have Features
- Recurring task sync where supported.
- Menu bar quick add and global hotkey.
- Saved filters by context tag.
- Basic markdown-style rich notes.
- Theming controls for typography and spacing.
- Import/export for TaskPaper and markdown.
- Weekly review assistant.
- Lightweight usage insights.

## Not Building Yet
- Team collaboration and shared workspaces.
- Cross-platform clients outside macOS.
- Calendar replacement features.
- Broad third-party integrations beyond Apple Reminders.
- Advanced AI copilots or autonomous planning.
- Heavy attachment management, OCR, or image annotation.
- Kanban, Gantt, mind maps, or similar heavyweight views.
- Plugin marketplace or public extension API.
- Enterprise auth, roles, SSO, or admin controls.
- Mandatory cloud accounts or remote-only storage.

## Success Criteria
- Users can capture and file a routine item in under 5 seconds.
- Apple Reminders sync is reliable for core create and update flows.
- Daily use holds up during dogfooding for at least 3 weeks.
- Most core actions can be completed from the keyboard.
- The app feels faster and less distracting than switching among separate notes and task apps.

## Risks / Assumptions
- Apple Reminders APIs and permissions are stable enough for dependable MVP sync.
- Users will accept a macOS-only release if the workflow is meaningfully better.
- SwiftUI may need AppKit bridging for the editor.
- Date parsing must be conservative enough to avoid user trust issues.
- Local-first behavior is a core differentiator and should stay simpler than cloud-first designs.