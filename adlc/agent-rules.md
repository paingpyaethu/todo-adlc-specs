# ADLC agent rules (canonical source)

This file is the single source of the rules block that every repo's `AGENTS.md` carries.
Edit the rules here first, then copy the block between the markers into:

- `todo-adlc-specs/AGENTS.md`
- `todo-adlc-api/AGENTS.md`
- `todo-adlc-mobile/AGENTS.md`

Bump the version in the begin marker on every change. A CI drift check is planned for Phase 3.
Changes to this file pass the config gate (see `gates.md`).

<!-- adlc-rules:begin v1 -->
## ADLC rules (shared by Claude Code and Codex)

1. Every change has an ID `CHG-NNN-slug`. Its intent.md and spec.md live in `todo-adlc-specs/changes/<id>/`. Its plan.md lives in each affected code repo at `docs/changes/<id>/plan.md`.
2. No implementation without an accepted plan.md for the change ID. If there is none, stop and produce a plan instead.
3. Read intent.md and spec.md at the specs-repo commit recorded in the plan.md header, not at the latest commit.
4. If the implementation departs from plan.md, update plan.md in the same commit.
5. For a bug fix, write a failing test first and commit it. After that, do not edit test files during the fix.
6. Before reporting a task done, run the build, test and lint commands listed under "Commands" and paste their output. If a test fails, fix the code, not the test.
7. Never push to `main`, approve a pull request, merge your own pull request, or deploy beyond the dev environment.
8. Never read, print or commit secrets (`.env*`, keys, tokens). Never disable TLS verification or authentication, not even for testing.
9. Do not introduce a language, framework or dependency that the accepted spec.md does not name.
10. When the same mistake is corrected twice, propose adding it to "Things agents get wrong" in AGENTS.md.
<!-- adlc-rules:end -->
