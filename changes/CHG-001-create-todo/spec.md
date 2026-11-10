# Spec: Create a Todo Task

- Change ID: CHG-001-create-todo
- Intent: `changes/CHG-001-create-todo/intent.md` at commit 1badd7f2a0c421a0958c55a826182f55e1d19c5d (merged to `main` by the product owner in PR #3, merge commit c5d76b1)
- Status: draft  <!-- draft | accepted | rejected; only a human sets accepted/rejected -->
- Risk class: high  <!-- see adlc/gates.md; reviewers may raise it, never lower it -->
- Repos affected: both
- Skills applied: write-spec v1 (2026-11-10), security-baseline v1 (2026-11-10)

Every decision marked **Pending G2** is a recommendation, not a decision. Merging this spec accepts the recommended option unless the approver edits or comments otherwise before merging.

## Summary

A user opens the mobile app, types a task title, taps "Add Task" and sees the task at the top of their list. Tasks are stored by the API, so they are still there after the app is closed and reopened. There is no login: each app install gets its own private list, protected by a random secret that the app generates and keeps in the device's secure storage. CHG-001 runs on a developer Mac (iOS Simulator and Android Emulator against an API running locally over TLS). It also chooses the stack and the API contract format for the pilot (D-003).

## Requirements

### Functional

- **FR-1** The app shows one screen with a title text field, an "Add Task" button and the list of tasks for this install.
- **FR-2** When the user taps "Add Task" with a valid title, the app sends the title to the API. On success, the new task appears at the top of the list and the text field is cleared.
- **FR-3** Title rules, enforced by the API and checked by the app before sending:
  - Leading and trailing whitespace is removed, and the trimmed title is what gets stored.
  - The trimmed title must be 1 to 200 characters (Unicode code points).
  - A title that is empty or only whitespace is rejected.
  - Duplicate titles are allowed.
- **FR-4** While a create request is in flight, the "Add Task" button is disabled, so a double tap cannot send two requests.
- **FR-5** The list shows tasks newest first (by creation time, then by ID for ties).
- **FR-6** When there are no tasks, the list area shows "No tasks yet. Add your first task above."
- **FR-7** On launch, the app loads the list from the API. The list survives closing and reopening the app.
- **FR-8** Each install sees only its own tasks. A request can never read or create tasks for another install.
- **FR-9** If a create request fails (no connection, timeout, server error or validation error), the app shows an error message, keeps the typed title in the field and does not add the task to the list. Nothing is queued for later.
- **FR-10** If loading the list fails on launch, the app shows an error message with a "Retry" button.
- **FR-11** The API limits each install to 500 tasks. Creating task 501 is rejected with a clear error.

### Non-functional

- **NFR-1 Platforms:** iOS and Android from one codebase. "Done" is verified on the iOS Simulator and the Android Emulator (both installed on the development Mac).
- **NFR-2 Environment:** local only. The API and its database run on the developer Mac. No hosted environment, cloud account or cost in CHG-001.
- **NFR-3 Offline:** online only (FR-9, FR-10). No local cache or sync.
- **NFR-4 Performance (local):** creating a task and listing tasks each respond within 300 ms at the 95th percentile on the development Mac with 500 tasks stored.
- **NFR-5 Accessibility (interim, no written policy yet):** the text field and button have accessible labels, text scales with the system font size, touch targets are at least 44 by 44 points, and text contrast is at least 4.5:1.
- **NFR-6 Localization:** English only. All user-visible strings sit in one place so translation can be added later.
- **NFR-7 Data:** test data only. No real users and no Yoma Fleet data (D-008).

### Acceptance criteria

1. Given an empty list, when the app opens, then it shows "No tasks yet. Add your first task above."
2. Given the user types "Buy milk" and taps "Add Task", then "Buy milk" appears at the top of the list and the field is empty.
3. Given tasks exist, when the app is closed fully and reopened, then the same tasks show in the same order.
4. Given the title is empty or only spaces, when the user taps "Add Task", then no request is sent and the app shows "Enter a task title."
5. Given a 201-character title, when it reaches the API directly, then the API returns 400 and stores nothing.
6. Given two installs A and B, when A creates a task, then B's list does not include it, and B cannot read it with any request.
7. Given a request with a missing or malformed credential, then the API returns 401 and reveals no task data.
8. Given the API is stopped, when the user taps "Add Task", then an error is shown, the title stays in the field and the list is unchanged.
9. Given an install already has 500 tasks, when it creates another, then the API returns 422 with error code `task_limit_reached` and the app shows "You have reached the 500 task limit."
10. Given the title " Buy milk " with surrounding spaces, then it is stored and shown as "Buy milk".
11. Given two tasks with the same title, then both are stored and both are shown.
12. The API test suite and the mobile test suite pass, and both repos pass lint, with output pasted in each PR (G4).

## Design

### User experience

- One screen. Top: a single-line text field (placeholder "What do you need to do?") and an "Add Task" button. Below: the list, newest first. Each row shows the title only.
- Pressing the keyboard's return key does the same as tapping "Add Task".
- Error messages appear inline under the text field, are announced by the screen reader and clear on the next successful action.
- No brand guide exists (see Areas of concern). Use the platform's default typography and colours, and support light and dark mode.

### API contract change

New API, version 1. The contract file (`contract/openapi.yaml` in todo-adlc-api, format **Pending G2**) is the source of truth. Summary:

| Operation | Auth | Request | Responses |
|---|---|---|---|
| `GET /health` | None (the one listed exception to baseline rule 1) | none | 200 `{"status":"ok"}` |
| `GET /v1/tasks` | Install credential | none | 200 `{"items":[Task]}` newest first, at most 500; 401 |
| `POST /v1/tasks` | Install credential | `{"title": string}`, no other fields | 201 Task; 400 validation; 401; 422 `task_limit_reached` |

- `Task` = `{"id": UUID, "title": string, "createdAt": RFC 3339 UTC timestamp}`.
- Install credential: header `Authorization: Bearer <token>`, where the token is 32 random bytes encoded as base64url (43 characters). Anything else gets 401.
- Unknown request fields are rejected with 400 (baseline rule 3).
- Errors use `application/problem+json` (RFC 9457) with a stable `code` field (`validation_failed`, `unauthorized`, `task_limit_reached`).
- Backward compatibility: not applicable (first version). Later breaking changes go to `/v2`.

### Data

- **installation**: `id` (UUID), `token_hash` (SHA-256 of the token, unique), `created_at`. The API creates an installation the first time it sees a well-formed token. It never stores the token itself.
- **task**: `id` (UUID, set by the server), `installation_id` (foreign key), `title` (text, 1 to 200 characters), `created_at` (UTC). Index on (`installation_id`, `created_at` descending).
- **Personal data:** task titles are free text and may contain personal data, so they are treated as personal data. The install token is a credential.
- **Retention:** kept until the local database is reset. No deletion feature in this change (out of scope). If the app is uninstalled, its token is lost and its tasks become unreachable. There is no recovery without accounts.
- Schema changes go through versioned migration files in todo-adlc-api.

### Security and privacy

How each security-baseline v1 rule applies:

1. **Authentication and authorization:** every endpoint except `GET /health` requires the install credential. Every query is scoped to the caller's `installation_id`. This is an anonymous per-install credential, not user authentication; see Areas of concern #1.
2. **Queries:** parameterized queries only, using the database driver's placeholders.
3. **Input:** request bodies are validated against the contract schema, and unknown fields are rejected. Title rules are as in FR-3.
4. **Secrets:** no secrets in code. Database connection settings come from the environment, and only `.env.example` is committed. The install token is generated on the device using the platform's secure random source and stored in iOS Keychain or Android Keystore-backed storage.
5. **Transport:** the API serves HTTPS only, even locally. A local development certificate authority signs the local API certificate and is trusted on the simulator and emulator. Certificate verification is never disabled. See Areas of concern #2.
6. **No auth bypass:** tests use generated test tokens. No test-only route skips the credential check.
7. **Logging:** never log the `Authorization` header, tokens, token hashes or task titles. Log request ID, route, status and duration only.
8. **Data at rest:** see Areas of concern #3. CHG-001 runs locally only.
9. **Least privilege:** no cloud resources in this change. The API's database user owns only its own schema.
10. **Pipeline:** CI with SAST and SCA arrives in Phase 3. GitHub secret scanning and push protection are already on. See Areas of concern #4.
11. **Regulated data:** none. The app collects no identity, financial or location data.

Abuse controls: a per-install cap of 500 tasks (FR-11), plus a request rate limit per token and per client IP. These limits matter little locally but must exist before any hosted environment.

## Stack and dependency decisions

All decisions in this section are **Pending G2**. What was observed on the development Mac (2026-11-10): Node.js 25 with npm, pnpm and bun, Python 3.14, Go 1.27, Java 17, Flutter 3.44 with Dart 3.12, Xcode 27.2 with iOS Simulator runtimes, the Android SDK with an emulator, and Docker 29. Fit with the team's skills is unknown, so the owner should confirm it. Exact versions are pinned in each plan.md (G3), using the current LTS or stable release at that time.

### S-1 API contract format: recommend OpenAPI 3.1

| Option | Trade-offs |
|---|---|
| **OpenAPI 3.1 (YAML)** | Mature, REST fits the two simple operations, and the schema doubles as the input validator (rule 3). Mobile can generate types from it, and many tools can lint and test against it. No lock-in. |
| GraphQL SDL | Flexible queries, but this needs no flexibility. It adds a server runtime and client library, and authorization per field is harder to review. |
| Protocol Buffers with gRPC | Strong typing and fast, but weak mobile tooling, needs a proxy for some clients, and errors are harder to read while debugging. Overkill here. |

### S-2 API stack: recommend TypeScript on Node.js LTS with Fastify and PostgreSQL

| Option | Trade-offs |
|---|---|
| **TypeScript, Node.js LTS, Fastify, PostgreSQL** | Same language as the recommended mobile stack, so one toolchain and the contract types are shared. Fastify validates requests with JSON Schema built in, which lines up with OpenAPI. Large ecosystem, cheap to host anywhere later. Risk: npm supply-chain exposure; SCA in Phase 3 mitigates it. |
| Python, FastAPI, PostgreSQL | Fast to write, generates OpenAPI from code. But the contract would then be derived from code, which contradicts contract first, and it means a second language next to the mobile app. |
| Go, chi router, PostgreSQL | Small, fast single binary, and a strong standard library cuts dependencies. More code for validation, and a second language next to the mobile app. |

Database: **PostgreSQL** in a Docker container for local work (it maps directly to managed PostgreSQL with encryption at rest later). Rejected: SQLite (simplest locally, but would need a migration to move to a hosted multi-instance setup).

Dependencies named by the recommendation (rule 9: nothing else without a spec change):
- Runtime: `fastify`, `pg` (node-postgres), `node-pg-migrate`.
- Dev and test: `typescript`, `vitest`, `eslint` with `typescript-eslint`, `@redocly/cli` (contract lint).
- Local tooling: Docker Compose (database), `mkcert` (local certificate authority, rule 5).

### S-3 Mobile stack: recommend React Native with Expo (TypeScript), iOS and Android

| Option | Trade-offs |
|---|---|
| **React Native with Expo, TypeScript** | One codebase for both platforms. Same language as the API, so OpenAPI types are shared. Secure storage through `expo-secure-store`. Builds run locally (`expo run:ios`, `expo run:android`); Expo's cloud build service (EAS) is not adopted, which avoids cost and lock-in. Risk: framework upgrades every few months. |
| Flutter (Dart) | One codebase and good performance, and it is installed here. A third language next to the API, and OpenAPI client generation for Dart is less mature. |
| Native Swift (SwiftUI) plus Kotlin (Jetpack Compose) | Best platform fit, but two codebases double the work and the review effort for every change. Too heavy for this pilot. |

Dependencies named by the recommendation:
- Runtime: `expo`, `react`, `react-native`, `expo-secure-store`, `expo-crypto` (secure random token).
- Dev and test: `typescript`, `jest` with `jest-expo`, `@testing-library/react-native`, `eslint` with `typescript-eslint`, `openapi-typescript` (types from the pinned contract).
- Out of scope: navigation libraries, state management libraries, UI kits, analytics and crash reporting. A future change adds them through its own spec.

### S-4 Install credential scheme: recommend a token generated on the device

| Option | Trade-offs |
|---|---|
| **Device generates a 32-byte random token; the API stores its hash and creates the installation on first use** | No unauthenticated registration endpoint, and no secret ever crosses the network except as a bearer credential over TLS. Weakness: anyone can make new installs (mitigated by the rate limits). |
| API issues the token from `POST /v1/installations` | The server controls issuance, but that endpoint must be unauthenticated, which is a second exception to rule 1. |
| One shared list for everyone, no credential | Simplest. It breaks rule 1 outright, any user could see every task, and it would be thrown away when accounts arrive. Rejected. |

## Sequencing across repos

1. **todo-adlc-api first:** plan.md, then a PR containing `contract/openapi.yaml`, migrations, the two endpoints, health, and tests. Merging it (G5) and tagging `contract-v1.0.0` publishes the contract.
2. **todo-adlc-mobile in parallel against the contract:** the mobile plan.md pins the contract by the API repo commit SHA of the contract PR. Until the API is merged, mobile tests use a fake HTTP layer that returns responses valid against the pinned schema. The mobile PR merges after the API PR. Its final check runs the app on both simulators against the locally running API.
3. Both plan.md files are written against the same commit of this spec.
4. Cross-repo sequencing owner: tech lead (paingpyaethu, D-007).

## Areas of concern

| # | Concern | Policy or source | Owner | Resolution |
|---|---|---|---|---|
| 1 | No login (intent) versus "every endpoint requires authentication" | security-baseline rule 1 | paingpyaethu (policy owner) | Proposed: the per-install bearer token (S-4) counts as authentication for CHG-001, and every request is authorized against its installation. It is anonymous, has no recovery, and tasks are lost on uninstall. **Needs explicit policy owner sign-off at G2.** Revisit when accounts are introduced. |
| 2 | TLS for local development: the simulator and emulator call an API on the Mac | security-baseline rule 5 | paingpyaethu | Proposed: a local certificate authority (`mkcert`), trusted on the simulator and emulator, with verification left on. No plain-HTTP exception requested. If setup proves impractical, raise a spec change; never disable verification. |
| 3 | Task titles may contain personal data; the local Docker database volume is not encrypted by PostgreSQL itself | security-baseline rule 8 | paingpyaethu | Proposed exception for CHG-001 only: local test data only, protected by the Mac's full-disk encryption (FileVault; owner to confirm it is on). Expires at the first hosted environment, which must use encrypted managed storage. |
| 4 | No CI yet, so no SAST or SCA, and no signed releases | security-baseline rule 10; gates G4 to G6 | paingpyaethu | Code PRs for CHG-001 cannot pass G5 until Phase 3 CI runs SAST, SCA and tests. Either do Phase 3 CI before CHG-001 build, or accept that CHG-001 code PRs stay open until it exists. |
| 5 | G6 "release" has no environment to release to | Intent success measure; gates G6 | paingpyaethu (release manager) | Proposed: for CHG-001, G6 is a signed, tagged API release artifact plus tagged mobile source, verified on both simulators. No production deployment. |
| 6 | No written brand, UX, accessibility or API design policy | write-spec skill step 2 | paingpyaethu (product owner) | Interim rules are in NFR-5, the UX section and the API contract section. A policy skill for each is a follow-up, not part of CHG-001. |
| 7 | Anonymous installs could be created without limit (abuse, storage cost once hosted) | Org standard: flag cost traps | paingpyaethu | Per-install cap of 500 and rate limits are in scope. Acceptable locally. Revisit before hosting. |
| 8 | Repos are public | D-008 | paingpyaethu | Test data only. No real users, Yoma Fleet data or credentials. The local CA key and `.env` files are never committed. |
| 9 | One person holds every role, including the tech lead approval required for high risk | D-007 | paingpyaethu | Accepted pilot exception. Mitigation: an independent review pass by the other agent tool (D-004) before G2 and G5. |
| 10 | The intent was merged with `Status: draft` left in the file; write-spec expects `accepted` | write-spec precondition; AGENTS.md "through the PR merge" | paingpyaethu (product owner) | Treated the merge (PR #3, G1 in gates.md) as acceptance, because the owner merged it and asked for this spec. Proposed fix in Phase 2: the author sets `Status: accepted` in the PR before merge, and a check verifies it. The file is not edited in this PR. |
| 11 | Team skills for the recommended stack are unknown | write-spec skill step 4 | paingpyaethu | Owner confirms S-2 and S-3 at G2, or picks an alternative. |
| 12 | Single points of failure | Org standard | paingpyaethu | Local only, so one Mac and one database are acceptable for CHG-001. Not acceptable for any hosted environment. |

Reasoning for risk class **high**: this change introduces the credential scheme (authentication), creates the first data model and migrations, and stores free text that may contain personal data (adlc/gates.md risk classes).

## Open questions

From intent.md:
1. **Whose tasks, and rule 1:** answered. Each install has its own private list, protected by an install token (FR-8, S-4). Exception or interpretation pending policy owner sign-off (concern #1).
2. **Platforms:** answered. iOS and Android from one codebase (NFR-1).
3. **Where it must work to count as done:** answered. The iOS Simulator and the Android Emulator against an API running locally over TLS (NFR-1, NFR-2). A real phone and a hosted dev environment are a later change.
4. **Offline:** answered. Show an error, keep the typed title, queue nothing (FR-9, NFR-3).
5. **Title rules:** answered (FR-3).
6. **Order and empty state:** answered (FR-5, FR-6).

New:
7. Is FileVault (full-disk encryption) on for the development Mac? Concern #3 depends on it.
8. Should Phase 3 CI happen before the CHG-001 build, or should the code PRs wait for it (concern #4)?

## Sign-off
- Product owner: <name>, <date>
- Tech lead (required if risk class is high): <name>, <date>
- Policy owner, security-baseline (concerns #1 to #4): <name>, <date>
