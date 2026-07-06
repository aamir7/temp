---
adr: 0002
title: Multitenancy composite indexes
status: accepted
date: 2026-07-01
tags: [multitenancy, database, performance, cmdb]
---

# ADR-0002: Multitenancy composite indexes

- **Status:** Accepted
- **Date:** 2026-07-01
- **Deciders:** Engineering architects (PR harvest 2026-07-01)
- **Consulted:** CMDB architects, database reviewers
- **Tags:** `[multitenancy]` `[database]` `[performance]` `[cmdb]`

## Context

The monolith isolates tenants via `company_id` and `ApplicationRecord#default_scope`. At scale, queries that filter by tenant without a leading `company_id` index cause full-table scans and cross-tenant performance bleed.

PR review repeatedly requested composite indexes on CMDB and relationship migrations:

- Indexes must lead with `company_id`
- CMDB relationship table shape deferred to architect 1:1 review when reviewers say "discuss this table in detail"

Migrations still ship without index plans despite existing rules in `database-migrations.mdc` and the PR template SQL section.

## Decision

We will **require `company_id` on all new tenant-scoped tables and composite indexes on tenant data that lead with `company_id`**, verified with `EXPLAIN` on representative queries before merge.

CMDB relationship schema changes (`cmdb_*` tables, relationship edges, non-trivial index changes) require **architect review** per `doc/playbooks/cmdb-schema-changes.md` — thread consensus alone is not sufficient for new relationship table shapes.

## Options considered

| Option | Pros | Cons |
|--------|------|------|
| **A — company_id-first composites + architect gate (chosen)** | Matches multitenancy model; predictable query plans | Slower CMDB schema iteration |
| **B — Single-column indexes only** | Simpler migrations | Poor selectivity at scale; misses tenant filter |
| **C — Post-merge index tuning** | Faster initial merge | Production pain; rework in hot paths |

## Consequences

### Positive

- Consistent tenant isolation performance
- Clear `/review-pr` and PR template SQL gate
- CMDB schema debates escalated to playbook instead of endless PR threads

### Negative / trade-offs

- Authors must run `EXPLAIN` and iterate indexes before review
- CMDB PRs may wait on architect availability

### Neutral / follow-ups

- Future CI: migration linter for `company_id` without index (Layer 4 — not before two harvest cycles)
- CAR sample on PR #37518 CMDB migration threads

## Implementation notes

- Rules: `.cursor/rules/database-migrations.mdc`, `multi-product-tenancy.mdc`
- Playbook: `doc/playbooks/cmdb-schema-changes.md`
- PR label: `Ready for Architect Review` when CMDB schema is non-trivial
- Review skill: `.cursor/skills/review-pr/checklist.md` — Multitenancy & CMDB sections

## Compliance & products

| Product flag | Affected? | Notes |
|--------------|-----------|-------|
| `IS_EZ_OFFICE_NOT_CMMS` | Yes | All tenant tables |
| `IS_CMMS` | Yes | |
| `IS_ASSET_SONAR` | Yes | CMDB + ITAM paths |
| `IS_RENTALS` | Yes | Basket/customer tables |

## References

- PR harvest: `doc/engineering-ai-metrics/harvest/2026-07-01-pr-harvest.md` (Theme 3)
- PR #37518 — CMDB relationship index discussion

---

*Accepted — Engineering AI guardrails.*
