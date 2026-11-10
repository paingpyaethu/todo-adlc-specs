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

## D-007 Single-owner exception to separation of duties
- Date: 2026-11-10. Status: accepted. Expires: when a second reviewer joins, or at the end of the pilot, whichever comes first.
- Type: pilot exception to PDF p.37-39 (code owner approval through branch protection).
- Decision: paingpyaethu holds every role (`roles.md`). Branch protection on `main` requires a pull request but 0 approvals, because GitHub blocks self-approval. The owner's merge, after reading the AI review findings, is the G5 decision. Agents never merge (rule 7); a hook enforcing that arrives in Phase 2.
- Risk: no independent human review, and a single approver is a single point of failure. Mitigation: AI review passes plus the independent second-tool review from D-004.

## D-008 Git host, identity and branch protection
- Date: 2026-11-10. Status: accepted.
- Type: pilot choice.
- Decision: all three repos are public GitHub repos owned by `paingpyaethu` (made public on 2026-11-10 so branch protection is available without GitHub Pro; the owner accepted that code, specs, this log and the commit author email are publicly visible). Remotes use the SSH alias `github.com-personal`. Commits use the repo-local identity `paingpyaethu <paingpyaethu@gmail.com>`.
- Intended protection for `main`: changes only through pull requests (administrators included), 0 required approvals (D-007), no force pushes, no deletion, conversations resolved before merge. Required status checks are added in Phase 3 when CI exists.
- Status of protection: **active** on all three repos since 2026-11-10, verified by reading the settings back from the GitHub API. (The first attempt, while the repos were private, failed with HTTP 403 because private-repo protection needs GitHub Pro.)
- Also enabled on all three repos: secret scanning, secret scanning push protection, and Dependabot alerts (part of G4).
- Constraint: no data from Yoma Fleet systems, real users or credentials may enter these repos while they are public. Revisit visibility before any change touches real data.

## D-009 Skills live once, in .agents/skills
- Date: 2026-11-10. Status: proposed (accepted when its PR merges).
- Type: pilot choice, implementing PDF p.25 (skills in the repo, version-controlled) for two tools, based on F-001.
- Decision: each skill is a folder with SKILL.md under `.agents/skills/`, which Codex discovers. `.claude/skills` is a symlink to `../.agents/skills` so Claude Code reads the same files. First skills: write-intent, write-spec (Plan and Design), security-baseline (policy, owner paingpyaethu).
- Not yet covered by a written policy: brand, UX, accessibility, API design conventions. The write-spec skill flags these as areas of concern until skills exist.

## Pending decisions
- Model access route for CI (direct API keys or Bedrock through OIDC) and the eval budget.
