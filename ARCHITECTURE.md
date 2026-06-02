# ARCHITECTURE.md

## 1. Architecture Summary
Flownote should be a small, macOS-native, local-first task and note editor built around a TaskPaper-style outline. The app’s technical job is to parse a single text-first document into structured items, keep that structure editable and durable offline, and synchronize a subset of items with Apple Reminders.

The smallest useful MVP architecture is a single-window SwiftUI app with one local persistence layer, one document model, one parser, one Reminders sync service, and one lightweight attachment path for inline images. Keep the system local-first, conservative about parsing, and simple enough that sync and editor behavior remain understandable.

## 2. Recommended Tech Stack
- Frontend: SwiftUI for the main app shell and navigation.
- Backend: None in the cloud; use local app services only.
- Database / storage: Core Data as the primary local store recommendation, with file-backed image storage in the app container. SwiftData is a possible later swap if the MVP stays simple enough, but Core Data is the safer default for sync metadata and explicit model control.
- Auth: None for MVP.
- AI/model layer: None for MVP.
- Background jobs: App lifecycle-based background sync and retry queues, not a separate job system.
- Testing tools: XCTest for unit and integration tests; manual validation for Reminders permission, sync, and editor behavior.
- Package manager: Swift Package Manager if dependencies are added later; avoid extra packages unless clearly justified.
- Deployment target: macOS only for MVP, with a practical minimum target of macOS 14 unless validation supports lower without compromising functionality.

## 3. System Components
- App shell and navigation: Owns the main window, command palette entry points, and focus view switching. MVP-critical.
- Dashboard module: Default launch surface showing due-today items with due times and quick-link chips to Inbox, Today, Scheduled, Completed, Projects, and custom views. MVP-critical.
- Outline editor: Renders and edits TaskPaper-style items, handles indentation, folding, reordering, and keyboard-first actions. MVP-critical.
- Parser and classifier: Converts text lines into task, project, and note entities and preserves formatting on save/load. MVP-critical.
- Local document store: Persists outline data, item metadata, sync state, and image references. MVP-critical.
- Due-date parser: Interprets natural-language dates and converts them to normalized dates. MVP-critical.
- Apple Reminders sync service: Maps local items to Reminders and handles create, update, complete, delete, due-date, and conflict flows. MVP-critical.
- Focus view service: Produces Today, Upcoming, Overdue, and Someday collections from local data and sync state. MVP-critical.
- Attachment handler: Stores pasted or dropped images locally and exposes lightweight previews. MVP-critical for the stated MVP, but intentionally minimal.
- Preferences/settings: Holds app-level choices such as completion display, archive behavior, and sync behavior. Future-later unless required for launch.
- Import/export: Not MVP-critical. Should stay out until the core editor and sync loop are proven.

## 4. Data Model
Core entities:
- Document: The editable top-level scratchpad or file-like container.
- Item: A single task, project, or note line with outline relationships.
- DueDate: Normalized scheduling data attached to an item.
- ReminderLink: Mapping between a local item and an Apple Reminders record.
- Attachment: Local image asset and its preview metadata.
- SyncState: Per-item and per-document status for syncing, conflict, and error handling.

Important fields:
- Document: id, title, createdAt, updatedAt, lastSyncedAt, version or change token.
- Item: id, documentId, parentId, orderIndex, itemType, text, rawLine, completedAt, folded, tags, dueDateId, reminderLinkId, createdAt, updatedAt.
- DueDate: id, itemId, parsedText, normalizedDate, confidence or parseSource.
- ReminderLink: id, itemId, reminderIdentifier, source, lastRemoteChangeAt, lastLocalChangeAt, syncStatus.
- Attachment: id, itemId, localFileURL, thumbnailURL, mimeType, createdAt.
- SyncState: status, lastError, retryCount, conflictFlag, pendingOperations.

Relationships:
- One Document has many Items.
- One Item may have zero or one parent Item.
- One Item may have zero or one DueDate.
- One Item may have zero or one ReminderLink.
- One Item may have zero or more Attachments, though MVP can start with one primary image attachment per item if that simplifies implementation.
- SyncState can be document-level plus item-level metadata.

