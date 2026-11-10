# todo-adlc-specs

Intent home for the todo ADLC pilot. It holds intent.md and spec.md for every change, the ADLC process docs and the templates. No application code lives here.

## Layout
- `changes/<id>/intent.md`, `changes/<id>/spec.md`: one folder per change.
- `templates/`: intent.md, spec.md, plan.md. Always start from these.
- `adlc/`: workflow, gates, roles, decision log, canonical agent rules.
- `lessons/`: post-incident write-ups that later investigations read.
- `.agents/skills/`: write-intent, write-spec, security-baseline. The only copy; `.claude/skills` is a symlink to it.

## Commands
None yet. This repo is markdown only. Phase 2 adds checks (template fields present, rules block in sync).

## Working in this repo
- Plan stage: ask the questions an analyst would ask (scope, users, constraints, what success looks like, what is out of scope) before writing intent.md. Keep the originator's own words.
- Design stage: write spec.md from an accepted intent.md only. List every policy conflict under "Areas of concern" with its owner. Never resolve a conflict silently.
- Set `Status:` to `draft` when writing. Only a human changes it to `accepted` or `rejected`, through the PR merge or close.
- For cross-repo work, start the session here with `claude --add-dir ../todo-adlc-api --add-dir ../todo-adlc-mobile` and treat the code repos as read-only.
- Process details: `adlc/workflow.md` and `adlc/gates.md`.

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

## Things agents get wrong
None recorded yet.
