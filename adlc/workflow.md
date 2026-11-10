# ADLC workflow

Source: "The AI-Native SDLC Playbook" (Anthropic). Page references are to that PDF.
Items marked **(pilot choice)** are decisions for this pilot, not playbook requirements.

## Repositories

| Repo | Holds |
|---|---|
| `todo-adlc-specs` | intent.md and spec.md per change, ADLC process docs, templates, lessons. No application code. **(pilot choice, D-001)** |
| `todo-adlc-api` | API code, tests, the API contract file, `docs/changes/<id>/plan.md` |
| `todo-adlc-mobile` | Mobile code, tests, `docs/changes/<id>/plan.md` |

The parent folder `todo-adlc/` is a local workspace only. It is not version-controlled and holds no instructions.

## Stages

Every stage ends by committing an artifact that the next stage reads (p.7). During the pilot every hand-off is triggered by hand (p.9); automation comes in Phase 4.

| Stage | Input | Output | Gate | Starts the next stage |
|---|---|---|---|---|
| Plan | Idea, ticket or incident | `changes/<id>/intent.md` (specs repo, via PR) | G1 | Merged intent PR (p.8) |
| Design | Accepted intent.md plus skills | `changes/<id>/spec.md` (specs repo, via PR) | G2 | Merged spec PR (p.8) |
| Build | Accepted spec.md | `docs/changes/<id>/plan.md`, then code and tests (each code repo) | G3, G4 | Pull request opened |
| Test | The diff | Pasted build/test/lint output, verifier report, CI check runs | G4 | All checks green |
| Deploy | The pull request | AI review findings, human approval, release record | G5, G6 | Merge (p.8), then release |
| Maintain | Band breach, ticket, channel message | New `intent.md` plus `lessons/<date>-<slug>.md` | G7 | Back to Plan (p.8) |

The first spec.md (CHG-001) also records the stack and the API contract format, so frameworks are chosen at G2 with human sign-off **(pilot choice, D-003)**.

## Change IDs and naming (pilot choice, D-005)

- ID: `CHG-NNN-slug`, for example `CHG-001-create-todo`. Next number = highest existing number in `changes/` plus one.
- Branch in every repo: `chg-NNN-slug`.
- Pull request titles start with `CHG-NNN:`.
- Each plan.md header records the specs-repo commit SHAs of the intent.md and spec.md it was planned against (linkage, p.22).

## Cross-repo coordination (pilot choice, D-004)

- Agents coordinate only through committed artifacts. Parallel sessions know nothing about each other (p.27).
- Contract first: a cross-repo spec.md defines the API contract change. `todo-adlc-api` owns the machine-readable contract file; `todo-adlc-mobile` pins a contract version and tests against it.
- Both plan.md files are written against the same spec commit and may be built in parallel.
- Merge order: API first (backward compatible unless the spec says otherwise), then mobile.
- The tech lead owns cross-repo sequencing.
- Cross-repo Plan and Design sessions run from this repo with the code repos added:
  `claude --add-dir ../todo-adlc-api --add-dir ../todo-adlc-mobile`

## Source of truth (p.22)

The repos are the source of truth for intent, spec and plan. If a ticket in another system (for example ClickUp) tracks the change, intent.md records the ticket ID and the ticket records the commit SHA.
