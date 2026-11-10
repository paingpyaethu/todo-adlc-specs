@AGENTS.md

## Claude Code notes
- AGENTS.md above is shared with Codex. Put shared guidance there; this section is for Claude-only behavior.
- Design sessions: attach the accepted intent.md and use the prompt shape from the playbook (p.16): apply the available skills, write spec.md, and describe every area of concern.
- Skills live in `.agents/skills/` (shared with Codex) and reach Claude Code through the `.claude/skills` symlink. Edit them there, never through a copy.