Data ownership boundaries:
- Local app storage is the source of truth for the editor and offline editing.
- Apple Reminders is an external sync target, not the master copy.
- The parser owns conversion between raw text and structured item fields.
- The sync service owns translation between local items and Reminders records.

Unknowns that still need decisions:
- Whether tags and saved filters are first-class in MVP or later.

## 5. Data Flow
1. User types or pastes content into the editor.
2. The editor emits raw text changes and outline commands.
3. The parser/classifier converts lines into task, project, or note items and updates structure.
4. The local store saves the updated document and item graph immediately or via autosave.
5. If due-date text is detected, the due-date parser normalizes it into a stored date value.
6. If the item should sync with Apple Reminders, the sync service queues the change.
7. The sync service pushes local changes to Reminders and pulls remote changes back into local state.
8. Conflict handling resolves differences using local-change-wins as default and marks items with visible sync status as needed.
9. Focus views read from local data and filtering rules to show Today, Upcoming, Overdue, and Someday.
10. Image paste or drag-drop stores files locally, generates previews, and links them to the relevant item.
11. When an item is completed, due date text is hidden, @DONE is shown inline, and the line is rendered in softened completed-state styling; bulk auto-archive can move all @DONE items in one action.

## 6. Folder Structure
Keep the app conventional and shallow.

```text
Flownote/
  App/
    FlownoteApp.swift
    AppState.swift
    AppCommands.swift
  Features/
    Editor/
    FocusViews/
    Capture/
    Settings/
  Domain/
    Models/
    Parsing/
    DueDates/
    Sync/
    Attachments/
  Persistence/
    CoreDataStack.swift
    Repositories/
  Services/
    RemindersSyncService.swift
    OutlineService.swift
    AttachmentService.swift
  Resources/
    Assets.xcassets
  Tests/
    Unit/
    Integration/
```

This structure keeps feature code separate from domain logic and leaves room for later growth without overengineering.

## 7. API / Interface Boundaries
Major internal interfaces:
- OutlineService: apply editor actions such as insert, indent, outdent, reorder, and fold.
- Parser: convert raw text into structured items and back into stable text.
- DueDateParser: parse user text into normalized dates and confidence metadata.
- RemindersSyncService: create, update, complete, delete, and pull remote changes.
- AttachmentService: store image files and generate previews.
- FocusQueryService: produce Today, Upcoming, Overdue, and Someday results.
- DashboardService: provide due-today summaries and quick-link chip models for the launch dashboard.
- SyncQueue: hold pending local operations and retry them safely.

Likely app commands or user actions:
- New item
- New task
- Toggle complete
- Auto-archive all @DONE
- Indent / outdent
- Move up / move down
- Fold / unfold
- Set due date
- Sync now
- Show Today / Upcoming / Overdue / Someday

External interface boundary:
- Apple Reminders framework access should be isolated behind one sync service so permission handling and future adjustments stay localized.

## 8. Security and Privacy Considerations
- Secrets handling: MVP should not require stored secrets beyond system-managed Apple permissions. If any future tokens appear, keep them in the system keychain.
- User data handling: Keep all task and note data local by default; avoid mandatory cloud storage.
- Auth/session concerns: No user account session model for MVP.
- File system access concerns: Store attachments only inside the app container unless the user explicitly imports files. Do not scan arbitrary directories.
- External API concerns: Apple Reminders permissions must be requested clearly and handled gracefully if denied or revoked.
- Destructive action safeguards: Deletions, completion sync, and conflict resolution should be reversible where possible and surfaced with clear status before data loss becomes permanent; bulk auto-archive should use a confirmation and undo window.

## 9. Error Handling and Edge Cases
Expected failure modes:
- Reminders permission denied or revoked.
- Remote sync conflict after local editing.
- Temporary network or OS-level Reminders failures.
- Ambiguous due-date parsing.
- Bulk archive command accidentally triggered.
- Large outline documents causing slow refresh.
- Broken or unsupported pasted images.

Empty states:
- No items yet in the document.
- No Reminders account available.
- No due dates assigned.
- No items in a given focus view.

Invalid input:
- Mixed syntax that does not clearly match task, project, or note rules.
- Invalid date phrases.
- Broken indentation or malformed outline structure.

Permission issues:
- Reminders access denied.
- File access denied for imported images.

