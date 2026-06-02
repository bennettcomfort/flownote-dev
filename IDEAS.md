# IDEAS.md

## Project Title
Flownote

## One-Sentence Summary
A minimal and light text-editor that is based on the taskpaper format and tasks engine with apple reminders sync, free form note taking, workflowey outlining, sleek add due dates

## Problem / Opportunity
Users struggle with fragmented task and note management across multiple apps (reminders, notes, to-do lists), lacking a unified, lightweight interface that combines TaskPaper's simplicity with Apple Reminders integration and natural outlining for both structured tasks and free-form thinking.

## Goal
To provide a single, intuitive application that streamlines task management and quick note-taking by integrating TaskPaper-like syntax with Apple Reminders, offering flexible outlining, and a clean, minimal user experience.

## Why This Matters
These types of apps are becoming popular for simplicity but also utility.

## Target Audience / Users
Individuals who manage tasks and notes across multiple platforms (Apple Reminders, general note apps) and are looking for a unified, minimalist, and efficient solution.

## Core Concept
The core concept is to create a "smart scratchpad" that understands and adapts to the user's input. It will primarily function as a text editor where users can quickly jot down thoughts, tasks, projects, and notes using a simple, intuitive syntax inspired by TaskPaper. The application will intelligently parse this input, distinguishing between tasks, notes, and projects.

## Scope
### In Scope
* TaskPaper-style editor and outlining for projects, tasks, and notes.
* Intent-aware capture that auto-classifies entries from one input stream.
* Apple Reminders bi-directional sync with due dates, status, and conflict handling.
* Natural language due-date parsing (e.g., tomorrow 3pm, next Monday).
* Focus views for Today, Upcoming, Overdue, and Someday.
* Keyboard-first workflow with command palette and fast task actions.
* Local-first, offline-reliable editing with safe background synchronization.
* Inline image support with quick paste and lightweight previews.

### Out of Scope
* Full team collaboration features (shared workspaces, mentions, comments, and real-time co-editing).
* Cross-platform support outside macOS for MVP (no Windows, Android, or web-first release).
* Calendar application replacement with full time-grid planning and event management.
* Email, chat, or communication hub integrations beyond core Apple Reminders sync.
* Advanced AI assistant features such as autonomous planning, task generation, or chat copilots.
* Deep file attachment management (versioning, galleries, cloud media libraries, and large file workflows).
* OCR pipelines, image annotation suites, and computer-vision automation in the first release.
* Gantt charts, Kanban boards, mind maps, and other heavyweight project-management views.
* Habit tracking, journaling streak systems, and gamification mechanics.
* Third-party plugin marketplace and public extension API.
* Multi-user admin, roles, permissions, SSO, and enterprise governance controls.
* Cloud-first account system with mandatory sign-in and remote-only data storage.

## Requirements
### Must-Have
* Users can create, edit, indent, outdent, reorder, and fold tasks, projects, and notes in a TaskPaper-style editor.
* The parser reliably distinguishes task, project, and note lines and preserves formatting through save and reload.
* Single-field capture supports fast entry and auto-classifies new lines into the correct item type.
* Apple Reminders sync is bi-directional for create, update, complete, delete, and due date changes.
* Sync behavior includes visible status states (syncing, success, error) and safe conflict resolution for edited items.
* Natural language date parsing supports common phrases like tomorrow 3pm, next Monday, and in two weeks.
* Focus views for Today, Upcoming, Overdue, and Someday are available and update in real time as items change.
* Core flows are fully keyboard-accessible, including command palette actions, navigation, quick schedule, and move operations.
* Local-first storage works offline with durable autosave and background sync recovery when connectivity returns.
* Inline image paste and drag-drop attachments render lightweight previews and remain linked to the related note or task.

### Nice-to-Have
* Recurring task rules synced to Apple Reminders where possible.
* Menu bar quick-add and global hotkey capture from anywhere in macOS.
* Saved filters by context tag (e.g., @work, @home, @deep).
* Basic markdown-style rich notes within task and project nodes.
* Theming controls (font size, line spacing, light translucency intensity).
* Import from plain TaskPaper files and export to markdown or TaskPaper.
* Simple weekly review assistant for inbox cleanup and deferred task triage.
* Lightweight usage insights (completed tasks, overdue trend, capture streak).

