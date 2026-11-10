---
name: write-spec
description: Turn an accepted ADLC intent.md into a requirements and design spec.md in todo-adlc-specs. Use when an intent.md under changes/ has been merged and someone asks for the spec, the requirements, the design, or to move a CHG change to Design.
---

# Write spec (ADLC Stage 2: Design)

Source: AI-Native SDLC Playbook p.15-17. Policy owner: product owner (see `adlc/roles.md`). Version 1, 2026-11-10.

## Preconditions

- `changes/<id>/intent.md` is merged on `main` with `Status: accepted`. Otherwise stop and say so.
- Record the intent commit: `git log -1 --format=%H main -- changes/<id>/intent.md`.

## Steps

1. Read the intent, `adlc/workflow.md`, `adlc/gates.md`, and the `AGENTS.md` of each affected code repo. Treat code repos as read-only.
2. Load every policy skill available (today: `security-baseline`). For each policy area with no skill yet (brand, UX, accessibility, API design conventions), add a row under "Areas of concern": "No written policy", owner = product owner.
3. Fill `templates/spec.md` into `changes/<id>/spec.md`:
   - Every functional requirement is numbered and testable. Acceptance criteria are concrete enough for a test to check.
   - Answer each open question from the intent, or carry it forward by name.
   - "Skills applied" lists each skill name and version used.
4. Stack and dependency decisions (required for CHG-001, and for any change that adds a dependency):
   - For each repo, and for the API contract format, give two or three options with trade-offs: maturity, fit with the team's skills, testability, hosting fit, cost, lock-in.
   - Give a recommendation and mark the decision "Pending G2". Never treat a recommendation as decided.
5. Areas of concern come first in your review: policy conflicts, policies you cannot satisfy, assumptions, and the reasoning for the risk class. Never resolve a policy conflict silently.
6. Risk class is `high` if the change touches authentication, personal data, a contract-breaking API change, infrastructure or secrets. Otherwise `routine`.
7. On branch `<id in lowercase>-spec`, commit only spec.md with message `CHG-NNN: spec draft`, push, and open a pull request titled `CHG-NNN: spec`. The PR body lists the decisions the approver must make (stack, concerns) and links the intent commit.
8. Stop. G2 is the product owner merging the PR (plus the tech lead if the risk class is high). Change `Status:` to `accepted` only when they say so explicitly. Never merge.
