# Decision log

Each decision records whether it comes from the playbook (PDF) or is a pilot choice.

## D-001 Intent home is a dedicated specs repo
- Date: 2026-11-10. Status: accepted (pilot proposal approved by the workspace owner).
- Type: pilot choice, based on PDF p.12 ("a dedicated intent repo is only worth the overhead when intent spans many repositories").
- Decision: intent.md and spec.md live in `todo-adlc-specs/changes/<id>/`. plan.md lives in each code repo, because plan mode reads that repo and review checks the diff against it (p.20, p.38).
- Rejected: API repo as intent home; per-repo `intent/` folders (two sources of truth for cross-repo changes).
- Cost: code-repo CI must check out this repo at a pinned SHA with a read-only token.

## D-002 Mandatory controls are tool-neutral
- Date: 2026-11-10. Status: accepted.
- Type: pilot choice, implementing PDF p.26 ("a policy that must always hold needs something deterministic behind the skill").
- Decision: each mandatory check is a script in `scripts/checks/`, called by Git hooks, CI, Claude Code hooks and Codex hooks. CI is the backstop.

## D-003 Stack is chosen in the first spec, not before
- Date: 2026-11-10. Status: accepted.
- Type: pilot choice.
- Decision: CHG-001 spec.md records the API and mobile stacks and the API contract format, approved at G2.

## D-004 Agents coordinate through artifacts, contract first
- Date: 2026-11-10. Status: accepted.
- Type: pilot choice, based on PDF p.27 (parallel sessions share nothing but the engineer).
- Decision: see `workflow.md`, "Cross-repo coordination". Default tool split: one of Claude Code or Codex implements, the other runs an independent review pass.

## D-005 Change IDs and naming
- Date: 2026-11-10. Status: accepted.
- Type: pilot choice. Decision: see `workflow.md`, "Change IDs and naming".

## D-006 Shared instructions: AGENTS.md canonical, CLAUDE.md imports it
- Date: 2026-11-10. Status: accepted.
- Type: pilot choice. CLAUDE.md is the PDF's artifact (p.23); AGENTS.md is added because Codex reads it.
- Decision: each repo's `AGENTS.md` holds the shared instructions (under one page, p.23). `CLAUDE.md` starts with `@AGENTS.md` and adds Claude-only notes. No instruction files in the unversioned parent folder.

## F-001 Codex CLI 0.155.1 capabilities (finding, verified from the installed binary, not yet from official docs)
- Date: 2026-11-10.
- Instructions: reads `AGENTS.override.md`, then `AGENTS.md`, then `project_doc_fallback_filenames` (empty by default, so it does not read CLAUDE.md).
- Skills: repo skills in `.agents/skills/`, user skills in `~/.codex/skills/`. Same `SKILL.md` folder format as Claude Code.
- Hooks: `hooks` feature is stable; repo config in `.codex/hooks.json` with events including PreToolUse, PostToolUse, SessionStart, Stop.
- Subagents: repo agent roles in `.codex/agents/`.
- Consequence for Phase 1: keep skills once in `.agents/skills/` and expose them to Claude Code through `.claude/skills` (symlink or copy, to be tested on both tools before relying on it).

## Pending decisions
- Git host, owning account or org, and remotes for all three repos.
- Named people for each role in `roles.md`, and whether a second reviewer exists for G5.
- Model access route for CI (direct API keys or Bedrock through OIDC) and the eval budget.
