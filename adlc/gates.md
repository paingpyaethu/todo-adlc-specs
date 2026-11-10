# Human approval and verification gates

The agent may act up to the production gate and cannot pass it (p.46). A skill is advisory; any control that must always hold is backed by a deterministic check (p.26).

**Enforcement rule (pilot choice, D-002):** each mandatory check is one script in `scripts/checks/` of the repo, called from three places: the Git hook, CI, and the agent hooks (Claude Code `.claude/settings.json`, Codex `.codex/hooks.json`). CI is the backstop because local hooks can be skipped.

| Gate | When | Approver | Evidence | Enforcement | Status |
|---|---|---|---|---|---|
| G1 Intent accepted | End of Plan | Product owner | Merged intent PR with author and timestamp (p.13) | PR merge in specs repo | Manual until remote exists |
| G2 Spec accepted | End of Design | Product owner; tech lead for high risk; named policy owner per flagged concern (p.16-17) | Merged spec PR, every concern resolved or carried | PR merge plus CODEOWNERS | Manual until remote exists |
| G3 Plan accepted | Start of Build | Engineer; tech lead for high risk (p.20) | plan.md committed before code | Claude plan mode; `plan-exists` check for both tools | Check planned for Phase 2 |
| G4 Verification | Before "done" | Automated | Pasted build/test/lint output, verifier report, SAST, SCA, secret scan (p.32-33) | Git hooks, agent hooks, CI; test-file guard during fixes | Phase 2 and 3 |
| G5 PR merge | Deploy | Human code owner, never the authoring agent (p.39) | AI review findings ranked by severity per REVIEW.md | Branch protection | Phase 3 |
| G6 Release | Production | Named release manager (p.46) | Release record | Production-gate hook plus CI environment protection | Phase 3 |
| G7 Triage | Maintain | Service owner; product owner for product-facing findings (p.50) | Agent-written intent.md with evidence | Same path as G1 | Phase 4 |
| Config gate | Change to AGENTS.md, CLAUDE.md, skills, hooks, agent rules | Owner of that config | Eval pass rate (p.34-35) | CI merge check | Phase 3 |

## Risk classes

- **Routine**: small blast radius, covered by existing tests. Engineer approves G3.
- **High risk**: auth, data model or migrations, contract-breaking API changes, infra, secrets handling, anything touching personal data. Tech lead approves G2 and G3.

The spec.md states the risk class; reviewers may raise it, never lower it without the tech lead.
