---
name: security-baseline
description: Apply the pilot's security baseline. Use whenever writing or reviewing a spec, plan, code or CI configuration that touches authentication, authorization, input handling, data storage, logging, secrets, network calls, dependencies or infrastructure.
---

# Security baseline

Policy owner: paingpyaethu (see `adlc/roles.md`). Source of truth: the organization's engineering security standards. Version 1, 2026-11-10. Changes to this skill need the policy owner's approval (config gate).

## Rules

1. **Authentication and authorization.** Every endpoint requires authentication except an explicitly listed health check. Authorize every request against the resource owner.
2. **Queries.** Parameterized queries or ORM bindings only. Never build a query by string concatenation.
3. **Input.** Validate all input against the API contract schema and reject unknown fields.
4. **Secrets.** No hardcoded secrets. Read them from the environment or a secret manager. Commit only `.env.example`.
5. **Transport.** TLS for all network traffic. Never disable certificate verification, including in tests; use local test certificates or test doubles.
6. **No auth bypass.** Never bypass authentication for testing. Use test accounts and fixtures.
7. **Logging.** Never log secrets, tokens or personal data, including in error messages.
8. **Data at rest.** Encrypt stored personal data.
9. **Least privilege.** Any cloud or IAM permission is the minimum needed, defined as infrastructure as code (Terraform preferred on AWS).
10. **Pipeline.** CI runs SAST, dependency scanning (SCA) and secret scanning. Release artifacts are signed.
11. **Regulated data.** Financial, identity (KYC) or location data tied to individuals must not enter these public repos. Route any such need to Legal or the Data Protection lead.

## How to apply

- In a spec: add each applicable rule to "Security and privacy". Any rule you cannot satisfy goes under "Areas of concern" with the policy owner named.
- In a plan or code: follow the rules. Summarize which ones applied at the end of the task.
- In review: a violated rule is an Important finding, not a nit.