## Constraints
* Timeline: ship a usable MVP as fast as possible with iterative weekly milestones.
* Team: solo builder, so scope control and implementation simplicity are mandatory.
* Platform: macOS-first only for MVP, no parallel cross-platform builds.
* Stack: SwiftUI with SwiftData or Core Data, with selective AppKit bridging only if editor limitations require it.
* Integration dependency: Apple Reminders APIs and permission model may constrain sync behavior and UX flows.
* Performance target: smooth editing and filtering on large notes without visible lag.
* UX constraint: minimal, low-noise interface with keyboard-first interactions by default.
* Storage model: local-first architecture with safe sync retries and no mandatory cloud account.

## Assumptions
* Target users are individual power users who value speed, keyboard control, and low visual complexity.
* Users will accept a macOS-only MVP if it is meaningfully faster and cleaner than alternatives.
* Apple Reminders permissions and APIs are stable enough for reliable core sync.
* Natural language due-date parsing can be accurate enough for everyday phrases in MVP.
* Most users need inline image context, but not OCR or advanced annotation in first release.
* Local-first reliability and trust will be a stronger differentiator than broad integrations.
* A text-first interface can still feel modern and approachable with careful interaction design.
* Weekly shipping cadence is achievable for a solo builder with strict scope discipline.

## Risks / Concerns
* Sync edge cases can create duplicate tasks or mismatched completion states across systems.
* Parsing ambiguity may misclassify mixed lines, reducing trust in intent-aware capture.
* Date parsing errors can schedule tasks incorrectly and cause users to miss commitments.
* Large documents may degrade editor responsiveness without careful indexing and rendering.
* Keyboard-first power may raise onboarding friction for less technical users.
* SwiftUI editor constraints may require AppKit integration, increasing implementation complexity.
* Permission denial for Reminders can weaken the core value proposition unless fallback UX is strong.
* Scope creep into AI, collaboration, or heavy PM features can delay MVP significantly.

## Success Criteria
* Personal daily active usage is sustained for at least 21 of 30 consecutive days.
* Average capture-to-saved-task time is under 5 seconds for routine entries.
* Apple Reminders sync success rate remains above 99 percent for core create and update operations.
* Date parsing accuracy for common phrases is at least 95 percent in manual validation tests.
* Crash-free sessions exceed 99.5 percent during internal MVP usage period.
* 80 percent or more of core actions are completed via keyboard in typical workflows.
* Overdue task drift decreases after 4 weeks of use compared with baseline behavior.
* At least 3 external alpha users report replacing part of their previous notes-task workflow.

## Inspirations / References
* TaskPaper for clean text-first task structuring.
* Workflowy for fluid outlining and collapse-expand navigation.
* Apple Reminders for trusted due-date and completion sync baseline.
* Things 3 for interaction polish and calm productivity UX.
* Drafts for frictionless capture from anywhere.
* Bear for minimalist writing environment and visual simplicity.

## Non-Goals
* Build an all-in-one team collaboration suite.
* Compete as a full project-management platform.
* Replace calendar apps for event planning and scheduling.
* Introduce mandatory user accounts before proving local-first value.
* Ship advanced AI automation in MVP.
* Support every integration before the core editor and sync loop is excellent.

## Open Questions
* Should completed items auto-archive by default or remain visible until manual cleanup?
* Which conflict resolution rule should win first in sync collisions: local change, latest timestamp, or user prompt?
* How aggressively should natural language parsing infer dates versus requiring confirmation?
* Should notes and tasks share one list view or split into separate modes with quick toggles?
* What is the right default focus view on app launch for habit-forming daily use?
* When should AppKit bridging be introduced if SwiftUI editor limits appear?
* What file export format should be prioritized first for user portability?
* What minimum macOS version best balances reach and modern API availability?

## Notes
* Current direction: MVP optimized for speed, trust, and low cognitive overhead.
* Builder profile: solo development with rapid iteration and strict anti-scope-creep discipline.
* Preferred architecture: SwiftUI plus SwiftData or Core Data, with AppKit used only when needed.
* Product principle: one excellent daily workflow beats many partially complete features.
* Release strategy: private dogfooding first, then a small alpha group, then broader beta.