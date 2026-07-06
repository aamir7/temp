# Architecture Decision Records (ADRs)

Durable records of **significant technical decisions**: context, decision, alternatives, consequences.

**Strategy:** [ENGINEERING-AI-STRATEGY.md](../ENGINEERING-AI-STRATEGY.md) · Phase 2+ in the incremental plan.

## When to write one

- Affects multiple products or teams
- Hard to reverse (data model, async, API contract)
- Keeps recurring in PR review
- Should guide AI-assisted implementation

**Do not** write ADRs for routine features — use `.cursor/rules/` or playbooks.

## Numbering

| File | Title | Status |
|------|-------|--------|
| `template.md` | Template | — |
| [0001-integration-credentials-storage.md](0001-integration-credentials-storage.md) | Integration credential storage | Accepted |
| [0002-multitenancy-composite-indexes.md](0002-multitenancy-composite-indexes.md) | Multitenancy composite indexes | Accepted |
| [0003-delayed-job-for-async-work.md](0003-delayed-job-for-async-work.md) | Use delayed_job for async work | Accepted |

## Process

1. Copy `template.md` → `NNNN-short-title.md`
2. PR with architect or tech lead review
3. Status: Proposed → Accepted → Deprecated / Superseded

**Do not edit Accepted ADRs in place** — supersede with a new ADR when decisions change.

## For AI assistants

Search `doc/adr/` before architectural changes. If work contradicts an **Accepted** ADR, stop and escalate.

---

*Part of Engineering AI Phase 2 (verification & decisions).*
