---
adr: 0001
title: Integration credential storage
status: accepted
date: 2026-07-01
tags: [security, integrations, multitenancy]
---

# ADR-0001: Integration credential storage

- **Status:** Accepted
- **Date:** 2026-07-01
- **Deciders:** Engineering architects (PR harvest 2026-07-01)
- **Consulted:** Security reviewers, integration team leads
- **Tags:** `[security]` `[integrations]` `[multitenancy]`

## Context

PR review repeatedly flagged credentials introduced directly in source:

- `access_token` / `itam_access_token` in `company.rb` or mapping constants
- Predictable `secure_code` values derived from `company.subdomain` in rake tasks
- Plaintext integration tokens in committed YAML/Ruby constants

AI-assisted coding amplifies this anti-pattern (training data often puts secrets in config/models). Committed secrets require rotation, are visible in git history, and violate least-privilege.

Existing patterns in the codebase:

- `WorkflowAutomator::CredentialVault` and `CredentialVaults::*` services
- Encrypted columns and integration-specific credential processors
- Environment / deploy-time secrets for infrastructure

## Decision

We will **never store live integration credentials, API tokens, or predictable security codes in committed application source** (models, rake tasks, initializers, constants, or product YAML checked into git).

Credentials must use one of:

1. **Credential vault** (`WorkflowAutomator::CredentialVault`, `CredentialVaults::CredentialProcessorService`) for tenant integration auth
2. **Encrypted model attributes** where an established encrypted-column pattern already exists for that integration
3. **Environment / deploy secrets** for non-tenant infrastructure keys — not per-company tokens in Ruby constants

Rake tasks and one-time scripts must not generate security-sensitive codes from guessable inputs (subdomain, id) without proper entropy and secure storage.

## Options considered

| Option | Pros | Cons |
|--------|------|------|
| **A — Vault / encrypt only (chosen)** | Aligns with existing workflow vault; tenant-scoped; auditable | Requires integration work upfront |
| **B — Constants in `company.rb` mappings** | Fast to ship | Git exposure; rotation pain; repeated PR security blocks |
| **C — Plain ENV for all tenant tokens** | Simple | Does not scale to per-tenant integrations |

## Consequences

### Positive

- Clear PR review gate (🔴 **Security**)
- AI rules (`verify-before-inventing.mdc`) and `/review-pr` hard stop
- Reduces CAR risk on integration PRs

### Negative / trade-offs

- Slightly slower integration setup vs pasting tokens in model
- Legacy mappings may need migration off constants

### Neutral / follow-ups

- Audit existing `PRIVATE_CLOUD_MAPPING`-style constants for rotation plan
- CAR sample on PRs #37556, #37684, #37685 (pre-ADR merges)

## Implementation notes

- Relevant code: `app/services/credential_vaults/`, `engines/workflow_automator/` credential vault models
- Rules: `.cursor/rules/verify-before-inventing.mdc`, `review-pr` hard stops
- Playbook: integration PRs must note credential storage approach in description

## Compliance & products

| Product flag | Affected? | Notes |
|--------------|-----------|-------|
| `IS_EZ_OFFICE_NOT_CMMS` | Yes | ITSM, telemetry, integrations |
| `IS_CMMS` | Yes | |
| `IS_ASSET_SONAR` | Yes | ITAM tokens |
| `IS_RENTALS` | Yes | Payment/integration tokens |

## References

- PR harvest: `doc/engineering-ai-metrics/harvest/2026-07-01-pr-harvest.md`
- PR review themes: credentials in source (Q2 2026)

---

*Accepted — Engineering AI guardrails.*
