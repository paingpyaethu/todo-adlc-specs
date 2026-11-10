---
name: write-intent
description: Capture an idea, feature request, bug report or incident as an ADLC intent.md in todo-adlc-specs. Use when someone describes a problem or idea they want built or fixed, or asks to write an intent, start a change, or capture an idea.
---

# Write intent (ADLC Stage 1: Plan)

Source: AI-Native SDLC Playbook p.11-13. Policy owner: product owner (see `adlc/roles.md`). Version 1, 2026-11-10.

## Steps

1. Let the originator describe the problem in their own words. Do not propose solutions or technology.
2. Ask the questions an analyst would ask, in one short batch, only for what is still missing:
   - Who is affected, and how often?
   - What can they not do today, and what does better look like?
   - Constraints: security, privacy, platform, deadline.
   - What is out of scope?
   - How will we know it worked?
   Repeat until the idea is concrete, at most three rounds. Anything still unknown becomes an open question.
3. Choose the change ID: highest number in `changes/` plus one, zero-padded to three digits. Slug: kebab-case, at most five words, from the outcome (for example `CHG-001-create-todo`).
4. Copy `templates/intent.md` to `changes/<id>/intent.md` and fill it.
   - Keep the originator's own words in "Problem" and "Proposed outcome".
   - Never fill a section by guessing. Write "Unknown" and add an open question.
   - `Status: draft`.
5. Show the full draft to the originator and apply their corrections.
6. On branch `<id in lowercase>` (for example `chg-001-create-todo`), commit only that intent.md with message `CHG-NNN: intent draft`, push, and open a pull request titled `CHG-NNN: intent`. List the open questions in the PR body.
7. Stop. G1 is the product owner merging the PR. Change `Status:` to `accepted` only when the product owner says so explicitly, as a last commit on the PR before they merge. Never merge.

## Do not

- Put requirements detail, design, technology choices or tasks in intent.md. Those belong to the spec and the plan.
- Include personal data, credentials, or data from company systems. These repos are public (D-008).
