# Roles

**Status: pending.** Names are not assigned yet. Fill this table, then generate `CODEOWNERS` in each repo once the Git host is chosen.

| Role | Approves | Person | Git host handle |
|---|---|---|---|
| Product owner | G1, G2, product-facing G7 | TBD | TBD |
| Tech lead / code owner | High-risk G2 and G3, G5, cross-repo sequencing, REVIEW.md | TBD | TBD |
| Engineer(s) | Routine G3 | TBD | TBD |
| Release manager | G6 | TBD | TBD |
| Service owner | G7 | TBD | TBD |
| Policy owners | Flagged concerns at G2, skill changes | TBD per policy | TBD |

## Separation of duties

The agent that wrote a change can never approve it (p.39). GitHub also prevents a person from approving their own pull request, so G5 needs a second human. If the pilot runs with one person, record that exception in `decisions.md` with its expiry.
