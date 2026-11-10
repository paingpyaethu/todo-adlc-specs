# todo-adlc-specs

The version-controlled intent home for the todo Agentic Development Lifecycle (ADLC) pilot, following "The AI-Native SDLC Playbook".

Related repos: `todo-adlc-api`, `todo-adlc-mobile` (siblings in the same local workspace).

## How a change flows

1. **Plan**: brainstorm with an agent, copy `templates/intent.md` to `changes/CHG-NNN-slug/intent.md`, open a PR. Product owner merge = G1.
2. **Design**: agent writes `spec.md` next to the intent from `templates/spec.md`, open a PR. Product owner (plus tech lead if high risk) merge = G2.
3. **Build**: in each affected code repo, plan mode produces `docs/changes/CHG-NNN-slug/plan.md` from `templates/plan.md`. Engineer accepts = G3, then implementation.
4. **Test, Deploy, Maintain**: see `adlc/workflow.md` and `adlc/gates.md`.

## Pilot status

Phase 0 (foundations) in progress. Open items are listed under "Pending decisions" in `adlc/decisions.md`.
