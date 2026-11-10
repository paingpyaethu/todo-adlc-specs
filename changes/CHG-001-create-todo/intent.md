# Intent: Create a Todo Task

- Change ID: CHG-001-create-todo
- Author: paingpyaethu (product owner)
- Status: draft  <!-- draft | accepted | rejected | superseded; only a human sets accepted/rejected -->
- Date: 2026-11-10
- Source: idea
- Linked record: none

## Problem
Users cannot create and save todo tasks from the mobile app because this feature has not been implemented yet.

## Proposed outcome
Users should be able to:

- Open the mobile app.
- Enter a task title.
- Tap the "Add Task" button.
- See the newly created task in their task list.
- Close and reopen the app without losing their saved tasks.

Implement the smallest working todo creation feature that involves both the backend API and the mobile application.

## Affected users and systems
- Users: anyone who wants to organize and manage their daily tasks using a mobile application.
- Systems: todo-adlc-api (stores and returns tasks), todo-adlc-mobile (task entry and task list). Neither exists yet; both are built from scratch in this change.

## Constraints
- No user authentication or registration in this change (out of scope, below).
- The security baseline skill applies. Its rule 1 requires authenticated endpoints, which conflicts with the constraint above. The spec must resolve this or record an exception (open question 1).
- Technology and architecture decisions are made in the Design stage, not here.

## Out of scope
- Editing or deleting tasks.
- Marking tasks as completed.
- User authentication and registration.
- Task priorities, categories, and reminders.
- Push notifications.
- Task sharing and collaboration.

## Success measure
- On the agreed platform(s), a user can enter a title, tap "Add Task", see the task in their list, and still see it after closing and reopening the app.
- CHG-001 validates the ADLC process: it passes requirement analysis, design, implementation planning, development, testing, review and approvals (gates G1 to G6), with every artifact committed.

## Open questions
1. Without login, whose tasks does a user see: their own per app install, or one list shared by everyone? How does the answer meet, or formally except, security baseline rule 1? (Deferred to Design by the originator.)
2. Which platforms must CHG-001 support: iOS, Android, or both? (Deferred to Design.)
3. Where must CHG-001 work to count as done: a simulator or emulator against a local API, or a real phone against a hosted dev environment? (Deferred to Design.)
4. If the phone has no connection when the user taps "Add Task", should the app show an error, or save the task and sync later? (Deferred to Design.)
5. What are the rules for a title: empty or whitespace-only, maximum length, duplicates allowed?
6. In what order does the list show tasks, and what does it show when there are none?

## Evidence (incident intents only)
Not applicable.
