# Roles

**Status: assigned 2026-11-10.** The pilot runs with a single owner who holds every role. See D-007 in `decisions.md` for the resulting exception.

| Role | Approves | Person | GitHub handle |
|---|---|---|---|
| Product owner | G1, G2, product-facing G7 | paingpyaethu | @paingpyaethu |
| Tech lead / code owner | High-risk G2 and G3, G5, cross-repo sequencing, REVIEW.md | paingpyaethu | @paingpyaethu |
| Engineer | Routine G3 | paingpyaethu | @paingpyaethu |
| Release manager | G6 | paingpyaethu | @paingpyaethu |
| Service owner | G7 | paingpyaethu | @paingpyaethu |
| Policy owners | Flagged concerns at G2, skill changes | paingpyaethu | @paingpyaethu |

`CODEOWNERS` in each repo (`.github/CODEOWNERS`) lists `@paingpyaethu` for every path. Update it and this table together when someone joins.

## Separation of duties

The agent that wrote a change can never approve or merge it (p.39). That still holds with one owner: agents open pull requests, and only the human owner merges them.

What the single-owner setup cannot give is a second human reviewer: GitHub does not let a person approve their own pull request, so the required approval count is 0 and the owner's merge is the G5 decision. This is the exception recorded in D-007.