Network or model failures:
- Not relevant to an AI model in MVP.
- Sync retries should continue when the OS or account becomes available again.

## 10. Testing Strategy
Unit tests:
- Parser classification for task, project, and note lines.
- Round-trip save/load preservation.
- Due-date parsing for common phrases.
- Focus view filtering logic.
- Outline operations such as indent, outdent, reorder, and fold.

Integration tests:
- Local persistence round trip.
- Sync queue behavior with mocked Reminders responses.
- Conflict handling path for local versus remote edits.
- Attachment storage and preview creation.

End-to-end tests, if useful:
- Basic editor capture flow.
- Keyboard-only navigation through core actions.
- A sync-enabled flow with mocked or sandboxed Reminders access if feasible.

Manual validation checklist:
- Create, edit, indent, outdent, reorder, and fold items.
- Add due dates from natural language.
- Toggle completion and verify sync behavior.
- Confirm offline edits persist and later sync.
- Paste an image and confirm preview and linkage.
- Deny Reminders permission and verify graceful fallback.

What must pass before work is considered done:
- Core editor round-trip is stable.
- Due-date parsing works for the common phrases in the spec.
- Apple Reminders sync works for the MVP lifecycle operations.
- Offline editing is durable.
- Focus views update from local state correctly.
- Dashboard due-today and quick-link chips render correctly on launch.
- Completed-item visual state matches the product-defined style and archive command behavior.

## 11. Architecture Decisions
- Use a local-first model with no mandatory cloud account. This matches the product intent and reduces trust and sync complexity early.
- Keep the editor and parser separate. This makes round-trip preservation and future syntax changes easier to manage.
- Isolate Reminders access behind a dedicated sync service. This contains permission handling and external API churn.
- Prefer a conservative due-date parser with soft confirmation for ambiguous phrases.
- Keep attachments minimal and local. Inline images are valuable, but deep media management is explicitly out of scope.
- Recommend Core Data for the MVP storage layer. It is a practical choice for explicit relationships, sync metadata, and a mature macOS ecosystem.

Alternatives considered:
- SwiftData instead of Core Data: attractive for simplicity, but the MVP needs explicit control over relationships and sync metadata, so Core Data is the safer recommendation.
- Cloud-backed sync architecture: rejected for MVP because the product explicitly favors local-first and Apple Reminders as the main external sync target.
- More aggressive AI-assisted parsing: rejected because it would add uncertainty and scope without solving the core problem.

## 12. Open Architecture Questions
Blocking implementation:
- None currently.

Can wait:
- Whether AppKit bridging should be introduced immediately or only when SwiftUI editor limits appear.
- Whether recurring task rules should sync where possible.
- Whether a menu bar quick-add entry point is needed in the first release.
- Which saved filter presets are most valuable beyond the core focus views.
- Whether markdown-style rich notes should be supported in MVP or after.
- Which theming controls matter most, if any.
- Whether a lightweight weekly review assistant should be planned for the next phase.
- Whether usage insights should be tracked at all in the early release.

New technical questions:
- Should conflict policy include an optional per-action override even when default is local-change-wins?

## 13. MVP Architecture Boundaries
Included in the MVP architecture:
- One macOS app.
- One local document model.
- TaskPaper-style editing and outline manipulation.
- Combined notes and tasks in one view, with softer visual treatment for notes.
- Conservative parsing and due-date normalization.
- Apple Reminders sync service.
- Offline local persistence and retryable sync.
- Image attachment storage and previews, with multiple inline images per item.
- Dashboard as default launch surface with due-today and quick-link chips.
- Focus views for Today, Upcoming, Overdue, and Someday.

Intentionally excluded:
- Cloud accounts and remote-only storage.
- Collaboration and shared editing.
- Plugins, extensions, and public APIs.
- Heavy project management views.
- OCR, annotation, and advanced attachment workflows.
- AI copilots or automatic task generation.
- Broad third-party integrations beyond Apple Reminders.

Should not be built yet:
- A complex sync server.
- A general-purpose attachment library.
- Advanced theming and personalization systems.
- Enterprise identity and administration features.
- A broad import/export ecosystem beyond the first portability format.

Selected policy note:
- Date parsing ambiguity uses soft confirmation: suggest parsed date/time inline and allow one-step accept or edit.
